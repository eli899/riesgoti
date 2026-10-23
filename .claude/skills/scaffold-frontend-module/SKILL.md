---
name: scaffold-frontend-module
description: Plantilla estándar de Clean Architecture para construir módulos en el frontend (React + TypeScript + RTK Query + Mappers/Adapters + Custom Hooks + Zod + HTML5 Semántico/SEO + Tailwind CSS) sobre la base visual de Figma.
---

# Skill: `scaffold-frontend-module` (Construcción Estandarizada de Módulos Frontend con Clean Architecture)

## Cuándo Usar esta Skill
Invoca esta skill al construir o conectar módulos en `front/src/` para los portales de **Reportante**, **Analista de Seguridad/Riesgos** y **Administrador**.
Complementa siempre esta skill con las skills técnicas auto-instaladas: `react-best-practices`, `composition-patterns`, `typescript-advanced-types`, `tailwind-css-patterns`, `accessibility` y `seo`.

---

## Estructura Estándar de Clean Architecture en `front/src/`
```text
front/src/
├── app/
│   ├── store.ts                 # configureStore + setupListeners
│   ├── hooks.ts                 # useAppDispatch, useAppSelector
│   └── router.tsx               # createBrowserRouter + ProtectedRoute RBAC
├── services/                    # CAPA 1: Servicios API (RTK Query puro)
│   ├── baseApi.ts               # createApi + mutex refresh JWT
│   ├── authApi.ts
│   ├── adminApi.ts
│   ├── incidentsApi.ts
│   └── risksApi.ts
├── mappers/                     # CAPA 2: Mappers / Adapters (Desacoplamiento API <-> Dominio)
│   ├── auth.mapper.ts           # fromApiToDomain / fromDomainToApi
│   ├── admin.mapper.ts
│   ├── incidents.mapper.ts
│   └── risks.mapper.ts
├── hooks/                       # CAPA 3: Custom Hooks Especializados (Casos de Uso)
│   ├── useAuthSession.ts
│   ├── useAIConfigManager.ts
│   ├── useKnowledgeBaseManager.ts
│   ├── useMatrixConfigManager.ts
│   ├── useIncidentGuidance.ts
│   ├── useAnalystQueue.ts
│   ├── useRiskWorkspace.ts
│   └── useRiskDashboard.ts
├── schemas/                     # Validación de Contratos de Formulario (Zod)
│   ├── aiConfig.schema.ts
│   ├── documentUpload.schema.ts
│   ├── incidentReport.schema.ts
│   └── riskEvaluation.schema.ts
├── types/
│   ├── api.types.ts             # DTOs crudos de red (snake_case, espejo de OutputSerializer)
│   └── domain.types.ts          # Modelos de dominio limpios para UI (camelCase)
├── components/                  # CAPA 5: Componentes UI Presentacionales Puros + SEO
│   ├── seo/
│   │   └── PageMeta.tsx         # Gestión declarativa de document.title y meta description
│   ├── shared/
│   │   ├── CitationDrawer.tsx   # <aside aria-label="..."> con fragmentos normativos
│   │   └── ProtectedRoute.tsx   # Guard RBAC + SoD
│   └── <module>/
│       └── *View / *Card / *Form / *Table.tsx # Reciben exclusivamente props puras de dominio
└── pages/                       # CAPA 4: Páginas / Contenedores de Vista
    ├── reporter/
    ├── analyst/
    └── admin/
```

---

## Reglas de Implementación Paso a Paso (`/agents/rules/clean_code_and_architecture.md`)

### 1. Cero Llamadas a API en Páginas o Componentes (Flujo Obligatorio de 5 Capas)
- **Prohibido** importar `fetch`, `axios` o hooks crudos de `services/*Api.ts` directamente dentro de `src/components/**` o mezclar lógica HTTP en el JSX de `src/pages/**`.
- El flujo obligatorio para toda operación de lectura o escritura es:
  1. **Servicio API (`src/services/*Api.ts`):** Define endpoints RTK Query tipados con `ApiDTO` (`src/types/api.types.ts`) e invalidación de tags (`['AIConfig', 'DocumentVersion', 'Incident', 'OperationalGuidance', 'RiskVersion', 'RiskDashboard', 'MatrixConfig']`).
  2. **Mapper / Adapter (`src/mappers/*.mapper.ts`):** Expone funciones puras y deterministas `fromApiToDomain(dto): DomainModel` y `fromDomainToApi(formValues): ApiInputDTO`. Si el backend cambia el nombre de una propiedad en un ajuste aislado (Escenario A), solo se actualiza el Mapper sin tocar ningún componente React.
  3. **Custom Hook / Caso de Uso (`src/hooks/use*.ts`):** Invoca internamente el hook de RTK Query, aplica el Mapper con `useMemo`, controla el polling condicional (`pollingInterval: 3000` para documentos `'PROCESANDO'`; `pollingInterval: 2500` para orientación operativa con `tareaIa.estado !== 'COMPLETADA'`) y expone métodos de negocio limpios a la Página.
  4. **Página Contenedora (`src/pages/**/*Page.tsx`):** Invoca el Custom Hook, configura `<PageMeta />`, define la estructura HTML5 semántica (`<main>`, `<header>`, `<h1>`, `<section>`) y pasa datos de dominio + callbacks a los Componentes Presentacionales.
  5. **Componente UI Presentacional (`src/components/**/*.tsx`):** Recibe `props` puras en `DomainModel`, sin conocer RTK Query ni endpoints del backend.

### 2. HTML5 Semántico, SEO y Accesibilidad (Prohibido el "Div Soup")
- Toda Página (`*Page.tsx`) DEBE renderizar `<PageMeta title="..." description="..." />` y un **único `<h1>`** descriptivo, seguido de `<h2>` para secciones (`<section aria-labelledby="...">`) y `<h3>` para tarjetas (`<article>`).
- Usar `<nav aria-label="...">` para barras de navegación y pestañas, `<aside>` para paneles laterales (`CitationDrawer`), `<fieldset>` + `<legend>` en formularios y `<footer>` para barras de acción o metadatos.
- Incluir `aria-live="polite"` en contenedores de estados asíncronos de IA (`PENDIENTE`, `PROCESANDO`, `COMPLETADA`, `SIN_RESPALDO`) y `role="grid"` / `role="gridcell"` con soporte de teclado en la matriz $5 \times 5$.

### 3. Formularios Tipados (`React Hook Form` + `zodResolver`)
- Nunca enviar payloads sin validar con Zod ni sin transformar mediante `fromDomainToApi`.
- Mapear errores `HTTP 400` del backend campo por campo usando `form.setError(fieldName, { message })`.

### 4. Renderizado Markdown Seguro y Citas Normativas
- Para `ORIENTACION_OPERATIVA` y `VERSION_RIESGO`:
  - Renderizar con `<ReactMarkdown remarkPlugins={[remarkGfm]} rehypePlugins={[rehypeSanitize]}>`.
  - Conectar cada insignia de cita `[Doc: Código vX | Págs. A-B]` a `CitationDrawer.tsx` (`<aside>`) mostrando `seccion_detectada`, `pagina_inicio`, `pagina_fin`, `score_similitud` y `contenido_markdown`.
