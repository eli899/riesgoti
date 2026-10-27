# Índice Maestro de Skills del Ecosistema Agéntico Híbrido — RiesgoTI

> **Sincronización Simétrica:** Todas las skills listadas en este índice están instaladas de forma idéntica tanto para **Google Antigravity** (`.agents/skills/<nombre>/SKILL.md`) como para **Claude Code** (`.claude/skills/<nombre>/SKILL.md`).

---

## 1. Skills de Gobernanza, Arquitectura y Dominio (`RiesgoTI Core`)

| # | Skill (`name`) | Propósito Técnico | Cuándo Invocarla (Disparador Obligatorio) |
| :-: | :--- | :--- | :--- |
| 1 | **`sync-brain`** | Lectura y escritura directa del archivo canónico `./agents/brain/project_memory.json` (sin servidor MCP `memory`). | **Antes de iniciar cualquier tarea** y **al finalizar cualquier paso del roadmap o cambio iterativo**. |
| 2 | **`iterative-change-tracker`** | Protocolo de 3 etapas (`Análisis + Brain -> Puerta de Aprobación -> Ejecución + Changelog + Brain`), normalización ortográfica y evaluación de contratos/Mappers. | **En toda solicitud del usuario** que modifique, ajuste, corrija o evolucione archivos existentes. |
| 3 | **`sync-autoskills`** | Ejecución de `npx.cmd -y autoskills`, unificación en `.agents/skills/` y `.claude/skills/` e indexación en Brain. | **Inmediatamente después** de ejecutar `npm install`, `pnpm add`, `pip install` o agregar librerías al stack. |
| 4 | **`scaffold-backend-module`** | Plantilla de construcción en Django 5 + DRF + Celery + FAISS con Input/Output DTO Serializers, RBAC/SoD e invariantes ISO 27001/27005. | Al crear o modificar cualquier app, modelo, serializer, ViewSet o tarea Celery en `back/apps/**` (`v0.0` a `v0.6`). |
| 5 | **`scaffold-frontend-module`** | Plantilla de Clean Architecture en React (`Servicio API -> Mapper/Adapter -> Custom Hook -> Página Contenedora -> Componente Presentacional`), Zod y HTML5 semántico. | Al crear o modificar cualquier servicio, mapper, custom hook, página o componente en `front/src/**` (`v0.0` a `v0.6`). |
| 6 | **`db-migration-check`** | Auditoría 1:1 de modelos Django y migraciones contra `docs/bd.sql` (31 tablas DDL + `CONFIGURACION_IA_RAG`, `CompositePrimaryKey`, triggers y constraints). | Antes y después de ejecutar `makemigrations` / `migrate` o modificar cualquier `models.py` en `back/`. |

---

## 2. Skills del Stack Tecnológico Auto-Instaladas (`npx autoskills v0.4.1`)

Tecnologías detectadas automáticamente en el workspace (`front/package.json`; el manifiesto de `figma/` se retiró en CHG-0009): **React, TypeScript, Tailwind CSS, Vite, Node.js, React Hook Form, Zod, Playwright**.

| # | Skill (`name`) | Fuente (`autoskills`) | Cuándo Invocarla (Disparador Obligatorio) |
| :-: | :--- | :--- | :--- |
| 7 | **`react-best-practices`** | `vercel-labs/agent-skills` | Al implementar Custom Hooks (`src/hooks/use*.ts`), manejo de efectos, memoización y optimización de renderizado en React (`v0.1` a `v0.6`). |
| 8 | **`composition-patterns`** | `vercel-labs/agent-skills` | Al separar **Páginas Contenedoras** (`src/pages/**`) de **Componentes UI Presentacionales** (`src/components/**`) y diseñar componentes compuestos reutilizables. |
| 9 | **`typescript-advanced-types`** | `wshobson/agents` | Al definir contratos de red (`src/types/api.types.ts`), modelos de dominio (`src/types/domain.types.ts`), esquemas inferidos de Zod y firmas tipadas de Mappers (`fromApiToDomain` / `fromDomainToApi`). |
| 10 | **`tailwind-css-patterns`** | `giuseppe-trisciuoglio/developer-kit` | Al estilizar layouts corporativos, grillas de la matriz de riesgo $5 \times 5$, estados semánticos (`CRITICO`, `ALTO`, `MEDIO`, `BAJO`) y paneles responsivos con Tailwind CSS. |
| 11 | **`accessibility`** | `addyosmani/web-quality-skills` | Al construir vistas con HTML5 semántico (`<main>`, `<section>`, `<article>`, `<nav>`, `<aside>`, `<fieldset>`), navegación por teclado, atributos `aria-*` y anuncios `aria-live` para tareas IA. |
| 12 | **`seo`** | `addyosmani/web-quality-skills` | Al configurar `<PageMeta />` en cada página (`*Page.tsx`), garantizando título dinámico, meta descripción y jerarquía estricta de un único `<h1>` con `<h2>`-`<h6>` lógicos. |
| 13 | **`frontend-design`** | `anthropics/skills` | Al refinar componentes visuales en `front/src/components/`, manteniendo jerarquía visual, micro-interacciones y legibilidad ejecutiva ISO 27001/27005. |
| 14 | **`vite`** | `antfu/skills` | Al configurar `front/vite.config.ts`, alias `@/`, variables de entorno `VITE_*` y optimización de bundles en `v0.0`. |
| 15 | **`nodejs-backend-patterns`** | `wshobson/agents` | Como referencia complementaria para patrones de clientes HTTP, manejo de colas/reintentos, mappers de contratos REST y scripts de tooling Node. |
| 16 | **`nodejs-best-practices`** | `sickn33/antigravity-awesome-skills` | Al escribir scripts de automatización, configuración de linters, pruebas E2E y gestión determinista de dependencias en `front/`. |
| 17 | **`react-hook-form`** | `pproenca/dot-skills` | Al construir formularios con `useForm` + `zodResolver` (configuración IA/RAG, carga de documentos, registro de incidencias, evaluación de riesgos), optimizando suscripciones (`useWatch`, `useController`) y re-renderizados (`v0.1` a `v0.6`). |
| 18 | **`zod`** | `pproenca/dot-skills` | Al definir esquemas en `front/src/schemas/*.schema.ts` (`aiConfig`, `document`, `incident`, `riskEvaluation`), reglas `.superRefine()`, `safeParse` y tipos inferidos con `z.infer` (`v0.1` a `v0.6`). |

> **Sincronización `v0.0` Frontend (2026-10-22):** `npx autoskills` ejecutado en `front/` tras instalar las dependencias de arquitectura detectó además **React Hook Form** y **Zod**, incorporando las skills `react-hook-form` y `zod`. Catálogo unificado y simétrico en `.agents/skills/` y `.claude/skills/` (18 skills).

| # | Skill (`name`) | Fuente (`autoskills`) | Cuándo Invocarla (Disparador Obligatorio) |
| :-: | :--- | :--- | :--- |
| 36 | **`playwright-best-practices`** | `currents-dev/playwright-best-practices-skill` | Al escribir, depurar o estabilizar pruebas en `front/e2e/` y configurar `front/playwright.config.ts`: localizadores, aserciones con espera automática, fixtures/POM, autenticación por rol, *mocking* de la API, accesibilidad (axe-core) y CI. Referencia detallada (58 guías temáticas) que complementa las reglas de `testing-playwright` (§3). |

> **Sincronización Playwright (2026-10-24, CHG-0015):** tras `pnpm add -D @playwright/test@1.64.1`, `npx autoskills` en `front/` detectó **Playwright** e incorporó `playwright-best-practices`; las 12 skills restantes se reinstalaron sin cambios. Catálogo unificado y simétrico en `.agents/skills/` y `.claude/skills/` (36 skills).
>
> **Sincronización React Router (2026-10-25, CHG-0018):** tras `pnpm add react-router-dom` (`7.18.4`), `npx autoskills` en `front/` no detectó una skill de registro para React Router y reinstaló las 13 skills del stack sin cambios versionados. La guía de enrutado es `react-router-dom-patterns` (#37, hub `my-skills`, CHG-0017). Catálogo unificado y simétrico (37 skills).

---

## 3. Skills del Hub Centralizado `my-skills` (`Jaguilars5/my-skills`)

Incorporadas en **CHG-0014** (2026-10-24) desde [`github.com/Jaguilars5/my-skills`](https://github.com/Jaguilars5/my-skills) (commit `8275dd6`) mediante copia selectiva de los `SKILL.md`, sin ejecutar el CLI del hub (que sobrescribiría sub-agentes, `.mcp.json`, Brain y `CLAUDE.md`). Única adaptación: `description` del frontmatter entrecomillada para que el YAML sea válido. Catálogo total: **35 skills**, simétrico en `.agents/skills/` y `.claude/skills/`.

| # | Skill (`name`) | Grupo | Cuándo Invocarla (Disparador Obligatorio) |
| :-: | :--- | :--- | :--- |
| 19 | **`django-drf-architecture`** | `backend` | Al crear o refactorizar apps en `back/apps/**`: lógica de escritura en `services.py`, lecturas optimizadas en `selectors.py` (`select_related` / `prefetch_related` anti N+1) y `*InputSerializer` / `*OutputSerializer` delgados. |
| 20 | **`postgres-performance-patterns`** | `backend` | Al diseñar índices (B-Tree, GIN para JSONB, parciales), analizar consultas lentas con `EXPLAIN ANALYZE` o escribir migraciones sin bloqueos sobre PostgreSQL 16. |
| 21 | **`redis-caching-patterns`** | `backend` | Al introducir caché sobre Redis 7 (Cache-Aside, claves jerárquicas `riesgoti:*`, TTL con jitter) o bloqueos distribuidos que eviten ejecuciones duplicadas de tareas Celery. |
| 22 | **`openapi-documentation`** | `backend` | Al documentar endpoints con `drf-spectacular` (`/api/schema/`), códigos RFC 9110 (incluido el `403` de SoD) y esquema `bearerAuth` JWT; base de la verificación OpenAPI ↔ Zod de `qa-reviewer`. |
| 23 | **`python-clean-architecture`** | `backend` | Al escribir código Python 3.12 con tipado estricto (`typing` / `mypy`), linting y formateo (`ruff`) y pruebas `pytest` / `pytest-django`. |
| 24 | **`cascade-activation-lifecycle`** | `backend` | Al activar o desactivar catálogos con dependientes (activos, usuarios, roles, versiones documentales): detección de dependientes activos, `409 Conflict` y confirmación `cascadeRelated=true` en transacción atómica. |
| 25 | **`typescript-advanced-contracts`** | `shared` | Al separar contratos Wire (`api.types.ts`) de Dominio (`domain.types.ts`) en los Mappers, modelar estados con uniones discriminadas + `assertNever` y usar tipos *branded* para IDs. |
| 26 | **`state-management-discipline`** | `shared` | Al decidir qué estado vive en Redux Toolkit / RTK Query frente a estado local de UI, y al escribir selectores memoizados. |
| 27 | **`constants-no-magic-strings`** | `shared` | Al usar estados de dominio (`PROCESANDO`, `CERRADA`, `VALIDADA`, `PENDIENTE_ATENCION_HUMANA`…), roles, permisos, rutas, endpoints, límites numéricos o intervalos de polling: centralizar en `*.constants.ts` con `as const`. El texto visible permanece en el componente (AUD-0001 R-03, CHG-0034). |
| 28 | **`git-commit-discipline`** | `shared` | Al redactar commits semánticos atómicos (`feat`, `fix`, `refactor`, `docs`…) en los submódulos `back/` / `front/` y luego en la raíz. |
| 29 | **`naming-conventions-standard`** | `shared` | Al nombrar variables, funciones, componentes, archivos y constantes (camelCase, PascalCase, UPPER_SNAKE_CASE, kebab-case) y al usar emojis en UI o consola. |
| 30 | **`prompt-standardization`** | `shared` | Al recibir requerimientos en bruto: corrección ortográfica, especificación trazable BDD (*Given/When/Then*) y cierre de ambigüedades antes de la Etapa 2 de `iterative-change-tracker`. |
| 31 | **`strict-code-quality`** | `shared` | Al escribir o revisar TypeScript y Python: sin `as any` ni *type sniffing*, `import type`, semántica RFC 9110, concurrencia optimista y límites de complejidad. |
| 32 | **`testing-playwright`** | `shared` | Al crear pruebas E2E de los flujos por rol (Reportante, Analista, Administrador) con Page Object Model, selectores semánticos y red interceptada de forma determinista. |
| 33 | **`tailwind-design-system`** | `react` | Al definir la escala de tokens Tailwind, la composición de clases con `clsx` + `tailwind-merge` y la accesibilidad WCAG AA (foco visible, roles, contraste). |
| 34 | **`vite-bundle-optimization`** | `react` | Al aplicar *code-splitting* por portal de rol con `React.lazy`, `manualChunks` para dependencias pesadas (`react-markdown`, etc.) y mejorar Core Web Vitals. |
| 35 | **`reusable-design-tokens`** | `react` | Al estilizar formularios, tablas y encabezados repetidos: objetos canónicos de clases (`inputClassName`, `selectClassName`, `tableClassName`, `headerClassName`) en lugar de cadenas inline duplicadas. |
| 37 | **`react-router-dom-patterns`** | `react` | Al declarar o modificar rutas del frontend, enlaces de navegación (`<Link>` / `<NavLink>` / `useNavigate`) o guardias de acceso por rol: objeto `ROUTES` inmutable con `as const` y constructores de URL tipados, guardia protegida con `<Navigate>` + `<Outlet>` y layouts anidados. Se aplica junto con `constants-no-magic-strings` (#27). Incorporada en CHG-0017. |

> **Precedencia (obligatoria):** Estas skills son guías genéricas del hub (sus ejemplos usan dominios como `orders` o `users` y rutas como `src/features/`). Ante cualquier conflicto prevalecen, en este orden, las invariantes de dominio (`CLAUDE.md` §2), `agents/rules/clean_code_and_architecture.md` y las skills `RiesgoTI Core`. En particular:
> * **`prompt-standardization`** complementa (no sustituye) la normalización y la Puerta de Aprobación de `iterative-change-tracker`.
> * **`cascade-activation-lifecycle`** nunca autoriza borrar ni anular filas históricas: desactivar una `VERSION_DOCUMENTO` o un catálogo preserva `CITA_ORIENTACION`, `CITA_VERSION_RIESGO` y `EVALUACION_RIESGO` (INV-04).
> * **`strict-code-quality`**: sus límites de longitud e indentación aplican a código nuevo o refactorizado; no obligan a reescribir módulos existentes fuera del alcance de un cambio aprobado.
> * **`testing-playwright`**: Playwright está instalado en `front/` (`@playwright/test@1.64.1`, CHG-0015); sus reglas se aplican junto con la guía detallada `playwright-best-practices` (#36).
> * **`reusable-design-tokens`** / **`tailwind-design-system`**: los tokens se ubican dentro de la estructura vigente de `front/src/` y conviven con `tailwind-css-patterns`.
> * **`react-router-dom-patterns`** (CHG-0017, mismo hub y método de copia selectiva): sus rutas de ejemplo (`src/router/routes.config.ts`, `src/features/auth/hooks/`) se adaptan a la estructura vigente: constantes en `front/src/constants/routes.constants.ts`, guardias en `front/src/features/auth/` y layouts en `front/src/components/layout/`. El control de acceso de la guardia reutiliza `useAuthSession().hasPermission` (RBAC del backend) y nunca concede al `ADMIN` rutas de evaluación o decisión de riesgos (INV-05). Las páginas de detalle siguen recibiendo callbacks (`onBack`, `onOpen*`) de su página contenedora en lugar de acoplarse al router.
