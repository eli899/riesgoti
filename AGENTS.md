# Reglas Maestras del Ecosistema Agéntico Híbrido — RiesgoTI

> **REGLA DE ORO OBLIGATORIA (SINCRONIZACIÓN DEL BRAIN):**
> *"Antes de iniciar cualquier tarea, consulta el Brain (`./agents/brain/project_memory.json`). Al finalizar cualquier paso del roadmap, actualiza el Brain con los archivos creados, endpoints expuestos y estado del módulo"*.

> **CRITICAL RULE / ALWAYS ON (CONTROL ESTRICTO DE ITERACIONES Y CHANGELOGS):**
> *"Cualquier solicitud del usuario que implique modificar, ajustar, agregar o eliminar funcionalidad/UI sobre archivos existentes DEBE invocar automáticamente el protocolo de la skill `iterative-change-tracker`. Nunca edites código sin antes consultar el Brain, normalizar/corregir ortográficamente la petición del usuario, evaluar el impacto en Contratos/Mappers, presentar el análisis de archivos afectados y recibir confirmación explícita. Tras ejecutar el cambio, genera siempre el documento en `/changelogs/entries/` con redacción profesional y actualiza el Brain."*

> **CRITICAL RULE / ALWAYS ON (CLEAN ARCHITECTURE, HTML5 SEMÁNTICO + SEO, MAPPERS/ADAPTERS Y AUTOSKILLS):**
> *"Todo código nuevo o modificado DEBE cumplir estrictamente las normas de `/agents/rules/clean_code_and_architecture.md`:*
> 1. *Cero llamadas a API en páginas o componentes: flujo obligatorio `Servicio API (RTK Query) -> Mapper/Adapter (fromApiToDomain / fromDomainToApi) -> Custom Hook (Caso de Uso) -> Página Contenedora -> Componente UI Presentacional`.*
> 2. *Separación estricta entre Páginas/Contenedores (`src/pages/**`) y Componentes Presentacionales (`src/components/**`).*
> 3. *Prohibido el 'div soup': uso obligatorio de etiquetas HTML5 semánticas (`<main>`, `<section>`, `<article>`, `<header>`, `<nav>`, `<aside>`, `<footer>`, `<fieldset>`), configuración SEO por vista (`<PageMeta />`, un único `<h1>` con jerarquía `<h2>`-`<h6>`) y accesibilidad `aria-*`.*
> 4. *Mapeo obligatorio de Entrada/Salida: DTOs/Serializers explícitos en Backend (prohibido devolver entidades de BD crudas) y Mappers/Adapters en Frontend (`src/mappers/*.mapper.ts`).*
> 5. *Política Dual de Compatibilidad: aplicar Retrocompatibilidad vía Adapter (Escenario A) para cambios aislados o Evolución Sincronizada Full-Stack en cadena (Escenario B) para nuevas funcionalidades.*
> 6. *Tras cualquier instalación de librerías (`npm install` / `pip install`), invocar obligatoriamente `sync-autoskills` (`npx.cmd -y autoskills`) e indexar en `/agents/skills_index.md` y el Brain."*

---

## 1. Visión del Sistema y Cumplimiento Normativo
**RiesgoTI** es una plataforma empresarial de gestión de incidencias operativas y gobierno de riesgos de Seguridad de la Información alineada con **ISO/IEC 27005:2022** (Gestión de Riesgos), **ISO/IEC 27001:2022** (SGSI) e **ISO/IEC 27002:2022** (Controles).

Cada reporte de incidencia (`INCIDENCIA`) desencadena **dos ramas asíncronas independientes** en `TAREA_IA`:
1. **Rama Operativa (`ORIENTACION_OPERATIVA`):** Recupera fragmentos autorizados y activos (`finalidad IN ('OPERATIVA', 'AMBAS')`) para entregar pasos de resolución inmediatos al reportante con trazabilidad exacta de citas (`CITA_ORIENTACION`).
2. **Rama Estratégica (`ANALISIS_INCIDENCIA`):** Extrae vectores de riesgo (activo, amenaza, vulnerabilidad, consecuencia), calcula similitud semántica contra reportes previos y propone agrupaciones o nuevas versiones de riesgo (`VERSION_RIESGO`) sustentadas en normativa interna (`finalidad IN ('ESTRATEGICA', 'AMBAS')`).

---

## 2. Invariantes Críticas del Dominio (Inquebrantables)
1. **Independencia de Procesos:** El registro de una `INCIDENCIA` es transaccional (`transaction.atomic()`) y previo a cualquier invocación de IA (`transaction.on_commit()`). Si el proveedor de IA falla o no hay documentos vigentes que superen el umbral de similitud (`SIN_RESPALDO`), la incidencia existe y se deriva automáticamente a `PENDIENTE_ATENCION_HUMANA`.
2. **Separación Operación vs. Riesgo:** Que un Reportante marque que la solución de IA `"FUNCIONO"` cambia el estado de la incidencia a `CERRADA`, pero **jamás elimina, detiene ni anula** el análisis estratégico del riesgo subyacente (`run_strategic_risk_task`) ni su recurrencia.
3. **No Auto-Entrenamiento Ciego:** Las notas de resolución de un analista humano en `ATENCION_INCIDENCIA` quedan como historial y evidencia de recurrencia, pero **jamás** se inyectan automáticamente al índice vectorial FAISS sin pasar por el flujo formal de creación y autorización de una nueva `VERSION_DOCUMENTO` por parte del Administrador.
4. **Inmutabilidad Histórica de Citas y Matrices:** Desactivar una versión documental (`activa = FALSE`) impide su uso en nuevas consultas RAG, pero preserva intactas las referencias en `CITA_ORIENTACION` y `CITA_VERSION_RIESGO`. Del mismo modo, publicar una nueva `CONFIGURACION_MATRIZ` no altera retroactivamente los `celda_id` en `EVALUACION_RIESGO`.
5. **Segregación de Funciones (SoD - RBAC ISO 27001):** Un usuario con rol `ADMIN` **no posee permisos para evaluar ni validar riesgos** (`HTTP 403 Forbidden` en endpoints de evaluación/decisión). Solo el rol `ANALISTA` puede evaluar en la matriz $5 \times 5$ y emitir decisiones (`VALIDADA`, `CORREGIDA`, `RECHAZADA`).

---

## 3. Ecosistema Tecnológico y Convenciones de Código

### 3.0 Mapa Documental del Repositorio
| Ruta | Contenido |
| :--- | :--- |
| `docs/bd.sql` | DDL maestro y fuente única de verdad del esquema (31 tablas). Toda mención a `bd.sql` en reglas, skills y sub-agentes se refiere a este archivo. |
| `docs/hoja_de_ruta_implementacion_backend.md` / `docs/hoja_de_ruta_implementacion_frontend.md` | Hojas de ruta maestras de las que derivan los roadmaps por versión. |
| `back/roadmap/v0.0…v0.6` / `front/roadmap/v0.0…v0.6` | Roadmaps de implementación por versión y módulo (`MOD-00` a `MOD-10`). Locales: excluidos de Git. |
| `docs/manuales/` (`README.md` + `manual_usuario_{reportante,analista,administrador}.md`) + `docs/manuales/capturas/` | Manuales de usuario por rol en Markdown, con capturas enlazadas (`capturas/*.png`) y diagramas de estados en Mermaid. Al cambiar una pantalla, actualizar su captura y la sección del manual. |
| `changelogs/` | Índice (`INDEX.md`), plantilla (`TEMPLATE.md`) y detalle de cambios (`entries/CHG-XXXX_*.md`). Local: excluido de Git; el protocolo `iterative-change-tracker` se sigue aplicando igual. |
| `agents/` | Brain (`brain/project_memory.json`), reglas (`rules/`) y catálogo de skills (`skills_index.md`). |
| `.gitmodules`, `.gitignore`, `.gitattributes` | Control de versiones: la raíz es un superproyecto y `back/` (`riesgoti-back`) y `front/` (`riesgoti-front`) son submódulos Git con `.gitignore`/`.gitattributes` propios (LF). Commit primero en el submódulo y luego en la raíz. Ver `README.md` §10. |

> El prototipo visual `figma/` fue retirado (CHG-0009): su diseño ya está migrado íntegramente a `front/src/`, que es ahora la única base visual. Las menciones a `figma/src/*` en roadmaps, changelogs y comentarios de código son procedencia histórica.

### 3.1 Backend (`back/` + `docker-compose.yml` — Django 5.x + DRF + Celery + PostgreSQL 16 + Redis 7 + pgAdmin 4)
* **Infraestructura Contenedorizada con Docker Compose (`docker-compose.yml`):**
  * Todo el entorno Backend y sus servicios de soporte corren en contenedores Docker: `db` (`postgres:16-alpine` en puerto `5432`), `redis` (`redis:7-alpine` en puerto `6379`), `pgadmin` (`dpage/pgadmin4` en `http://localhost:5050` con servidor preconfigurado en `back/docker/pgadmin/servers.json`), `backend` (`python:3.12-slim` en `http://localhost:8000`) y `celery_worker`.
  * Los comandos de gestión, migraciones y pruebas de Django se ejecutan mediante `docker compose exec backend python manage.py ...` y `docker compose exec backend pytest`.
  * **Stack E2E aislado (`docker-compose.e2e.yml`, proyecto `riesgoti-e2e`):** BD en `tmpfs` + `seed_riesgoti` en cada arranque, `storage/` propio, API en `:8001` y front de pruebas en `:5174`. Lo levanta y destruye la suite Playwright (`cd front; pnpm test:e2e`, modos `E2E_STACK=fresh|keep|reuse|off`); nunca usar el stack de desarrollo para E2E porque las decisiones de riesgo son irreversibles.
* **Fidelidad Estricta a `bd.sql` (`docs/bd.sql`):**
  * Todos los modelos Django DEBEN declarar explícitamente `class Meta: db_table = 'NOMBRE_TABLA'` y `db_column` exactos respetando el DDL de `bd.sql`.
  * Prohibido alterar nombres de tablas o columnas del esquema base; la única tabla técnica aditiva autorizada es `CONFIGURACION_IA_RAG` (`apps/ai_config/models.py`).
  * Para llaves primarias compuestas (`ROL_PERMISO`, `CRITERIO_PROBABILIDAD`, `CRITERIO_IMPACTO`, `CITA_VERSION_RIESGO`, `VERSION_INCIDENCIA`, `PASO_ORIENTACION`, `CITA_ORIENTACION`), utilizar `CompositePrimaryKey` (Django 5.2) o `UniqueConstraint` en `Meta.constraints` garantizando compatibilidad total con el ORM y el DDL.
  * Para `ultima_modificacion`, usar `auto_now=True` en Django y trigger PostgreSQL en migración.
* **DTOs de Entrada y Mappers/Serializers de Salida:**
  * Prohibido devolver entidades de BD crudas. Separar explícitamente `*InputSerializer` (validación de entrada) y `*OutputSerializer` (mapeo de salida).
* **Criptografía de Credenciales (`AES-256-GCM`):**
  * Ninguna API Key (`api_key_llm`, `api_key_embedding`) se almacena en texto plano ni se expone en serializers de lectura.
  * Cifrar con `RIESGOTI_MASTER_KEY` ($32$ bytes) produciendo `Base64(IV_12B || AuthTag_16B || Ciphertext)`. Exponer únicamente las máscaras (`api_key_llm_mascara`).
* **Pipeline RAG y FAISS:**
  * Extracción página por página con `pymupdf4llm.to_markdown(pdf_path, page_chunks=True)`.
  * Chunking jerárquico con `MarkdownHeaderTextSplitter` + `RecursiveCharacterTextSplitter`, calculando `pagina_inicio = min(paginas)` y `pagina_fin = max(paginas)` e inyectando prefijo `[Documento: {codigo} v{version} | Sección: {H1 > H2}]`.
  * Vectores normalizados con norma $L_2$ e indexados en `faiss.IndexIDMap2(IndexFlatIP(dim))` usando `FRAGMENTO_DOCUMENTO.id`.
  * Filtrado estricto en el Retriever: `activa = True`, `procesamiento = 'AUTORIZADO'` y `modelo_embedding == config_actual.modelo_embedding`.
* **Auditoría Transversal:**
  * Toda mutación crítica debe registrar un `EVENTO_AUDITORIA` mediante `AuditLogMixin`.

### 3.2 Frontend (`front/` — React + TS + RTK Query + Mappers + Custom Hooks + Zod + Tailwind)
* **Arquitectura Limpia de 5 Capas:**
  * Prohibido usar `fetch`/`axios` o llamadas directas de red en páginas o componentes. Flujo estricto: `Servicio API (RTK Query) -> Mapper/Adapter (src/mappers/*.mapper.ts) -> Custom Hook (src/hooks/use*.ts) -> Página Contenedora (src/pages/**) -> Componente UI Presentacional (src/components/**)`.
* **HTML5 Semántico, SEO y Accesibilidad:**
  * Prohibido el "div soup". Usar `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<fieldset>`, `<footer>`.
  * Incluir `<PageMeta title="..." description="..." />` en cada página, un único `<h1>` por vista, jerarquía `<h2>`-`<h6>` y atributos `aria-*`.
* **Estado y Caché de Red:**
  * Prohibido usar estado mockeado en memoria para flujos productivos. Todo consumo de API se realiza con **RTK Query** (`baseApi`, `adminApi`, `incidentsApi`, `risksApi`) e invalidación por tags (`['AIConfig', 'DocumentVersion', 'Incident', 'OperationalGuidance', 'RiskVersion', 'RiskDashboard', 'MatrixConfig']`).
  * Implementar polling automático encapsulado en Custom Hooks para documentos en estado `'PROCESANDO'` ($3000\text{ ms}$) y orientación operativa con `tarea_ia.estado !== 'COMPLETADA'` ($2500\text{ ms}$).
* **Validación de Contratos y Formularios (Zod + React Hook Form):**
  * Todo formulario debe validar con `zodResolver` antes de transformar con `fromDomainToApi` y emitir peticiones HTTP:
    * `aiConfig.schema.ts`: `chunk_overlap < chunk_size / 2`.
    * `riskEvaluation.schema.ts`: `.superRefine()` exigiendo `probabilidad` ($1-5$), `impacto` ($1-5$), `justificacion` ($\ge 15$ chars) y `justificacion_sin_fuentes` (si `tiene_citas_estrategicas === false`) cuando `accion === 'VALIDADA'`.
* **Renderizado Seguro de Markdown y Citas:**
  * Usar `react-markdown` + `remark-gfm` + `rehype-sanitize` para tablas técnicas y bloques de código, junto con `CitationDrawer.tsx` (`<aside aria-label="...">`) para inspeccionar fragmentos por rango de páginas.

---

## 4. Ecosistema Agéntico Unificado (Claude Code + Google Antigravity)

### 4.1 Servidores MCP Configurados (`.mcp.json` y `.agents/mcp_config.json`)
* **Brain (sin servidor MCP):** El Brain compartido es el archivo JSON canónico `./agents/brain/project_memory.json`, que se lee y edita directamente (skill `sync-brain`). El servidor `@modelcontextprotocol/server-memory` fue retirado (CHG-0004) porque persiste en formato JSONL y corrompería el archivo.
* **`sequential-thinking` (`@modelcontextprotocol/server-sequential-thinking`):** Razonamiento estructurado para decisiones arquitectónicas complejas.
* **`postgres` (`@modelcontextprotocol/server-postgres`):** Inspección en vivo del esquema y datos sobre PostgreSQL (`bd.sql`).
* **`context7` (`@upstash/context7-mcp`):** Consulta de documentación oficial actualizada de librerías del stack.

### 4.2 Catálogo de Skills Activas (`.agents/skills/` y `.claude/skills/` — Detalle en `/agents/skills_index.md`)

#### A. Skills de Gobernanza y Arquitectura (`RiesgoTI Core`)
1. `sync-brain`: Protocolo de lectura y escritura en el Brain compartido (`./agents/brain/project_memory.json`).
2. `iterative-change-tracker`: Protocolo obligatorio de 3 etapas (`Análisis + Brain + Contratos -> Puerta de Aprobación -> Ejecución + Changelog en /changelogs/entries/ + Brain`) con normalización ortográfica estricta.
3. `sync-autoskills`: Ejecución automática de `npx.cmd -y autoskills`, unificación simétrica en `.agents/skills/` y `.claude/skills/` e indexación en `/agents/skills_index.md` y Brain tras instalar dependencias.
4. `scaffold-backend-module`: Plantilla estándar para crear modelos, Input/Output DTO Serializers, ViewSets, permisos RBAC, tareas Celery y rutas en Django/DRF.
5. `scaffold-frontend-module`: Plantilla estándar de Clean Architecture para servicios RTK Query, Mappers (`fromApiToDomain`/`fromDomainToApi`), Custom Hooks, esquemas Zod y páginas/componentes semánticos React.
6. `db-migration-check`: Verificación de compatibilidad 1:1 de modelos y migraciones contra `bd.sql`.

#### B. Skills Técnicas del Stack Auto-Instaladas (`npx autoskills`)
7. `react-best-practices`: Buenas prácticas de hooks, estado, efectos y rendimiento en React.
8. `composition-patterns`: Separación de páginas contenedoras vs. componentes UI presentacionales y composición limpia.
9. `typescript-advanced-types`: Tipado estricto de DTOs de red (`api.types.ts`), modelos de dominio (`domain.types.ts`), Mappers y Zod.
10. `tailwind-css-patterns`: Sistemas de diseño, grillas $5 \times 5$ y badges semánticos con Tailwind CSS.
11. `accessibility`: Etiquetas HTML5 semánticas, navegación por teclado y atributos `aria-*` / `aria-live`.
12. `seo`: Títulos dinámicos (`<PageMeta />`), meta descripciones y jerarquía única `<h1>` -> `<h2>`-`<h6>`.
13. `frontend-design`: Refinamiento visual corporativo e interfaces ejecutivas ISO 27001/27005.
14. `vite`: Configuración de bundler, alias `@/` y variables `VITE_*`.
15. `nodejs-backend-patterns`: Patrones de adaptadores HTTP, manejo de errores y contratos REST.
16. `nodejs-best-practices`: Gestión determinista de paquetes, scripts y pruebas automatizadas.
17. `react-hook-form`: Formularios con `useForm` + `zodResolver`, suscripciones optimizadas (`useWatch`, `useController`) y control de re-renderizados.
18. `zod`: Esquemas de validación en `src/schemas/*.schema.ts`, `.superRefine()`, `safeParse` y tipos inferidos con `z.infer`.

#### C. Skills del Hub Centralizado `my-skills` (`Jaguilars5/my-skills`, CHG-0014 — disparadores y precedencia en `/agents/skills_index.md` §3)
19. `django-drf-architecture`: Servicios (escritura) y selectores (lectura) en Django/DRF, serializers delgados y ORM sin N+1.
20. `postgres-performance-patterns`: Índices B-Tree/GIN/parciales, `EXPLAIN ANALYZE` y migraciones sin bloqueos en PostgreSQL.
21. `redis-caching-patterns`: Cache-Aside, claves jerárquicas, TTL con jitter y bloqueos distribuidos sobre Redis.
22. `openapi-documentation`: Contratos REST vivos con `drf-spectacular`, códigos RFC 9110 y esquema Bearer JWT.
23. `python-clean-architecture`: Python 3.12 con tipado estricto (`mypy`), `ruff` y `pytest`.
24. `cascade-activation-lifecycle`: Activación/desactivación segura de catálogos con dependientes (`409 Conflict`, `cascadeRelated=true`), siempre subordinada a la invariante 4.
25. `typescript-advanced-contracts`: Contratos Wire vs. Dominio, Zod como fuente de tipos, uniones discriminadas exhaustivas y tipos *branded*.
26. `state-management-discipline`: Qué estado va a Redux Toolkit / RTK Query y selectores memoizados.
27. `constants-no-magic-strings`: Estados, rutas y textos centralizados en `*.constants.ts` con `as const`.
28. `git-commit-discipline`: Commits semánticos atómicos en submódulos y raíz.
29. `naming-conventions-standard`: Convenciones de nomenclatura y uso restringido de emojis.
30. `prompt-standardization`: Refinamiento de requerimientos en especificaciones BDD; complementa a `iterative-change-tracker`.
31. `strict-code-quality`: Cero `as any`, `import type`, semántica RFC 9110 y límites de complejidad en código nuevo o refactorizado.
32. `testing-playwright`: Pruebas E2E por rol con Page Object Model y red interceptada.
33. `tailwind-design-system`: Escala de tokens, `clsx` + `tailwind-merge` y accesibilidad WCAG AA.
34. `vite-bundle-optimization`: *Code-splitting* con `React.lazy`, `manualChunks` y Core Web Vitals.
35. `reusable-design-tokens`: Objetos canónicos de clases Tailwind para formularios, tablas y encabezados.

#### D. Skills Auto-Instaladas Posteriores (`npx autoskills`, CHG-0015)
36. `playwright-best-practices`: Guía detallada de Playwright (localizadores, aserciones, fixtures/POM, autenticación por rol, *mocking*, accesibilidad y CI) para `front/e2e/`; complementa a `testing-playwright`.

#### E. Skills Posteriores del Hub `my-skills` (CHG-0017)
37. `react-router-dom-patterns`: Rutas centralizadas en un objeto `ROUTES` con `as const`, guardia protegida por rol con `<Navigate>` + `<Outlet>` y layouts anidados; se aplica junto con `constants-no-magic-strings`.

### 4.3 Sub-agentes Especializados (`.agents/subagents/` y `.claude/agents/`)
1. `db-architect`: Especialista en PostgreSQL, mapeo ORM de `bd.sql`, constraints, triggers y semillas.
2. `backend-builder`: Especialista en Django 5 + DRF, Input/Output DTO Serializers, SimpleJWT, RBAC, AES-256-GCM, Celery, PyMuPDF4LLM y FAISS.
3. `frontend-builder`: Especialista en React Clean Architecture (Servicios -> Mappers -> Custom Hooks -> Contenedores -> Presentacionales), TypeScript, RTK Query, Zod, HTML5 semántico/SEO y Tailwind CSS.
4. `qa-reviewer`: Auditor de calidad, Clean Architecture, seguridad RBAC/SoD, invariantes ISO 27001/27005 y contratos OpenAPI/Zod/Mappers.
