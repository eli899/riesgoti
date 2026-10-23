---
name: sync-autoskills
description: Protocolo automático para ejecutar npx autoskills tras instalar dependencias, unificar las skills descargadas en .agents/skills/ y .claude/skills/, actualizar el catálogo /agents/skills_index.md, los archivos maestros CLAUDE.md y AGENTS.md y sincronizar el registro en el Brain.
---

# Skill: `sync-autoskills` (Sincronización e Indexación Automática de Skills del Stack)

## Cuándo Invocar esta Skill (DISPARADOR AUTOMÁTICO)
- **SIEMPRE** después de instalar o actualizar dependencias en el proyecto (`npm install`, `pnpm add`, `yarn add`, `pip install`, o durante la ejecución del paso `v0.0` de Backend/Frontend).
- Cuando el usuario o un sub-agente solicite actualizar las capacidades técnicas de los agentes en función de las librerías detectadas en `front/package.json` o `back/requirements.txt`.

---

## Procedimiento Paso a Paso

### Paso 1: Ejecución de `npx autoskills`
1. En entornos Windows PowerShell, utiliza siempre `npx.cmd -y autoskills` para evitar bloqueos de `ExecutionPolicy` sobre `npx.ps1`.
2. Ejecuta el escaneo sobre los directorios que contienen manifiestos de paquetes (`front/`):
   ```powershell
   cd front; npx.cmd -y autoskills
   ```
3. Si `autoskills` genera carpetas locales `.agents/skills/*` o `.claude/skills/*` dentro del subdirectorio `front/`, unifícalas inmediatamente hacia la raíz del repositorio para que tanto **Google Antigravity** (`.agents/skills/`) como **Claude Code** (`.claude/skills/`) tengan exactamente el mismo catálogo:
   ```powershell
   Copy-Item -Path "front\.agents\skills\*" -Destination ".agents\skills\" -Recurse -Force
   Copy-Item -Path "front\.agents\skills\*" -Destination ".claude\skills\" -Recurse -Force
   Remove-Item -Path "front\.agents", "front\.claude" -Recurse -Force -ErrorAction SilentlyContinue
   ```

### Paso 2: Inspección de Metadatos (`SKILL.md`)
1. Verifica que cada carpeta en `.agents/skills/<skill-name>/SKILL.md` y `.claude/skills/<skill-name>/SKILL.md` posea su frontmatter YAML válido (`name` y `description`).
2. Clasifica cada skill en una de las dos categorías del proyecto:
   - **Skills de Gobernanza y Arquitectura Propia (`RiesgoTI Core`):** `sync-brain`, `iterative-change-tracker`, `sync-autoskills`, `scaffold-backend-module`, `scaffold-frontend-module`, `db-migration-check`.
   - **Skills Técnicas Auto-Generadas (`autoskills Stack`):** `react-best-practices`, `composition-patterns`, `typescript-advanced-types`, `tailwind-css-patterns`, `vite`, `frontend-design`, `accessibility`, `seo`, `nodejs-backend-patterns`, `nodejs-best-practices`.

### Paso 3: Actualización del Índice Maestro (`/agents/skills_index.md`) y Reglas (`CLAUDE.md` / `AGENTS.md`)
1. Actualiza `/agents/skills_index.md` documentando para cada skill:
   - Nombre identificador (`name`).
   - Origen (`RiesgoTI Core` o `npx autoskills`).
   - Rutas espejadas en `.agents/skills/` y `.claude/skills/`.
   - **Regla de Invocación:** En qué tipo de tarea o módulo del roadmap es obligatorio invocarla.
2. Verifica que la tabla resumen de invocación de skills esté sincronizada en `CLAUDE.md` y `AGENTS.md`.

### Paso 4: Sincronización en el Brain (`./agents/brain/project_memory.json`)
1. Actualiza la clave `skills_registry` en `./agents/brain/project_memory.json` con la lista completa de skills activas, su fecha de sincronización y tecnologías detectadas.
