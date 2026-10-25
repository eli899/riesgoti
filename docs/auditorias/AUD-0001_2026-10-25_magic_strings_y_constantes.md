# AUD-0001 — Auditoría de *magic strings*, números mágicos y textos hardcodeados

| Campo | Valor |
| :--- | :--- |
| **Fecha** | 2026-10-25 |
| **Skill de referencia** | `constants-no-magic-strings` (complementada por `naming-conventions-standard`, `typescript-advanced-contracts` y `react-router-dom-patterns`) |
| **Alcance** | `front/src/**` (147 archivos `.ts`/`.tsx`), `front/e2e/**`, `back/apps/**` (excluye `migrations/`) |
| **Tipo** | Auditoría de solo lectura. No se modificó código. La remediación debe ejecutarse por fases bajo `iterative-change-tracker`. |
| **Método** | Barrido con `grep`/`ripgrep` por patrones (comparaciones contra literales, uniones de tipos literales, `toast.*`, `z.*().min(n, '...')`, `required_permission = '...'`, `ValidationError({...: '...'})`, `pollingInterval`, `setTimeout`, `slice(0, n)`) y revisión manual de una muestra de cada categoría. Los conteos de textos de UI son aproximados (heurística). |

---

## 1. Resumen ejecutivo

El proyecto ya tiene una base correcta en dos áreas: las **rutas** (`src/constants/routes.constants.ts`) y los **textos de navegación** (`src/constants/navigation.constants.ts`) están centralizados con `as const`. Las etiquetas de estados viven en `src/lib/labels.ts` y algunos intervalos de *polling* ya son constantes con nombre.

El problema principal es que los **valores de dominio** (estados, roles, permisos, acciones de auditoría) solo existen como **uniones de tipos** (`'CERRADA' | ...`). No hay un objeto en tiempo de ejecución. Por eso cada comparación se escribe a mano (`status === 'CERRADA'`) en componentes, hooks y mappers. En el backend ocurre lo mismo: cada app vuelve a declarar sus propios `PENDIENTE = 'PENDIENTE'`, o compara directamente contra el literal.

| # | Categoría | Capa | Ocurrencias aprox. | Archivos | Severidad |
| :-- | :--- | :--- | --: | --: | :--- |
| F-01 | Estados de dominio comparados contra literales | Front | ~150 | 39 | **Alta** |
| F-02 | Códigos de permiso RBAC hardcodeados | Front | ~45 | 14 | **Alta** |
| F-03 | Códigos de rol (`'ADMIN'`, `'ANALISTA'`, `'REPORTANTE'`) | Front | ~19 | 6 | **Alta** |
| F-04 | Uniones literales duplicadas en `api.types.ts` y `domain.types.ts` | Front | 17 tipos × 2 | 2 | Media |
| F-05 | Endpoints REST literales en los servicios RTK Query | Front | 37 | 5 | Media |
| F-06 | Tags de caché RTK Query como strings | Front | ~60 | 4 | Media |
| F-07 | Textos de UI en JSX, `aria-*`, `title`/`placeholder` y `<PageMeta>` | Front | ~550 | ~85 | Media (volumen alto) |
| F-08 | Mensajes de *toast* y de bloqueo en hooks | Front | ~60 | 13 | Media |
| F-09 | Mensajes y límites de validación Zod | Front | ~45 | 8 | Media |
| F-10 | Números mágicos (límites de lista, longitudes, tiempos) | Front | ~15 | 10 | Baja |
| F-11 | Modos e identificadores de UI (`'all'`, `'queue'`, `'nuevas'`, `'root.server'`) | Front | ~25 | 8 | Baja |
| F-12 | Literales en la suite E2E | E2E | ~295 localizadores | 8 | Baja |
| B-01 | Estados redeclarados por app o usados inline | Back | ~40 decl. + 16 inline | 9 | **Alta** |
| B-02 | Códigos de permiso en vistas y en `permissions.py` | Back | ~25 | 7 | **Alta** |
| B-03 | Códigos de acción de auditoría (`EVENTO_AUDITORIA.accion`) inline | Back | ~36 | 8 | **Alta** |
| B-04 | Constantes duplicadas entre módulos (`CORRECTABLE_FIELDS`, `MAX_ERROR_CHARS`, `PROVIDER_TIMEOUT_SECONDS`, `TAG`) | Back | 9 nombres | 10 | Media |
| B-05 | Mensajes de error de negocio inline (`ValidationError`, `PermissionDenied`, `NotFound`) | Back | ~51 | 13 | Media |
| B-06 | Límites de longitud en serializers desalineados de Zod | Back | ~45 | 7 | Media |
| B-07 | Números mágicos en servicios (`[:300]`, `SERIE_DIAS = 30` local, `TASK_OPTIONS`) | Back | ~8 | 4 | Baja |

**Riesgo principal.** Los valores de F-01, F-02, B-01, B-02 y B-03 forman el **contrato** entre frontend, backend y `bd.sql`. Si un estado se renombra o se escribe mal (por ejemplo, `'RETIRADO'` en lugar de `'RETIRADA'`), TypeScript solo lo detecta cuando la variable comparada tiene el tipo literal correcto. Python no lo detecta nunca. Además, varias invariantes del dominio dependen de estos literales: INV-02 (`'FUNCIONO'` → `'CERRADA'`), INV-05 (`roleCode !== 'ADMIN'`) e INV-04 (`procesamiento='AUTORIZADO'`).

---

## 2. Hallazgos del Frontend

### F-01 — Estados de dominio comparados contra literales (Alta)

Literales más repetidos: `'ADMIN'` (15), `'PENDIENTE'` (11), `'CORREGIDA'` (11), `'ERROR'` (9), `'PROCESANDO'` (7), `'FUNCIONO'` (7), `'CERRADA'` (7), `'VALIDADA'`, `'RETIRADA'`, `'RECHAZADA'`, `'EN_ATENCION'` (5 c/u), `'PENDIENTE_ATENCION_HUMANA'`, `'CONFIRMADA'` (4 c/u).

Archivos más afectados:

| Archivo | Comparaciones |
| :--- | --: |
| `components/incidents/OperationalStrategicStatusAside.tsx` | 10 |
| `hooks/incidents/useReporterIncidents.ts` | 9 |
| `components/admin/DocumentVersionsTable.tsx` | 9 (incluye estados derivados `'DISPONIBLE'`/`'INACTIVO'` que no existen en ningún tipo) |
| `components/risk/RiskDecisionDialogView.tsx` | 8 |
| `components/staff/PriorAISuggestionBadge.tsx` | 7 |
| `hooks/incidents/useAnalystSummary.ts` | 6 |
| `schemas/riskEvaluation.schema.ts`, `hooks/incidents/useOperationalGuidance.ts`, `components/incidents/OperationalGuidanceSection.tsx` | 5 c/u |
| Otros 30 archivos | 1–4 c/u |

Ejemplos:

```ts
// components/incidents/OperationalStrategicStatusAside.tsx:34
const running = strategic === 'PENDIENTE' || strategic === 'PROCESANDO'
// hooks/incidents/useReporterIncidents.ts:36
awaitingHuman: incidents.filter((i) => i.status === 'PENDIENTE_ATENCION_HUMANA' || i.status === 'EN_ATENCION')
// components/admin/DocumentVersionsTable.tsx:14 — estado derivado sin tipo
v.processing === 'AUTORIZADO' ? (v.active ? 'DISPONIBLE' : 'INACTIVO') : v.processing
```

**Corrección.** Crear `src/constants/domain.constants.ts` con un objeto `as const` por cada enumeración y derivar el tipo de ese objeto. Las agrupaciones que se repiten pasan a constantes con nombre: `AI_TASK_RUNNING_STATUSES`, `INCIDENT_HUMAN_QUEUE_STATUSES`, `DOCUMENT_DISPLAY_STATE`.

### F-02 — Códigos de permiso RBAC hardcodeados (Alta)

`PermissionCode` es solo un tipo (`types/domain.types.ts:8-23`). Los códigos se escriben a mano en:

- `pages/admin/AdminPortal.tsx:28-38` (11) y `pages/staff/AnalystPortal.tsx:23-27` (5): permisos de cada ruta del portal.
- `schemas/role.schema.ts:4-13` (10): regla SoD que prohíbe a ADMIN tener permisos de riesgo. Duplica `PERMISOS_PROHIBIDOS_ADMIN` del backend.
- Hooks: `useAssetCatalogManager.ts:13`, `useOperationalGuidance.ts:50`, `useReporterIncidents.ts:57`, `useSupportCaseAttention.ts:49`, `useRiskClusterReview.ts:36`, `useRiskDecisionFlow.ts:31`, `useRiskProposalsQueue.ts:40`, `useRiskVersionCompare.ts:25`, `pages/admin/MatrixPage.tsx:18`.
- `features/auth/authSlice.ts:63`: `canEvaluateRisk` (INV-05).
- `lib/labels.ts` (`PERMISSION_GROUPS`): 15 códigos más.

**Corrección.** Crear `src/constants/rbac.constants.ts` con `PERMISSIONS` (`as const`), `ADMIN_FORBIDDEN_PERMISSIONS` y `PermissionCode` derivado del objeto. `PERMISSION_GROUPS` debe referenciar `PERMISSIONS.*`.

### F-03 — Códigos de rol (Alta)

- `App.tsx:24`: selección del portal con un ternario de tres literales.
- `features/auth/authSlice.ts:63`: `roleCode !== 'ADMIN'` (INV-05).
- `components/risk/VersionHistorySection.tsx:34`: `v.origin === 'ANALISTA'`. Aquí `'ANALISTA'` es el **origen** de una versión, no un rol: es otra enumeración (`RISK_VERSION_ORIGIN`) que hoy no tiene tipo propio.
- `hooks/admin/useUserRoleManager.ts`, `components/admin/RolePermissionsMatrixView.tsx`, `components/auth/AccountProfileArticle.tsx`, `pages/admin/UsersPage.tsx`.
- `constants/navigation.constants.ts`: `PORTAL_NAV_ARIA_LABELS` usa claves de rol escritas a mano. Debe tiparse como `Record<RoleCode, string>`.

**Corrección.** Agregar `ROLES` en `rbac.constants.ts`. Reemplazar el ternario de `App.tsx` por un mapa `Record<RoleCode, ComponentType>`.

### F-04 — Uniones literales duplicadas en `api.types.ts` y `domain.types.ts` (Media)

Hay 17 enumeraciones (`RolCodigoDto`/`RoleCode`, `EstadoIncidenciaDto`/`IncidentStatus`, `ProcesamientoDocumentoDto`/`DocumentProcessingStatus`, `EstadoTareaDto`/`AITaskStatus`, etc.) declaradas dos veces con los mismos valores. A esto se suman `z.enum(['OPERATIVA', 'ESTRATEGICA', 'AMBAS'])` en `schemas/document.schema.ts:16` y `AuditTargetKind`/`RiskRegistryCategory`/`RiskInbox` en `domain.types.ts`.

**Corrección.** Ambos archivos deben importar los tipos derivados de `domain.constants.ts`. Zod debe usar `z.enum(objectValues(DOCUMENT_PURPOSES))` o `z.nativeEnum`.

### F-05 — Endpoints REST literales (Media)

Hay 37 URLs en `app/api/adminApi.ts` (17), `incidentsApi.ts` (8), `risksApi.ts` (7), `authApi.ts` (2) y `baseApi.ts` (3). `baseApi.ts:35` compara `url.startsWith('auth/login')` y `'auth/refresh'`: si la ruta cambia, la renovación del token falla sin dar error. `'POST'`, `'PATCH'` y `'PUT'` aparecen 23 veces.

**Corrección.** Crear `src/constants/api.constants.ts` con `API_ENDPOINTS` (estáticos y constructores `(id: number) => \`incidents/${id}/attention/\``) y `HTTP_METHODS`. Seguir el mismo patrón que `ROUTES`.

### F-06 — Tags de caché RTK Query (Media)

`API_TAG_TYPES` ya existe (`baseApi.ts:5`), pero es un *array*. Los `providesTags` e `invalidatesTags` siguen escribiendo `'RiskVersion'` (10), `'DocumentVersion'` (10), `'Incident'` (9), `'Role'` (7), `'User'` (6), etc. Si un tag se escribe mal, la caché deja de invalidarse sin aviso.

**Corrección.** Agregar `API_TAGS = { AI_CONFIG: 'AIConfig', ... } as const` y derivar de él `API_TAG_TYPES = Object.values(API_TAGS)`.

### F-07 — Textos de UI hardcodeados (Media, volumen alto)

Hay ~550 textos visibles en ~85 archivos: contenido JSX, `aria-label`/`aria-description`, `title`, `placeholder`, `label`, `description` y `alt`, además de los `title`/`description` de `<PageMeta>`.

| Archivo | Textos aprox. |
| :--- | --: |
| `pages/risk/RiskWorkspacePage.tsx` | 19 |
| `components/admin/AIConfigFormView.tsx` | 18 |
| `components/admin/DocumentVersionsTable.tsx` | 18 |
| `pages/staff/SupportDetailPage.tsx` | 17 |
| `pages/risk/RiskDetailPage.tsx` | 17 |
| `pages/staff/AnalystSummaryPage.tsx` | 15 |
| `components/reporter/CreateIncidentFormModal.tsx` | 15 |
| `pages/admin/AuditPage.tsx`, `components/ui.tsx`, `components/risk/GroupingConfirmationFieldset.tsx`, `components/admin/RolePermissionsMatrixView.tsx` | 14 c/u |
| Otros ~75 archivos | 1–13 c/u |

También hay textos de respaldo en mappers: `'Usuario eliminado'` (`mappers/admin.mapper.ts:376`), `'Usuario no disponible'` (`mappers/common.mapper.ts:12`) y `'Activo fuera del catálogo'` (`hooks/incidents/useAnalystSummary.ts:45`, repetido dos veces en la misma línea). Las paginaciones de `components/ui.tsx` (`'Anterior'`, `'Siguiente'`) y `lib/format.ts:14` (`'Menos de 1 h'`, `' h'`, `' d'`) se repiten en toda la app.

**Corrección.** Centralizar los textos por módulo en `src/constants/copy/{common,incidents,risks,admin,auth}.constants.ts` (`as const`), con claves semánticas (`INCIDENT_COPY.DETAIL.PAGE_TITLE`). Prioridad: (1) textos repetidos y de componentes compartidos (`ui.tsx`, `AsyncContent`, `CitationDrawer`), (2) `<PageMeta>` y `aria-*`, (3) el resto por módulo. No se propone i18n: la app tiene un solo idioma. Un objeto de textos centralizado deja preparado el camino si se necesita en el futuro.

### F-08 — Mensajes de *toast* y de bloqueo en hooks (Media)

Hay ~60 mensajes de éxito y error en 13 hooks. Por ejemplo, `useSupportCaseAttention.ts:55-82` (7), `useUserRoleManager.ts:48-96` (incluye reglas de negocio: «Debe quedar al menos un administrador activo.»), `useRiskDecisionFlow.ts:15-17` y `37-39`, `useOperationalGuidance.ts:42-64` y `useKnowledgeBaseManager.ts:26-67`. Algunos mensajes describen invariantes (INV-02: «El análisis de riesgo continúa de forma independiente»). Conviene que sean revisables en un único lugar.

`useKnowledgeBaseManager.ts:26-30` y `useRiskDecisionFlow.ts:15-17` ya usan mapas por acción. Ese es el patrón que debe generalizarse.

**Corrección.** Crear `src/constants/messages/*.messages.constants.ts`, o una sección `TOASTS` dentro de cada archivo `copy/*`.

### F-09 — Mensajes y límites de validación Zod (Media)

Hay ~45 mensajes en `schemas/aiConfig.schema.ts` (11), `user.schema.ts` (7), `riskEvaluation.schema.ts` (7), `incident.schema.ts`, `document.schema.ts` y `asset.schema.ts` (5 c/u). Los límites numéricos (`256`/`2048`/`512`/`15` en `aiConfig`, `5`/`20`/`10`/`10000` en `incident`, `50`/`150`/`255`/`2000`) se repiten **dentro del texto del mensaje**: `.min(20, 'Describa qué ocurrió con al menos 20 caracteres')`. Si se cambia el número, el mensaje queda desactualizado.

Puntos ya correctos: `MAX_PDF_SIZE_BYTES`, `MAX_EVIDENCE_SIZE_BYTES`, `MAX_EVIDENCES_PER_INCIDENT`, `MIN_JUSTIFICATION` y `MIN_OBSERVATION`. Aun así, `'25 MB'` y `'10 MB'` siguen literales en los mensajes, y `MIN_OBSERVATION = 10` aparece como `10` en el texto.

**Corrección.** Crear `src/constants/validation.constants.ts` con `FIELD_LIMITS` alineado 1:1 con los `max_length` de los serializers DRF y de `bd.sql`. Los mensajes deben interpolar el límite: `` `al menos ${FIELD_LIMITS.INCIDENT_DESCRIPTION_MIN} caracteres` ``.

### F-10 — Números mágicos (Baja)

| Ubicación | Valor | Propuesta |
| :--- | :--- | :--- |
| `pages/admin/AdminDashboardPage.tsx:55`, `components/staff/PendingProposalsArticle.tsx:13`, `hooks/incidents/useAnalystSummary.ts:48` | `slice(0, 5)` | `DASHBOARD_LIMITS.TOP_ITEMS` |
| `pages/reporter/ReporterHomePage.tsx:20` | `slice(0, 4)` | `DASHBOARD_LIMITS.RECENT_INCIDENTS` |
| `hooks/incidents/useAnalystSummary.ts:40` | `90` días | `ANALYST_SUMMARY.MAX_SERIES_DAYS` |
| `components/admin/PublishMatrixDialog.tsx:32` | `3` caracteres | `FIELD_LIMITS.MATRIX_NAME_MIN` |
| `components/ui.tsx:290` | `setTimeout(..., 10)` | `UI_TIMINGS.DIALOG_FOCUS_DELAY_MS` |
| `components/ui.tsx:364-365` | etapa `5` del *stepper* | `PROCESSING_STAGES.length - 1` |
| `lib/download.ts:11` | `1000` ms | `UI_TIMINGS.OBJECT_URL_REVOKE_MS` |
| `lib/format.ts:14` | `24` h | `HOURS_PER_DAY` |
| Matriz 5×5 (`InteractiveRiskMatrix5x5Grid.tsx`, `Matrix5x5EditorGrid.tsx`, `riskEvaluation.schema.ts:42` `.min(1).max(5)`) | `5` | `RISK_MATRIX.SIZE` (el backend ya tiene `MATRIX_SIZE = 5`) |

Las constantes de *polling* (`DOCUMENT_POLLING_MS`, `GUIDANCE_POLLING_MS`, `STRATEGIC_POLLING_MS`, `REPORTER_LIST_POLLING_MS`, `QUEUE_POLLING_MS`) ya tienen nombre, pero están repartidas en 4 hooks. Conviene reunirlas en `POLLING_INTERVALS_MS` dentro de `src/constants/timing.constants.ts`.

### F-11 — Modos e identificadores de UI (Baja)

- `'all'` (9) en `components/reporter/IncidentsTable.tsx` y filtros similares → `FILTER_ALL`.
- `'queue'` (11) en `hooks/incidents/useSupportQueue.ts` y su página → `SUPPORT_VIEW_MODES`.
- `'nuevas'`, `'pendientes_revision'`, `'rechazadas'`, `'decididas'` (`hooks/risks/useRiskProposalsQueue.ts:11-13`) deben coincidir con `BANDEJAS` del backend (`apps/risks/views.py:31`) → `RISK_INBOXES`.
- `'VALIDADO'`, `'PENDIENTE'`, `'RECHAZADO'` (`hooks/risks/useRiskRegistryAndDashboard.ts:8-10`) → `RISK_REGISTRY_CATEGORIES`.
- `'root.server'` (`components/auth/LoginForm.tsx:38`, `components/forms/applyServerErrors.ts:18`) y `'non_field_errors'`/`'detail'` (`lib/apiErrors.ts`) → `FORM_ERROR_KEYS`.
- `'probability'`, `'impact'`, `'matrix'`, `'authorize'`: modos internos de la evaluación y de la base documental.

### F-12 — Suite E2E (Baja)

Las rutas ya se importan de `routes.constants.ts`. Sin embargo, ~295 localizadores (`getByRole(..., { name: '...' })`, `getByText('...')`, `getByLabel('...')`) repiten textos de la UI. Cuando F-07 esté resuelto, los *Page Objects* deben importar los mismos objetos `copy/*`. Así un cambio de texto no rompe la suite. Las cuentas de prueba y los estados esperados deben reutilizar `ROLES`, `domain.constants.ts` y las credenciales del *seed* centralizadas.

---

## 3. Hallazgos del Backend

### B-01 — Estados redeclarados por app o usados inline (Alta)

No existe ningún `TextChoices`/`StrEnum`. Cada módulo declara su propia copia:

| Valor | Declaraciones independientes |
| :--- | :--- |
| `'PENDIENTE'` | `incidents/services.py:36` (`TAREA_PENDIENTE`), `risks/services.py:67` (`TAREA_PENDIENTE`), `risks/services.py:518` (`ESTADO_PENDIENTE`) |
| `'AUTORIZADO'` | `documents_rag/services.py:26`, `documents_rag/faiss_store.py:23` (`PROCESAMIENTO_AUTORIZADO`) |
| `'SUGERENCIA'` | `incidents/services.py:40`, `risks/services.py:68` |
| `'OPERATIVA'/'ESTRATEGICA'/'AMBAS'` | Finalidad: `documents_rag/serializers.py:5`, `documents_rag/faiss_store.py:24-25`. Rama de tarea: `incidents/services.py:34-35` |
| `'OPENAI'/'GEMINI'/...` (proveedores) | `ai_config/serializers.py:5-6` (`LLM_PROVIDERS`, `EMBEDDING_PROVIDERS`), duplica `ProveedorLlmDto` del front |
| `'VALIDADA'/'CORREGIDA'/'RECHAZADA'` | `risks/serializers.py:5` + literales inline |

Usos inline sin constante (16):

- `ai_config/services.py:37`: `procesamiento='AUTORIZADO'`. Es el filtro del *retriever* (INV-04).
- `documents_rag/serializers.py:78`: `('PENDIENTE_AUTORIZACION', 'AUTORIZADO')`.
- `risks/management/commands/requeue_ai_tasks.py:24-31`: `'PROCESANDO'`, `'PENDIENTE'` (4).
- `risks/serializers.py:66-71`, `205`: `'RECHAZADA'`, `'CORREGIDA'`, `'RETIRADA'`.
- `risks/services.py:240`: `TaskLifecycle.claim(tarea_id, 'OPERATIVA')`, aunque `RAMA_OPERATIVA` ya existe en otro módulo.
- `risks/services.py:466`: `'NUEVO_RIESGO'`.
- `risks/views.py:56`: `estado='RECHAZADA'`.
- `risks/llm_schemas.py:15`: `Literal['SUGERENCIA', 'SENSIBLE']`.

**Corrección.** Crear `apps/<app>/constants.py` con clases `models.TextChoices` (o `enum.StrEnum`) por tabla: `EstadoIncidencia`, `EstadoTareaIA`, `RamaTareaIA`, `ProcesamientoDocumento`, `FinalidadDocumento`, `EstadoVersionRiesgo`, `AccionDecisionRiesgo`, `EstadoVinculo`, `TipoOrientacion`, `ResultadoOrientacion`, `OrigenVersionRiesgo`, `CasoAgrupacion` y `NivelRiesgo`. Los valores provienen de los `CHECK` de `docs/bd.sql`. **Restricción de `bd.sql`:** en la primera fase no se asignan como `choices=` en los campos del modelo. Así se evitan migraciones `AlterField` (no generan DDL, pero ensucian el historial) y no se toca el esquema. Se usan solo en servicios, selectores, serializers (`ChoiceField(choices=EstadoX.choices)`) y comandos.

### B-02 — Códigos de permiso (Alta)

- `accounts/permissions.py:5-20`: `ADMIN_ROLE_CODE` y la lista SoD.
- `accounts/management/commands/seed_riesgoti.py:29-60`: `PERMISOS`, `PERMISOS_PROHIBIDOS_ADMIN`, `ROL_PERMISOS`. Es una tercera copia de la regla SoD (las otras están en `permissions.py` y en `front/src/schemas/role.schema.ts`).
- `required_permission = '...'` en `accounts/views.py:84,124,156`, `ai_config/views.py:22,43`, `assets_matrix/views.py:18,105`, `audit/views.py:37`, `documents_rag/views.py:27` y `risks/views.py:30,161`.
- Mapas acción→permiso en `incidents/views.py:41-42,69,91-94,120` y `risks/views.py:77-81`.

**Corrección.** Crear `apps/accounts/rbac.py` con `class Permiso(StrEnum)`, `class RolCodigo(StrEnum)` y `PERMISOS_PROHIBIDOS_ADMIN: frozenset[Permiso]`, como **única** fuente consumida por vistas, `permissions.py` y el *seed*. El catálogo debe coincidir con `PERMISSIONS` del front.

### B-03 — Acciones de auditoría (Alta)

~36 literales (`'CREAR_USUARIO'`, `'EVALUAR_RIESGO'`, `'DECIDIR_VERSION_RIESGO'`, ...) en `accounts/services.py` (9), `risks/services.py` (7), `documents_rag/services.py` (6), `incidents/services.py` (5), `assets_matrix/views.py` (4), `ai_config/services.py` (2), `audit/mixins.py` (2) y `assets_matrix/services.py` (1). El front los repite en `AUDIT_ACTION_LABEL` (`lib/labels.ts`), tipado como `Record<string, string>`. Si se escribe mal una acción, el evento queda sin etiqueta y no hay aviso. Es una brecha de trazabilidad para ISO 27001 (A.8.15).

**Corrección.** Crear `apps/audit/constants.py` con `class AccionAuditoria(StrEnum)` y `class EntidadAuditoria(StrEnum)`. En el front, `AUDIT_ACTIONS` debe ser `as const` y `AUDIT_ACTION_LABEL` debe tiparse como `Record<AuditAction, string>`.

### B-04 — Constantes duplicadas entre módulos (Media)

| Nombre | Ubicaciones |
| :--- | :--- |
| `CORRECTABLE_FIELDS` | `risks/serializers.py:6`, `risks/services.py:524` (valores idénticos) |
| `MAX_ERROR_CHARS = 2000` | `documents_rag/services.py:30`, `risks/services.py:80` |
| `PROVIDER_TIMEOUT_SECONDS` | `ai_config/services.py:17` (20), `documents_rag/embeddings.py:11` (60): mismo nombre con valores distintos |
| `TAG` (OpenAPI) | `documents_rag/views.py:18`, `incidents/views.py:40`, `risks/views.py:28` |
| `COPY_BLOCK_BYTES` / `HASH_BLOCK_BYTES` (`64 * 1024`) | `incidents/services.py:42`, `documents_rag/services.py:29` |
| `MIN_JUSTIFICACION = 15` y `MIN_JUSTIFICACION_CAMBIO_ACTIVO = 10` | `risks/services.py:522-523`, con equivalentes en `front/src/schemas/riskEvaluation.schema.ts:3-4` y `min_length=10` inline en `risks/serializers.py:30,66` |

**Corrección.** Llevar cada constante al `constants.py` de su app, o a `config/constants.py` si es transversal (tamaños de bloque, longitud máxima de errores, *tags* OpenAPI en `OPENAPI_TAGS`). Renombrar los *timeouts*: `LLM_PING_TIMEOUT_SECONDS` y `EMBEDDING_TIMEOUT_SECONDS`.

### B-05 — Mensajes de error de negocio inline (Media)

Hay ~51 mensajes en `ValidationError`/`PermissionDenied`/`NotFound`: `risks/services.py` (12), `incidents/services.py` (8), `accounts/services.py` (8), `documents_rag/services.py` (7) y otros. Ejemplos: `incidents/services.py:196` («Solo el reportante de la incidencia puede confirmar la orientación.»), `risks/services.py:558` («Esta versión de riesgo ya cuenta con una decisión registrada y es inmutable.»). No hay códigos de error semánticos: el front solo puede distinguir errores por `status` (`lib/apiErrors.ts:12-15`) y por el nombre del campo.

**Corrección.** Crear `apps/<app>/messages.py` con constantes o plantillas (`str.format`). Opcionalmente, agregar `apps/core/error_codes.py` (`ErrorCode(StrEnum)`: `ALREADY_DECIDED`, `NOT_ASSIGNED_ANALYST`, `INVALID_STATE_TRANSITION`, ...) y exponerlo con `default_code`. Esto permitiría al front mapear mensajes por código en lugar de por texto.

### B-06 — Límites de longitud desalineados (Media)

Los serializers declaran `max_length`/`min_length` inline (~45: `150`, `128`, `100`, `50`, `255`, `512`, `10000`, `2000`, `8`, `5`, `20`). Estos límites deben coincidir con `bd.sql` y con Zod (F-09). Hoy coinciden «a mano». Ejemplo: `incidents/serializers.py:24-25` (`5`, `20`) ↔ `front/src/schemas/incident.schema.ts:12-13`.

**Corrección.** Crear `apps/<app>/constants.py` → `LIMITES_CAMPO` (o leer `Model._meta.get_field(...).max_length` para los `max_length` derivados de `bd.sql`). Documentar la tabla de equivalencias con `FIELD_LIMITS` del front, o publicarla vía OpenAPI (`drf-spectacular` ya expone `maxLength`).

### B-07 — Números mágicos en servicios (Baja)

- `risks/services.py:168-169`: `[:300]` → `MAX_RESUMEN_CHARS`.
- `risks/services.py:732`: `SERIE_DIAS = 30` local dentro de una función → constante de módulo.
- `risks/services.py:466,470`: `1.0`/`0.0` como similitud de origen → `SIMILITUD_ORIGEN`.
- `risks/tasks.py:7`: `soft_time_limit: 180`, `time_limit: 240` → `CELERY_TASK_SOFT_LIMIT_SECONDS`, `CELERY_TASK_HARD_LIMIT_SECONDS` (en `settings`, configurables por entorno).
- `risks/services.py:81`: `GROUPING_LOCK_KEY = 270050`. Tiene nombre y comentario, así que es correcto.

---

## 4. Exclusiones (no son *magic strings*)

- Clases Tailwind (ya gobernadas por `reusable-design-tokens` y el guardián de tokens de CHG-0023).
- `db_table`/`db_column` y `source='rol.codigo'` de serializers: son nombres de esquema fijados por `bd.sql`.
- Migraciones, *fixtures* y datos del *seed* (`seed_*.py`, `seed_documents.py`): son datos, no lógica. Se excluyen salvo las copias de RBAC (B-02).
- *Prompts* del LLM (`OPERATIONAL_SYSTEM_PROMPT`, `STRATEGIC_SYSTEM_PROMPT`) y textos de `SIN_RESPALDO`/`SENSIBLE`: ya son constantes de módulo con nombre.
- Comentarios y *docstrings* que citan valores (`permissions.py:51`, `incidents/services.py:209`).
- `0`, `1`, `-1`, `length > 0`, `* 100` (porcentajes) y `split(' ')`.

---

## 5. Arquitectura objetivo

### Frontend (`front/src/constants/`)

```
constants/
├── routes.constants.ts         (existente)
├── navigation.constants.ts     (existente; tipar PORTAL_NAV_ARIA_LABELS con RoleCode)
├── domain.constants.ts         F-01, F-04, F-11 — enumeraciones de bd.sql + agrupaciones
├── rbac.constants.ts           F-02, F-03 — ROLES, PERMISSIONS, ADMIN_FORBIDDEN_PERMISSIONS
├── audit.constants.ts          B-03 (espejo) — AUDIT_ACTIONS, AUDIT_TARGETS
├── api.constants.ts            F-05, F-06 — API_ENDPOINTS, API_TAGS, HTTP_METHODS
├── timing.constants.ts         F-10 — POLLING_INTERVALS_MS, UI_TIMINGS
├── validation.constants.ts     F-09 — FIELD_LIMITS, FILE_LIMITS
├── ui.constants.ts             F-10, F-11 — RISK_MATRIX, DASHBOARD_LIMITS, FILTER_ALL, FORM_ERROR_KEYS
├── labels.constants.ts         migración de lib/labels.ts (Record<Enum, string>)
└── copy/
    ├── common.constants.ts     F-07 — paginación, estados vacíos, botones comunes
    ├── incidents.constants.ts  F-07, F-08
    ├── risks.constants.ts
    ├── admin.constants.ts
    └── auth.constants.ts
```

Patrón obligatorio:

```ts
export const INCIDENT_STATUS = {
  REGISTRADA: 'REGISTRADA',
  EN_ANALISIS: 'EN_ANALISIS',
  PENDIENTE_ATENCION_HUMANA: 'PENDIENTE_ATENCION_HUMANA',
  EN_ATENCION: 'EN_ATENCION',
  CERRADA: 'CERRADA',
} as const

export type IncidentStatus = (typeof INCIDENT_STATUS)[keyof typeof INCIDENT_STATUS]

export const INCIDENT_HUMAN_QUEUE_STATUSES: readonly IncidentStatus[] = [
  INCIDENT_STATUS.PENDIENTE_ATENCION_HUMANA,
  INCIDENT_STATUS.EN_ATENCION,
]
```

Como los valores de estos objetos son idénticos en la red y en el dominio (solo cambian los nombres de los campos), `api.types.ts` y `domain.types.ts` pueden importar el mismo tipo. Si en el futuro un valor diverge, se traduce en el *mapper* correspondiente (Escenario A de la política dual).

### Backend (`back/apps/*/`)

```
apps/accounts/rbac.py            B-02 — RolCodigo, Permiso, PERMISOS_PROHIBIDOS_ADMIN
apps/audit/constants.py          B-03 — AccionAuditoria, EntidadAuditoria
apps/incidents/constants.py      B-01 — EstadoIncidencia, RamaTareaIA, EstadoTareaIA, TipoOrientacion, ResultadoOrientacion
apps/documents_rag/constants.py  B-01 — ProcesamientoDocumento, FinalidadDocumento (+ FINALIDADES_OPERATIVAS/ESTRATEGICAS)
apps/risks/constants.py          B-01, B-04 — EstadoVersionRiesgo, AccionDecisionRiesgo, EstadoVinculo, OrigenVersionRiesgo, CasoAgrupacion, CORRECTABLE_FIELDS, MIN_JUSTIFICACION
apps/ai_config/constants.py      B-01 — ProveedorLlm, ProveedorEmbedding, timeouts
apps/<app>/messages.py           B-05
config/constants.py              B-04 — OPENAPI_TAGS, IO_BLOCK_BYTES, MAX_ERROR_CHARS
```

---

## 6. Plan de remediación por fases

Cada fase es una iteración independiente bajo `iterative-change-tracker` (análisis → aprobación → ejecución + `CHG-XXXX` + Brain). Las fases 1 a 3 no cambian el contrato REST (Escenario A: refactor interno sin efecto externo).

| Fase | Contenido | Hallazgos | Riesgo | Verificación |
| :--- | :--- | :--- | :--- | :--- |
| **1 — Contrato de dominio** | `domain.constants.ts`, `rbac.constants.ts`, `audit.constants.ts`. Derivar tipos y reemplazar comparaciones en los 39 archivos. Backend: `rbac.py`, `audit/constants.py`, `constants.py` por app y eliminación de redeclaraciones. | F-01, F-02, F-03, F-04, B-01, B-02, B-03 | Medio (toca invariantes INV-02/04/05) | `pnpm typecheck`, `pnpm lint`, `pytest`, `pnpm test:e2e` completo |
| **2 — Infraestructura de red y tiempos** | `api.constants.ts` (endpoints, tags, métodos), `timing.constants.ts`, `ui.constants.ts`, constantes duplicadas del backend. | F-05, F-06, F-10, F-11, B-04, B-07 | Bajo | `typecheck`, `pytest`, E2E de humo |
| **3 — Validación alineada** | `validation.constants.ts`, `FIELD_LIMITS` backend, mensajes Zod interpolados, `messages.py`. | F-09, B-05, B-06 | Bajo | Pruebas de esquemas y serializers |
| **4 — Textos de UI** | `copy/*.constants.ts` módulo por módulo (common → auth → reporter → incidents/staff → risks → admin), *toasts*, `<PageMeta>`, `aria-*`, migración de `lib/labels.ts`. | F-07, F-08 | Bajo (sin cambios de lógica), volumen alto | E2E completo y capturas de manuales sin cambios visuales |
| **5 — E2E y guardián** | *Page Objects* consumiendo `copy/*` y `domain.constants.ts`. Regla de lint que impida regresiones. | F-12 | Bajo | `pnpm lint` falla ante un literal de estado nuevo |

### Guardián propuesto (fase 5)

- **ESLint** (`no-restricted-syntax`): prohibir `BinaryExpression[operator=/^[!=]==$/] > Literal[value=/^[A-Z][A-Z_]{2,}$/]` y `Literal[value=/^(incidencia|atencion|riesgo|activo|documento|matriz|usuario|rol)\.[a-z_]+$/]` fuera de `src/constants/**`.
- **Ruff**/prueba de arquitectura en `pytest`: buscar con `ast` comparaciones o *kwargs* `estado=`/`procesamiento=`/`accion=` contra `str` literales fuera de `constants.py`/`rbac.py`/`migrations/`.
- **Prueba de contrato**: un test de `pytest` que exporte las enumeraciones del backend y un test de Vitest/Playwright que verifique que `domain.constants.ts` y `rbac.constants.ts` contienen los mismos valores (o generación desde el esquema OpenAPI con `drf-spectacular` + `ChoiceField`).

---

## 7. Métricas de seguimiento

| Métrica | Línea base (2026-10-25) | Meta |
| :--- | --: | --: |
| Comparaciones `=== 'LITERAL_MAYÚSCULAS'` fuera de `constants/` (front) | ~150 | 0 |
| Literales de permiso fuera de `rbac.constants.ts` (front) | ~45 | 0 |
| Literales de estado/acción inline o redeclarados (back) | ~90 | 0 |
| Fuentes de la regla SoD (ADMIN sin permisos de riesgo) | 3 | 2 (1 back + 1 front, verificadas por prueba de contrato) |
| Archivos con textos de UI hardcodeados | ~85 | 0 en `components/**` y `pages/**` |
| Mensajes Zod con el límite numérico repetido en el texto | ~20 | 0 |
