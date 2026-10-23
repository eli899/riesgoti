# 02. Hoja de Ruta de Implementación Frontend (React + TS + Redux Toolkit + Zod + Tailwind) — RiesgoTI

## Arquitectura Frontend sobre la Base de Figma (`src/`)
Se toma como base el diseño visual del prototipo provisto (`src/App.tsx`, `src/ui.tsx`, `src/pages/*.tsx`) y se reemplaza el estado mockeado en memoria (`src/store.tsx` y `src/data.ts`) por una arquitectura empresarial escalable basada en **Redux Toolkit (RTK + RTK Query)** y validación de contratos y formularios con **Zod**:

```text
src/
├── app/
│   ├── store.ts                 # Configuración central de Redux Store + middlewares de RTK Query
│   ├── hooks.ts                 # useAppDispatch y useAppSelector tipados
│   └── api/
│       ├── baseApi.ts           # createApi con inyección de JWT Bearer y refresh automático
│       ├── adminApi.ts          # Endpoints RTK Query: AI Config, Test Ping, Ingesta PDF, Matriz
│       ├── incidentsApi.ts      # Endpoints RTK Query: Incidencias, Orientación RAG, Atención N2
│       └── risksApi.ts          # Endpoints RTK Query: Agrupaciones, Evaluación 5x5, Decisiones, Dashboard
├── features/
│   ├── auth/
│   │   ├── authSlice.ts         # Usuario activo, rol ('REPORTANTE' | 'ANALISTA' | 'ADMIN'), permisos y tokens
│   │   └── RequirePermission.tsx# Guardia de rutas y ocultamiento de acciones según RBAC
│   └── ui/
│       └── uiSlice.ts           # Control global del Drawer de Citas RAG, modales y filtros activos
├── schemas/                     # Esquemas Zod (Single Source of Truth para Forms y respuestas de API)
│   ├── aiConfig.schema.ts
│   ├── document.schema.ts
│   ├── incident.schema.ts
│   └── riskEvaluation.schema.ts
├── components/
│   ├── ui.tsx                   # Componentes base Tailwind existentes + MarkdownViewer seguro
│   └── CitationDrawer.tsx       # Drawer lateral interactivo para CITA_ORIENTACION y CITA_VERSION_RIESGO
└── pages/
    ├── admin.tsx                # Vistas del Administrador (Config IA/RAG + Ingesta/Preview Markdown + Matriz)
    ├── reporter.tsx             # Vistas del Reportante (Formulario Zod + Solución RAG con citas)
    ├── staff.tsx                # Vistas del Analista N2 (Consola de Atención Humana)
    └── risk.tsx                 # Vistas del Analista de Riesgos (Agrupaciones IA + Matriz 5x5 + Decisiones)
```

---

## Etapa 1: Configuración de Redux Toolkit, RTK Query y Esquemas Zod Base
**Archivos objetivo:** `src/app/store.ts`, `src/features/auth/authSlice.ts`, `src/schemas/*.ts`.

### Pasos de Construcción
1. **Configuración de Redux Store y Persistencia de Sesión (`authSlice.ts`):**
   * Configurar `configureStore` integrando `authSlice`, `uiSlice` y el reducer de `baseApi` (RTK Query) con los tags de caché: `['AIConfig', 'DocumentVersion', 'Incident', 'OperationalGuidance', 'RiskVersion', 'RiskDashboard', 'MatrixConfig']`.
   * Implementar en `RequirePermission.tsx` la restricción estricta de vistas: el rol `ADMIN` no puede renderizar ni disparar mutaciones de evaluación o validación de riesgos.
2. **Definición de Esquemas Zod (`src/schemas/`):**
   * **Esquema de Configuración IA/RAG (`aiConfig.schema.ts`):**
     ```typescript
     import { z } from 'zod';

     export const aiConfigFormSchema = z.object({
       proveedor_llm: z.enum(['OPENAI', 'GEMINI', 'HUGGINGFACE_LOCAL']),
       modelo_llm: z.string().min(2, 'Seleccione o ingrese un modelo LLM válido'),
       endpoint_base_llm: z.string().url('URL inválida').optional().or(z.literal('')),
       api_key_llm: z.string().optional(),
       proveedor_embedding: z.enum(['OPENAI', 'GEMINI', 'SENTENCE_TRANSFORMERS']),
       modelo_embedding: z.string().min(2, 'El modelo de embedding es obligatorio'),
       api_key_embedding: z.string().optional(),
       chunk_size: z.number().int().min(256).max(2048),
       chunk_overlap: z.number().int().min(0).max(512),
       top_k: z.number().int().min(1).max(15),
       umbral_similitud: z.number().min(0).max(1),
     }).refine((data) => data.chunk_overlap < data.chunk_size / 2, {
       message: 'El solapamiento (chunk_overlap) debe ser estrictamente menor a la mitad del chunk_size',
       path: ['chunk_overlap'],
     });
     ```
   * **Esquema de Decisión y Evaluación de Riesgo (`riskEvaluation.schema.ts`):**
     * Usa `.superRefine()` en Zod para exigir:
       * Si `accion === 'VALIDADA'`: `probabilidad` ($1-5$), `impacto` ($1-5$) y `justificacion` ($\ge 15$ caracteres) son obligatorios; además, si `tiene_citas_estrategicas === false`, exige `justificacion_sin_fuentes` ($\ge 15$ caracteres).
       * Si `accion === 'RECHAZADA'`: exige únicamente `observacion` (motivo del rechazo) y exime de seleccionar probabilidad e impacto.

---

## Etapa 2: Vistas del Administrador (`src/pages/admin.tsx`)
Se conectan las pestañas de Administración con `adminApi.ts` (RTK Query) y formularios validados con `react-hook-form` + `zodResolver`.

### 2.1 Sub-vista: Configuración de API Keys, Proveedores y Parámetros RAG
1. **Formulario de Credenciales y Hiperparámetros:**
   * Conectado a `useGetActiveAIConfigQuery()` y `useUpdateAIConfigMutation()`.
   * Los campos de API Key muestran en el placeholder la máscara recibida del backend (`api_key_llm_mascara`, ej. `AIzaSy...4321`) y solo envían un valor nuevo si el Administrador decide reemplazar la credencial.
   * Botón **"Probar Conexión (Test Ping)"** conectado a `useTestAIConnectionMutation()`: muestra estado de carga por proveedor y renderiza tarjetas de diagnóstico con latencia en $\text{ms}$ y dimensiones del vector (`768 dims` / `1536 dims`).
   * Si la respuesta de `PUT /api/v1/admin/ai-config/` indica `requiere_reindexacion_documentos: true`, despliega una alerta advirtiendo qué versiones documentales activas fueron generadas con un `modelo_embedding` anterior.

### 2.2 Sub-vista: Subida de PDF, Previsualización Markdown y Autorización (`VERSION_DOCUMENTO`)
1. **Formulario de Carga (`react-dropzone` + Zod):**
   * Valida que el archivo sea `.pdf` ($\le 25\text{ MB}$), exige `codigo`, `titulo`, `version` y `finalidad` (`OPERATIVA`, `ESTRATEGICA`, `AMBAS`) y ejecuta `useUploadDocumentVersionMutation()`.
2. **Tabla de Ciclo de Vida con Polling Automático en RTK Query:**
   * La consulta `useGetDocumentVersionsQuery(undefined, { pollingInterval: hasProcessingDocs ? 3000 : 0 })` refresca automáticamente mientras exista alguna fila con `procesamiento === 'PROCESANDO'`.
   * Badges mapeados exactamente a `VERSION_DOCUMENTO`:
     * `PROCESANDO`: Badge azul animado.
     * `ERROR`: Badge rojo con popover que muestra `error_procesamiento`.
     * `PENDIENTE_AUTORIZACION`: Badge ámbar con botón **"Previsualizar Markdown y Autorizar"**.
     * `AUTORIZADO` + `activa === true`: Badge Esmeralda **"Disponible en RAG"** con botón para desactivar.
     * `AUTORIZADO` + `activa === false`: Badge Gris **"Autorizado - Inactivo"**.
3. **Modal de Previsualización de Chunks Markdown (`useGetDocumentPreviewQuery(versionId)`):**
   * **Columna izquierda:** Lista de fragmentos (`FRAGMENTO_DOCUMENTO.orden`) con su etiqueta de trazabilidad `Pág. {pagina_inicio} - {pagina_fin}`.
   * **Columna derecha:** Pestaña *"Vista Renderizada"* usando `<ReactMarkdown remarkPlugins={[remarkGfm]}>` (para verificar que las tablas técnicas del PDF se ven perfectamente estructuradas) y pestaña *"Markdown Crudo"*.
   * Botón **"Autorizar y Activar Versión"** conectado a `useAuthorizeDocumentVersionMutation()`, que invalida el tag `'DocumentVersion'`.

---

## Etapa 3: Vistas del Reportante (`src/pages/reporter.tsx`)

### Pasos de Construcción
1. **Formulario de Creación de Incidencias (`incidentFormSchema` con Zod):**
   * Campos: `titulo` ($\ge 5$ caracteres), `descripcion` ($\ge 20$ caracteres, redactada en lenguaje cotidiano del usuario), selector de `activo_reportado_id` o campo de texto libre `activo_texto`, y zona de adjuntos opcionales (`EVIDENCIA_INCIDENCIA`).
   * Al enviar mediante `useCreateIncidentMutation()`, redirige inmediatamente a la vista de detalle de la incidencia donde RTK Query hace polling de `useGetOperationalGuidanceQuery(incidenciaId, { pollingInterval: 2500 })` hasta que `tarea_ia.estado === 'COMPLETADA'`.
2. **Panel de Solución Operativa Estructurada y Citas Interactivas:**
   * Validación en tiempo de ejecución de la respuesta con `operationalGuidanceResponseSchema` (Zod).
   * Renderiza `orientacion.resumen`, el banner de `orientacion.advertencia` (si existe) y la lista numerada de `orientacion.pasos` (`PASO_ORIENTACION`).
   * **Citas Interactivas (`CitationDrawer.tsx`):** Cada etiqueta `[1]`, `[2]` en las instrucciones o al pie del panel dispara `dispatch(openCitationDrawer(cita))`, abriendo el Drawer lateral que muestra:
     * Código y título del documento (`DOCUMENTO.codigo`, `DOCUMENTO.titulo`).
     * Versión (`VERSION_DOCUMENTO.version`) y rango exacto de páginas (`FRAGMENTO_DOCUMENTO.pagina_inicio` a `pagina_fin`).
     * Contenido del fragmento renderizado en Markdown con sus tablas e instrucciones originales.
3. **Botones de Confirmación (`useSubmitOrientationFeedbackMutation()`):**
   * **"Sí, funcionó" (`FUNCIONO`):** Cambia el estado de la incidencia a `CERRADA` e invalida `'Incident'`.
   * **"No funcionó, escalar a Analista" (`NO_FUNCIONO`):** Cambia el estado a `PENDIENTE_ATENCION_HUMANA` e informa que el caso pasó a la bandeja de Atención Humana junto con el diagnóstico previo.

---

## Etapa 4: Vistas del Analista (`src/pages/staff.tsx` y `src/pages/risk.tsx`)

### 4.1 Consola de Atención Humana (`src/pages/staff.tsx`)
1. **Vista Consolidada del Caso Derivado:**
   * Muestra la descripción original, descargas de `EVIDENCIA_INCIDENCIA` y el bloque **"Historial de Orientación IA Previa"** (mostrando qué pasos sugirió el RAG y qué fuentes citó, o si derivó por `SIN_RESPALDO`, evitando que el analista le pida al usuario repetir lo que ya falló).
2. **Formulario de Intervención (`humanAttentionSchema` en Zod):**
   * Botón **"Tomar Caso (`EN_ATENCION`)"** $\rightarrow$ habilita los campos `acciones` y `nota_resolucion`.
   * Permite **"Guardar Avance"** (mantiene el caso en `EN_ATENCION`) o **"Confirmar Resolución y Cerrar"** (exige ambos campos completos en Zod y transiciona a `CERRADA`).

### 4.2 Revisión de Agrupaciones IA, Matriz $5 \times 5$ y Decisiones (`src/pages/risk.tsx`)
1. **Depuración de Agrupación (`VERSION_INCIDENCIA`) y Confirmación de Activo:**
   * Lista las incidencias agrupadas mostrando `similitud` (ej. $91.4\%$), `motivo_similitud` y `extracto_reporte` (incluyendo tanto incidencias abiertas como cerradas).
   * Acción **"Retirar falso positivo"**: abre modal validado con Zod (`z.object({ motivo_retiro: z.string().min(10) })`) conectado a `useRemoveIncidentFromRiskGroupMutation()`.
   * Selector de confirmación de `activo_confirmado_id` mostrando la badge de `criticidad` del activo.
2. **Matriz Interactiva $5 \times 5$ (`EVALUACION_RIESGO`) y Fuentes ISO (`CITA_VERSION_RIESGO`):**
   * Renderiza la matriz activa traída desde `useGetActiveMatrixQuery()`.
   * Al seleccionar una celda `(probabilidad, impacto)`, calcula visualmente el nivel resultante (ej. `P4` $\times$ `I3` = **`ALTO`**) y vincula el `celda_id` al formulario de `react-hook-form`.
   * Si la versión de riesgo no posee citas estratégicas en `CITA_VERSION_RIESGO`, despliega automáticamente el campo obligatorio `justificacion_sin_fuentes`.
3. **Ejecución de Decisión (`VALIDADA` | `CORREGIDA` | `RECHAZADA`) y Tablero de Indicadores:**
   * Conectado a `useSubmitRiskDecisionMutation()`, que invalida `'RiskVersion'` y `'RiskDashboard'`.
   * La pestaña de **Registro e Indicadores** consume `useGetRiskDashboardQuery()`, mostrando riesgos vigentes por nivel, activos con mayor recurrencia y trazabilidad de versiones sin duplicar el conteo por `RIESGO.id`.

---

## Criterios de Prueba Visual y Funcional Paso a Paso (QA Checklist con Redux & Zod)

| # | Flujo a Verificar | Validación en Frontend (Zod + Redux) | Validación en Backend (DRF + BD) |
| :-: | :--- | :--- | :--- |
| **1** | **Guardado de Parámetros RAG Inválidos** | Ingresar `chunk_size = 500` y `chunk_overlap = 300`. `zodResolver` bloquea el submit y marca error en `chunk_overlap`. | No se emite petición HTTP innecesaria; el serializer de DRF posee la misma regla como segunda barrera. |
| **2** | **Subida PDF y Polling de Estado** | Al subir un PDF, RTK Query hace polling cada $3\text{ s}$ hasta que el badge pasa de `PROCESANDO` a `PENDIENTE_AUTORIZACION`. | Celery extrae Markdown con `pymupdf4llm`, crea filas en `FRAGMENTO_DOCUMENTO` con `pagina_inicio`/`fin` e indexa en FAISS. |
| **3** | **Inspección de Citas en Solución RAG** | El Reportante hace clic en `[1]`; `uiSlice` abre `CitationDrawer` y `<ReactMarkdown>` dibuja la tabla exacta del manual y sus páginas. | Los datos provienen del join entre `CITA_ORIENTACION`, `FRAGMENTO_DOCUMENTO`, `VERSION_DOCUMENTO` y `DOCUMENTO`. |
| **4** | **Retiro de Falso Positivo en Grupo IA** | El Analista intenta retirar el reporte de impresora sin escribir motivo; Zod exige mínimo $10$ caracteres. Al enviar, desaparece de los activos del grupo. | `VERSION_INCIDENCIA` conserva la fila con `estado = 'RETIRADA'`, `motivo_retiro` y `revisada_por`. |
| **5** | **Validación ISO 27005 en Matriz $5 \times 5$** | El botón *Validar* exige activo confirmado, celda seleccionada en la matriz (`P4 x I3 = ALTO`), `justificacion` y (si faltan citas) `justificacion_sin_fuentes`. | DRF persiste `EVALUACION_RIESGO` y `DECISION_RIESGO` dentro de `transaction.atomic()` y registra en `EVENTO_AUDITORIA`. |