---
name: db-migration-check
description: Protocolo de validación y auditoría de esquemas SQL y migraciones de Django contra el archivo maestro docs/bd.sql en PostgreSQL.
---

# Skill: Verificación de Esquema y Migraciones (`db-migration-check`)

## Propósito
Asegurar que los modelos de Django (`models.py`) y sus migraciones generadas respeten al $100\%$ la estructura relacional, tipos de datos, restricciones de integridad y nombres de tablas/columnas definidos en `bd.sql`.

---

## 1. Mapa de las 31 Tablas Oficiales de `bd.sql` (`docs/bd.sql`) + 1 Tabla Técnica Aditiva
Cualquier cambio en `models.py` o archivo de migración debe verificarse contra estas tablas:

1. **Seguridad y Catálogos Base (`apps/accounts/` y `apps/assets_matrix/`):**
   * `ROL` (`PK: id`)
   * `PERMISO` (`PK: id`)
   * `ROL_PERMISO` (`PK compuesta: (rol_id, permiso_id)`)
   * `USUARIO` (`PK: id`, `UNIQUE: correo`, columna generada `nombre_completo`)
   * `TIPO_ACTIVO` (`PK: id`)
   * `ACTIVO` (`PK: id`, `FK: tipo_id -> TIPO_ACTIVO.id`)
   * `DOCUMENTO` (`PK: id`)
   * `RIESGO` (`PK: id`)
   * `NIVEL_RIESGO` (`PK: id`)
2. **Matriz de Riesgo (`apps/assets_matrix/`):**
   * `CONFIGURACION_MATRIZ` (`PK: id`, `FK: creada_por -> USUARIO.id`)
   * `CRITERIO_PROBABILIDAD` (`PK compuesta: (matriz_id, valor)`)
   * `CRITERIO_IMPACTO` (`PK compuesta: (matriz_id, valor)`)
   * `CELDA_MATRIZ` (`PK: id`, `FKs: matriz_id, nivel_id, (matriz_id, probabilidad), (matriz_id, impacto)`)
3. **Gestión Documental y RAG (`apps/documents_rag/` y `apps/ai_config/`):**
   * `VERSION_DOCUMENTO` (`PK: id`, `FKs: documento_id, cargada_por, autorizada_por`)
   * `FRAGMENTO_DOCUMENTO` (`PK: id`, `FK: version_documento_id -> VERSION_DOCUMENTO.id`)
   * `CONFIGURACION_IA_RAG` *(Única extensión técnica autorizada)* (`PK: id`, `FK: actualizada_por -> USUARIO.id`)
4. **Versiones de Riesgo y Decisiones (`apps/risks/`):**
   * `VERSION_RIESGO` (`PK: id`, `FKs: riesgo_id, creada_por, activo_confirmado_id, agrupacion_confirmada_por`)
   * `EVALUACION_RIESGO` (`PK/FK 1:1: version_id -> VERSION_RIESGO.id`, `FKs: celda_id, evaluador_id`)
   * `DECISION_RIESGO` (`PK/FK 1:1: version_id -> VERSION_RIESGO.id`, `FK: analista_id`)
   * `CITA_VERSION_RIESGO` (`PK compuesta: (version_id, numero)`, `FK: fragmento_id`)
5. **Incidencias, Atención y Evidencias (`apps/incidents/`):**
   * `INCIDENCIA` (`PK: id`, `FKs: reportante_id, activo_reportado_id`)
   * `HISTORIAL_ESTADO_INCIDENCIA` (`PK: id`, `FKs: incidencia_id, actor_id`)
   * `ATENCION_INCIDENCIA` (`PK/FK 1:1: incidencia_id -> INCIDENCIA.id`, `FK: analista_id`)
   * `EVIDENCIA_INCIDENCIA` (`PK: id`, `FK: incidencia_id`)
6. **Tareas IA, Análisis y Orientación (`apps/risks/`):**
   * `TAREA_IA` (`PK: id`, `FKs: incidencia_id, solicitada_por`)
   * `ANALISIS_INCIDENCIA` (`PK/FK 1:1: tarea_id -> TAREA_IA.id`)
   * `VERSION_INCIDENCIA` (`PK compuesta: (version_id, analisis_id)`, `FK: revisada_por`)
   * `ORIENTACION_OPERATIVA` (`PK/FK 1:1: tarea_id -> TAREA_IA.id`)
   * `PASO_ORIENTACION` (`PK compuesta: (tarea_id, orden)`)
   * `RESPUESTA_ORIENTACION` (`PK/FK 1:1: tarea_id -> ORIENTACION_OPERATIVA.tarea_id`)
   * `CITA_ORIENTACION` (`PK compuesta: (tarea_id, numero)`, `FK: fragmento_id`)
7. **Auditoría Transversal (`apps/audit/`):**
   * `EVENTO_AUDITORIA` (`PK: id`, `FKs: actor_id, incidencia_id, version_riesgo_id, version_documento_id, matriz_id, activo_id, usuario_objeto_id, rol_id`)

---

## 2. Reglas de Adaptación PostgreSQL vs. Sintaxis Híbrida de `bd.sql`
Al ejecutar o generar migraciones sobre **PostgreSQL**:
1. **`ON UPDATE CURRENT_TIMESTAMP`:** Es sintaxis de MySQL presente en `bd.sql`. En PostgreSQL debe traducirse mediante `auto_now=True` en el modelo Django y una función/trigger PL/pgSQL (`update_ultima_modificacion_column()`) inyectada con `migrations.RunSQL`.
2. **Columna Generada `USUARIO.nombre_completo`:** Mantener en PostgreSQL como `GENERATED ALWAYS AS (CONCAT(nombres, ' ', apellidos)) STORED` mediante `RunSQL` o propiedad calculada, evitando que Django intente hacer `INSERT` directo sobre esa columna si es generada por el motor.
3. **Secuencias / Auto-incremento en `id INT NOT NULL`:** En `bd.sql` los `id` están declarados como `INT NOT NULL`. Usar `AutoField` / `IDENTITY` en PostgreSQL para permitir inserciones desde el ORM manteniendo el tipo entero exacto.
4. **Verificación con MCP `postgres`:** Tras ejecutar `python manage.py migrate`, usar el servidor MCP `postgres` (o `python manage.py sqlmigrate`) para comprobar que los nombres de tablas y columnas coinciden exactamente con `bd.sql`.
