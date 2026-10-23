---
name: sync-brain
description: Protocolo estándar para consultar, actualizar y sincronizar nodos de arquitectura, esquemas de base de datos y estado de progreso de módulos en el Brain compartido (./agents/brain/project_memory.json) entre Claude Code y Google Antigravity.
---

# Skill: Sincronización del Brain Compartido (`sync-brain`)

## Propósito
Garantizar que **Claude Code** y **Google Antigravity** operen con contexto unificado y sin desincronización, leyendo y actualizando `./agents/brain/project_memory.json` antes y después de cada intervención.

> **Fuente única de verdad (CHG-0004):** El Brain es exclusivamente el archivo JSON `./agents/brain/project_memory.json`, editado directamente. **No se utiliza el servidor MCP `memory`** (`@modelcontextprotocol/server-memory`): persiste en formato JSONL (un objeto por línea) y, apuntado a este archivo, lo corrompería. No reintroducirlo en `.mcp.json` ni en `.agents/mcp_config.json`.

---

## 1. Protocolo Pre-Tarea (Lectura Obligatoria)
Antes de planificar, escribir o modificar cualquier archivo del proyecto:
1. **Consultar el Brain:**
   * Leer `./agents/brain/project_memory.json` (o buscar en él por código de módulo, p. ej. `MOD-01-DB-MODELS-SEEDS`) para obtener el estado actual del módulo objetivo, sus dependencias en `master_module_index`, su plan en `roadmap_plans` y las tablas de `bd.sql` involucradas (`database_schema`).
2. **Verificar Dependencias:**
   * Comprobar que todos los módulos predecesores en `master_module_index` tengan estado `COMPLETED` o que sus contratos/modelos requeridos ya existan.
3. **Verificar Invariantes:**
   * Revisar las invariantes de negocio registradas en `domain_invariants` (Independencia de Procesos, Separación Operación vs. Riesgo, No Auto-Entrenamiento Ciego, Inmutabilidad Histórica y SoD RBAC).

---

## 2. Protocolo Post-Tarea (Actualización Obligatoria)
Al finalizar cualquier paso del roadmap o subtarea:
1. **Registrar Entidades y Observaciones en el Brain:**
   * Actualizar el estado del módulo en `./agents/brain/project_memory.json` (`PENDING` -> `IN_PROGRESS` -> `COMPLETED`).
   * Añadir u actualizar las observaciones del módulo con:
     * **Archivos creados/modificados:** Rutas relativas exactas (ej. `back/apps/accounts/models.py`, `front/src/app/api/baseApi.ts`).
     * **Modelos o Esquemas Zod expuestos:** Nombres de clases o esquemas y tablas de `bd.sql` vinculadas.
     * **Endpoints expuestos / consumidos:** Método HTTP, ruta (ej. `POST /api/v1/incidents/`) y permiso RBAC exigido.
     * **Verificación DoD:** Estado de los criterios de aceptación (tests, migraciones o validaciones ejecutadas).
2. **Preservar la Integridad del Archivo JSON:**
   * Editar únicamente las claves afectadas (`master_module_index`, `roadmap_plans`, `iterative_change_governance.changelogs`, `skills_registry`, etc.), preservando intacta la estructura de primer nivel.
   * Actualizar `last_updated` y validar que el archivo siga siendo JSON válido tras cada edición (p. ej. `node -e "JSON.parse(require('fs').readFileSync('agents/brain/project_memory.json','utf8'))"`).
