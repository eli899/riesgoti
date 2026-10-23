# Documentación Central del Ecosistema Agéntico Híbrido — RiesgoTI

Este directorio centraliza el **Brain Compartido** y la arquitectura agéntica sincronizada entre **Claude Code** y **Google Antigravity**.

## Estructura Sincronizada del Proyecto

| Componente | Ruta Google Antigravity | Ruta Claude Code | Ruta Central Compartida |
| :--- | :--- | :--- | :--- |
| **Reglas Maestras** | `AGENTS.md` | `CLAUDE.md` | `agents/README.md` |
| **Configuración MCP** | `.agents/mcp_config.json` | `.mcp.json` | `sequential-thinking`, `postgres`, `context7` (sin `memory`; el Brain se edita directamente) |
| **Brain Compartido** | — | — | `agents/brain/project_memory.json` |
| **Skills** | `.agents/skills/<skill>/SKILL.md` | `.claude/skills/<skill>/SKILL.md` | 5 skills simétricas (`sync-brain`, `scaffold-backend-module`, `scaffold-frontend-module`, `db-migration-check`, `iterative-change-tracker`) |
| **Changelogs** | — | — | `changelogs/INDEX.md`, `changelogs/TEMPLATE.md`, `changelogs/entries/` |
| **Sub-agentes** | `.agents/subagents/<agent>.md` | `.claude/agents/<agent>.md` | 4 perfiles especializados |
