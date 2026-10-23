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

Tecnologías detectadas automáticamente en el workspace (`front/package.json`; el manifiesto de `figma/` se retiró en CHG-0009): **React, TypeScript, Tailwind CSS, Vite, Node.js, React Hook Form, Zod**.

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
