# Reglas Críticas de Clean Code, Arquitectura Limpia y Contratos Full-Stack — RiesgoTI

> **NIVEL DE PRIORIDAD:** `CRITICAL / ALWAYS ON`
> **ALCANCE:** Obligatorio para todas las implementaciones, refactorizaciones y revisiones ejecutadas por **Claude Code** y **Google Antigravity** tanto en `back/` (Django 5 + DRF) como en `front/` (React + TypeScript).

---

## 1. Arquitectura Limpia en Frontend (React + Clean Architecture)

### 1.1 Flujo Unidireccional de 5 Capas (Cero Llamadas a API en Vistas o Componentes)
Queda **terminantemente prohibido** invocar `fetch`, `axios`, clientes HTTP directos o mutaciones/queries crudas de red dentro de páginas o componentes visuales. Todo flujo de datos en `front/src/` DEBE atravesar estrictamente las siguientes 5 capas desacopladas:

```text
[1. Servicio API (RTK Query / HTTP)]
        │  (DTO crudo de red: snake_case / estructura backend)
        ▼
[2. Mapper / Adapter (src/mappers/*.mapper.ts)]
        │  fromApiToDomain(dto) / fromDomainToApi(model)
        ▼
[3. Custom Hook Especializado / Caso de Uso (src/hooks/use*.ts)]
        │  Orquesta estado, caché, errores, formularios (Zod) y selectores
        ▼
[4. Página / Contenedor de Vista (src/pages/**/*Page.tsx)]
        │  Consume únicamente Custom Hooks + define layout HTML5 semántico + SEO
        ▼
[5. Componente UI Presentacional (src/components/**/*.tsx)]
        │  Recibe exclusivamente props puras tipadas y callbacks; 100% reutilizable
```

### 1.2 Separación Estricta de Componentes
1. **Páginas / Contenedores (`src/pages/**`):**
   - Son los únicos autorizados para invocar Custom Hooks de casos de uso (`useOperationalGuidance`, `useRiskEvaluation`, `useAIConfig`, etc.).
   - No contienen lógica de transformación de payloads de red ni llamadas directas a endpoints RTK Query.
   - Estructuran la vista mediante etiquetas HTML5 semánticas e inyectan metadatos SEO (`<PageMeta />`).
2. **Componentes Presentacionales / UI (`src/components/**`):**
   - Son funciones puras de interfaz que **únicamente reciben `props` tipadas en dominio (`camelCase`)** y emiten eventos mediante callbacks (`onSubmit`, `onSelectCell`, `onInspectCitation`).
   - Prohibido acoplar un componente presentacional a la estructura de respuesta de un endpoint del backend.

### 1.3 HTML5 Semántico, SEO y Accesibilidad (Prohibido el "Div Soup")
1. **Semántica Estructural Obligatoria:**
   - Prohibido el uso indiscriminado de `<div>` para estructurar layouts o secciones lógicas.
   - Es obligatorio emplear las etiquetas semánticas apropiadas según el rol del bloque:
     - `<header>`: Encabezado institucional o cabecera de vista/módulo.
     - `<nav aria-label="...">`: Barras de navegación lateral, migas de pan (`breadcrumbs`) y pestañas de navegación.
     - `<main id="main-content">`: Contenedor principal único por vista renderizada.
     - `<section aria-labelledby="...">`: Agrupaciones temáticas dentro de una vista (ej. sección de orientación RAG, sección de matriz $5 \times 5$).
     - `<article>`: Entidades autónomas repetibles o autocontenidas (ej. tarjeta de incidencia, tarjeta de paso de orientación, cita normativa en el drawer, versión de riesgo).
     - `<aside>`: Paneles laterales complementarios (ej. `CitationDrawer`, panel de historial de versiones o resumen de activo).
     - `<fieldset>` + `<legend>`: Agrupaciones lógicas de controles dentro de formularios (`<form>`).
     - `<footer>`: Pie de página, metadatos de auditoría o barra inferior de acciones de formulario.
2. **Configuración SEO y Jerarquía de Encabezados en cada Vista:**
   - Toda Página (`*Page.tsx`) DEBE incluir el componente `<PageMeta title="..." description="..." />` para gestionar dinámicamente `document.title` y `<meta name="description" />`.
   - Toda Página DEBE poseer **exactamente un único `<h1>`** que describa el propósito principal de la vista, seguido de una jerarquía estricta y sin saltos (`<h2>` para secciones principales, `<h3>` para sub-bloques o tarjetas).
   - Todos los controles interactivos, modales, drawers y celdas de la matriz $5 \times 5$ deben declarar atributos `aria-*` (`aria-label`, `aria-expanded`, `aria-live="polite"` para estados de polling IA, `role="grid"` / `role="gridcell"` en la matriz).

---

## 2. Contratos, Mappers y Política Dual de Compatibilidad (Front & Back)

### 2.1 Mapeo Obligatorio de Entrada/Salida (DTOs y Adapters)
1. **Backend (`back/apps/**`):**
   - **Prohibido devolver instancias crudas del ORM o diccionarios sin contrato.**
   - Toda entrada HTTP debe validarse mediante un **Input DTO / Serializer de Entrada** (`*InputSerializer` / `*WriteSerializer`) con reglas explícitas de tipos, rangos y negocio.
   - Toda respuesta HTTP debe transformarse mediante un **Output Mapper / Serializer de Salida** (`*OutputSerializer` / `*ReadSerializer`) que exponga únicamente los campos autorizados por contrato y garantice el ocultamiento criptográfico (ej. jamás exponer `api_key_llm_cifrada`, solo `api_key_llm_mascara`).
2. **Frontend (`front/src/mappers/**`):**
   - Cada módulo de servicio (`adminApi`, `incidentsApi`, `risksApi`, `authApi`) DEBE contar con su archivo Mapper dedicado (`src/mappers/admin.mapper.ts`, `src/mappers/incidents.mapper.ts`, `src/mappers/risks.mapper.ts`, `src/mappers/auth.mapper.ts`).
   - Todo payload proveniente del backend (`ApiDTO`) debe transformarse con funciones puras `fromApiToDomain(...)` antes de ser entregado por el Custom Hook a la vista.
   - Todo payload enviado desde un formulario validado por Zod hacia el backend debe transformarse con `fromDomainToApi(...)` antes de invocar la mutación RTK Query.

### 2.2 Política Dual de Compatibilidad vs. Cambio Sincronizado Full-Stack
Cada vez que una tarea o solicitud iterativa modifique un endpoint, modelo o flujo de datos, el agente DEBE clasificar y aplicar obligatoriamente uno de los siguientes dos escenarios documentándolo en el análisis previo (`iterative-change-tracker`):

* **Escenario A — Modificación Aislada (Preservar Retrocompatibilidad):**
  - *Cuándo aplica:* Cuando solo se refactoriza una de las capas (solo Backend o solo Frontend) o cuando múltiples vistas dependen del contrato existente y no deben romperse.
  - *Regla:* El agente **no puede romper el contrato existente**. Si el cambio ocurre en el Frontend o si el Backend agrega campos internos, el `Mapper/Adapter` (`fromApiToDomain` / `fromDomainToApi`) o el `OutputSerializer` debe absorber la diferencia para que los componentes UI y consumidores actuales sigan funcionando sin alteraciones.
* **Escenario B — Evolución de Funcionalidad (Cambio Sincronizado Full-Stack):**
  - *Cuándo aplica:* Cuando el usuario solicita explícitamente agregar, modificar o eliminar campos/funcionalidades de extremo a extremo (ej. un nuevo campo en un formulario o un nuevo filtro analítico).
  - *Regla:* El agente tiene **autorización y obligación de actualizar todas las capas en cadena** dentro de la misma iteración controlada:
    1. Modelo / Migración (respetando `bd.sql`)
    2. Input/Output DTO Serializer en Backend (`back/apps/**/serializers.py`)
    3. Servicio / ViewSet en Backend (`back/apps/**/views.py`)
    4. Contrato API TypeScript y Esquema Zod (`front/src/types/api.types.ts`, `front/src/schemas/*.schema.ts`)
    5. Mapper / Adapter (`front/src/mappers/*.mapper.ts`)
    6. Custom Hook de Caso de Uso (`front/src/hooks/use*.ts`)
    7. Página Contenedora y Componente UI Presentacional (`front/src/pages/**`, `front/src/components/**`)

---

## 3. Regla de Automatización de Skills (`npx autoskills` + `sync-autoskills`)

1. Cada vez que se instalen o actualicen librerías en el proyecto (`npm install`, `pnpm add`, `pip install`, etc.), el agente DEBE ejecutar `npx.cmd -y autoskills` (usando `npx.cmd` en entornos Windows PowerShell) o la skill `sync-autoskills`.
2. Todas las skills generadas deben sincronizarse de forma idéntica en `.agents/skills/` (Google Antigravity) y `.claude/skills/` (Claude Code).
3. Toda skill nueva o actualizada debe registrarse en `/agents/skills_index.md`, en las tablas de invocación obligatoria de `CLAUDE.md` y `AGENTS.md`, y en el nodo `skills_registry` del Brain (`./agents/brain/project_memory.json`).
