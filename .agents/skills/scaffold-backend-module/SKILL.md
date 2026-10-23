---
name: scaffold-backend-module
description: Plantilla estándar y guía arquitectónica para construir módulos del backend en Django 5.x + Django REST Framework + Celery + FAISS con DTOs/Serializers de Entrada y Salida explícitos, respetando estrictamente bd.sql, RBAC ISO 27001 y auditoría transversal.
---

# Skill: `scaffold-backend-module` (Construcción Estandarizada de Módulos Backend)

## Cuándo Usar esta Skill
Invoca esta skill al crear o ampliar cualquier aplicación dentro de `back/apps/` (`iam_audit`, `catalogs`, `ai_config`, `knowledge_base`, `incidents`, `risks_iso`).

---

## Estructura Estándar de una App Backend (`back/apps/<app_name>/`)
```text
back/apps/<app_name>/
├── __init__.py
├── apps.py
├── models.py          # Mapeo 1:1 contra bd.sql (db_table y db_column explícitos)
├── serializers.py     # Separación estricta: *InputSerializer (DTO entrada) y *OutputSerializer (Mapper salida)
├── permissions.py     # Clases BasePermission (RBAC y Segregación de Funciones SoD)
├── services.py        # Lógica de negocio transaccional y transformación a DTOs de dominio
├── tasks.py           # Tareas Celery (@shared_task) desacopladas vía transaction.on_commit()
├── views.py           # ViewSets / APIViews con AuditLogMixin y @extend_schema
├── urls.py            # DefaultRouter / urlpatterns registrados bajo /api/v1/
└── tests/
    └── test_<app_name>.py
```

---

## Reglas de Implementación Paso a Paso

### 1. Modelos (`models.py`)
- Declarar siempre `class Meta: db_table = 'NOMBRE_TABLA_SQL'`.
- Usar `db_column` cuando el atributo Python difiera o para explicitar llaves foráneas (`usuario_id`, `version_id`, `celda_id`).
- Respetar todos los `CHECK` constraints de `bd.sql` usando `models.CheckConstraint` y `models.TextChoices`.
- Para tablas con llave primaria compuesta en `bd.sql`, usar `models.CompositePrimaryKey` (Django 5.2) o `UniqueConstraint` sin introducir columnas espurias que rompan el DDL.

### 2. DTOs de Entrada y Mappers/Serializers de Salida (`serializers.py` — Regla Clean Architecture)
- **Prohibido devolver entidades de BD crudas o usar `fields = '__all__'` sin control de contrato.**
- Separar explícitamente en cada flujo:
  - **`*InputSerializer` / `*WriteSerializer`:** Valida tipos, rangos, invariantes (`chunk_overlap < chunk_size / 2`, obligatoriedad de `justificacion_sin_fuentes` cuando `tiene_citas_estrategicas == False`) y transforma la entrada validada en un DTO limpio para la capa `services.py`.
  - **`*OutputSerializer` / `*ReadSerializer`:** Actúa como Mapper de Salida determinista hacia el contrato JSON (`api.types.ts` del Frontend).
- **Seguridad Criptográfica:** Jamás incluir `api_key_llm_cifrada` ni `api_key_embedding_cifrada` en `fields` de salida; usar `write_only=True` en el `*InputSerializer` para `api_key_llm` y exponer únicamente `api_key_llm_mascara` en el `*OutputSerializer`.

### 3. Permisos y Segregación de Funciones (`permissions.py`)
- Verificar rol y código de permiso contra `USUARIO_ROL -> ROL_PERMISO -> PERMISO`.
- **Invariante SoD (ISO 27001):** En endpoints de evaluación en matriz $5 \times 5$ y decisiones de versiones de riesgo (`VALIDADA`, `CORREGIDA`, `RECHAZADA`), rechazar explícitamente al rol `ADMIN` con `HTTP 403 Forbidden`.

### 4. Transacciones, Servicios y Tareas Asíncronas (`services.py` / `views.py` / `tasks.py`)
- Envolver toda mutación de negocio en `with transaction.atomic():` dentro de `services.py`.
- Invocar tareas de IA (`run_operational_guidance_task`, `run_strategic_risk_task`, `process_document_version_task`) **exclusivamente** dentro de `transaction.on_commit(lambda: ...)`.
- Capturar excepciones de red/proveedor LLM dentro de la tarea Celery y actualizar `TAREA_IA.estado = 'FALLIDA'` y `INCIDENCIA.estado = 'PENDIENTE_ATENCION_HUMANA'` sin perder el registro transaccional.

### 5. Auditoría (`AuditLogMixin`)
- Registrar automáticamente en `EVENTO_AUDITORIA` (`usuario_id`, `accion`, `entidad_tipo`, `entidad_id`, `detalle_json`, `ip_origen`) toda acción administrativa, cambio de configuración IA/Matriz, carga/desactivación documental y decisión de riesgo.
