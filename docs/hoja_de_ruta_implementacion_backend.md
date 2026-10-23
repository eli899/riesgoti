# 01. Hoja de Ruta de Implementación Backend (Django REST Framework) — RiesgoTI

## Estructura Modular del Proyecto Django
```text
riesgoti_backend/
├── config/                  # settings.py, urls.py, celery.py, wsgi.py
├── apps/
│   ├── accounts/            # Modelos: Rol, Permiso, RolPermiso, Usuario + JWT + RBAC Permissions
│   ├── assets_matrix/       # Modelos: TipoActivo, Activo, NivelRiesgo, ConfiguracionMatriz, Criterios, CeldaMatriz
│   ├── ai_config/           # Modelo: ConfiguracionIARAG + CryptoService (AES-256-GCM) + Test Ping View
│   ├── documents_rag/       # Modelos: Documento, VersionDocumento, FragmentoDocumento + PyMuPDF4LLM + FAISS
│   ├── incidents/           # Modelos: Incidencia, HistorialEstadoIncidencia, EvidenciaIncidencia, AtencionIncidencia
│   ├── risks/               # Modelos: TareaIA, Orientacion*, AnalisisIncidencia, Riesgo, VersionRiesgo, Evaluacion*, Decision*
│   └── audit/               # Modelo: EventoAuditoria + Middleware/Mixins de trazabilidad
└── storage/
    ├── pdfs/                # Archivos PDF originales sellados con SHA-256
    ├── evidences/           # Evidencias adjuntas de incidencias
    └── faiss_indices/       # Índices vectoriales persistidos por modelo_embedding
```

---

## Etapa 1: Modelos Django Mapeados a `bd.sql`, Migraciones y Semillas
**Dependencias:** Ninguna (Base fundacional).

### Tareas de Implementación
1. **Modelado Exacto contra `bd.sql` (`models.py`):**
   * Definir cada modelo Django especificando `class Meta: db_table = 'NOMBRE_TABLA'` y `db_column` exactos para respetar al $100\%$ el DDL provisto en `bd.sql`.
   * En los campos `ultima_modificacion`, usar `auto_now=True` de Django (que actualiza el timestamp en cada `.save()`) más un trigger en PostgreSQL en la migración inicial para cubrir actualizaciones SQL directas.
   * Para `USUARIO.nombre_completo`, declararlo en PostgreSQL como columna generada (`GENERATED ALWAYS AS (CONCAT(nombres, ' ', apellidos)) STORED`) o propiedad de lectura en el modelo Django.
   * Para llaves primarias compuestas presentes en `bd.sql` (`ROL_PERMISO`, `CRITERIO_PROBABILIDAD`, `CRITERIO_IMPACTO`, `CITA_VERSION_RIESGO`, `VERSION_INCIDENCIA`, `PASO_ORIENTACION`, `CITA_ORIENTACION`), usar `UniqueConstraint` en `Meta.constraints` (o `CompositePrimaryKey` nativo en Django 5.2) asegurando compatibilidad total con el ORM.
2. **Modelo Aditivo `ConfiguracionIARAG` (`apps/ai_config/models.py`):**
   ```python
   from django.db import models

   class ConfiguracionIARAG(models.Model):
       version_configuracion = models.CharField(max_length=50, unique=True)
       proveedor_llm = models.CharField(max_length=50)          # 'OPENAI', 'GEMINI', 'HUGGINGFACE_LOCAL'
       modelo_llm = models.CharField(max_length=100)            # ej. 'gemini-2.5-flash'
       endpoint_base_llm = models.URLField(max_length=255, null=True, blank=True)
       api_key_llm_cifrada = models.TextField(null=True, blank=True)
       api_key_llm_mascara = models.CharField(max_length=50, null=True, blank=True)
       proveedor_embedding = models.CharField(max_length=50)    # 'OPENAI', 'GEMINI', 'SENTENCE_TRANSFORMERS'
       modelo_embedding = models.CharField(max_length=100)      # Sincronizado con VERSION_DOCUMENTO.modelo_embedding
       api_key_embedding_cifrada = models.TextField(null=True, blank=True)
       api_key_embedding_mascara = models.CharField(max_length=50, null=True, blank=True)
       chunk_size = models.PositiveIntegerField(default=800)
       chunk_overlap = models.PositiveIntegerField(default=120)
       top_k = models.PositiveIntegerField(default=4)
       umbral_similitud = models.DecimalField(max_digits=4, decimal_places=3, default=0.720)
       activa = models.BooleanField(default=True)
       actualizada_por = models.ForeignKey('accounts.Usuario', on_delete=models.PROTECT, db_column='actualizada_por')
       creado_en = models.DateTimeField(auto_now_add=True)

       class Meta:
           db_table = 'CONFIGURACION_IA_RAG'
   ```
3. **Management Command de Semillas (`python manage.py seed_riesgoti`):**
   * Poblar `ROL` (`REPORTANTE`, `ANALISTA`, `ADMIN`), `PERMISO` y `ROL_PERMISO` (excluyendo expresamente permisos de evaluación/validación de riesgos para el rol `ADMIN`).
   * Poblar `NIVEL_RIESGO` (`BAJO`, `MEDIO`, `ALTO`, `CRITICO`) y la matriz $5 \times 5$ inicial en `CONFIGURACION_MATRIZ`, `CRITERIO_PROBABILIDAD`, `CRITERIO_IMPACTO` y las $25$ celdas en `CELDA_MATRIZ`.

### Criterios de Aceptación (DoD - Etapa 1)
* [ ] `python manage.py migrate` y `python manage.py seed_riesgoti` se ejecutan limpiamente sobre PostgreSQL sin alterar los nombres de tablas ni columnas de `bd.sql`.
* [ ] `CeldaMatriz.objects.get(matriz__activa=True, probabilidad=4, impacto=3).nivel.codigo` retorna `'ALTO'`.

---

## Etapa 2: Autenticación SimpleJWT, Permisos RBAC en DRF y Criptografía AES-256-GCM
**Dependencias:** Etapa 1 completada.

### Tareas de Implementación
1. **Custom User & SimpleJWT Serializer (`apps/accounts/`):**
   * Configurar `Usuario` con `AbstractBaseUser` mapeado a la tabla `USUARIO` (`USERNAME_FIELD = 'correo'`, mapeando el hash a la columna `password_hash`).
   * Personalizar `TokenObtainPairSerializer` en `djangorestframework-simplejwt` para inyectar en el payload y en la respuesta de login: `usuario_id`, `nombre_completo`, `rol_codigo` y el arreglo `permisos` obtenidos desde `ROL_PERMISO`.
2. **Clases `BasePermission` Granulares y Auditoría:**
   * Implementar `HasRiesgoTIPermission(permiso_codigo)` en DRF que verifique el permiso en `ROL_PERMISO` y bloquee cualquier intento de un `ADMIN` de invocar vistas de evaluación/decisión de riesgos.
   * Implementar `AuditLogMixin` para los `GenericViewSet` que inserte automáticamente registros en `EVENTO_AUDITORIA` (`actor_id`, `accion`, `detalle`, e IDs de las entidades afectadas).
3. **Servicio de Cifrado `EnvelopeCryptoService` (`apps/ai_config/crypto.py`):**
   * Usar `cryptography.hazmat.primitives.ciphers.aead.AESGCM` con la llave `settings.RIESGOTI_MASTER_KEY` ($32$ bytes).
   * Método `encrypt_api_key(raw_key: str) -> tuple[str, str]`: devuelve `(payload_base64, mascara)` donde el payload concatena $\text{IV}_{12\text{B}} \parallel \text{AuthTag}_{16\text{B}} \parallel \text{Ciphertext}$.
   * Implementar `AIConfigViewSet` (`GET`, `PUT`) y la acción `@action(detail=False, methods=['post'], url_path='test-connection')` para probar latencia y dimensiones contra OpenAI, Gemini o Sentence Transformers.

### Criterios de Aceptación (DoD - Etapa 2)
* [ ] Un request con token de `ADMIN` hacia `/api/v1/risk-versions/{id}/decision/` devuelve `HTTP 403 Forbidden`.
* [ ] Ningún serializer de DRF incluye los campos `api_key_llm_cifrada` ni la llave en texto plano en `to_representation()`.
* [ ] El endpoint `POST /api/v1/admin/ai-config/test-connection/` valida conectividad real y devuelve la latencia en $\text{ms}$ y las dimensiones del modelo de embeddings.

---

## Etapa 3: Ingesta Documental en DRF + Celery (PDF $\rightarrow$ Markdown $\rightarrow$ Chunks $\rightarrow$ FAISS)
**Dependencias:** Etapas 1 y 2 completadas.

### Tareas de Implementación
1. **ViewSet de Carga y Gobernanza (`DocumentVersionViewSet`):**
   * `POST /api/v1/admin/document-versions/upload/` (`MultiPartParser`): valida archivo PDF, calcula `sha256`, crea o vincula `DOCUMENTO` y crea `VERSION_DOCUMENTO` con `procesamiento = 'PROCESANDO'` y `activa = False`.
   * Encola con `transaction.on_commit(lambda: process_pdf_version_task.delay(version.id))`.
2. **Tarea Celery `process_pdf_version_task` (`apps/documents_rag/tasks.py`):**
   * Extrae Markdown página por página mediante `pymupdf4llm.to_markdown(file_path, page_chunks=True)`.
   * Verifica densidad de texto: si el PDF es imagen escaneada sin texto, marca `procesamiento = 'ERROR'` y registra la causa en `error_procesamiento`.
   * Ejecuta el segmentador jerárquico Markdown aplicando `chunk_size` y `chunk_overlap` de `ConfiguracionIARAG.objects.get(activa=True)`.
   * Calcula para cada chunk su `pagina_inicio` y `pagina_fin` exactas a partir de las páginas de los bloques que lo conforman, antepone el encabezado jerárquico `[Documento: {codigo} v{version} | Sección: {headers}]` e inserta en lote con `FragmentoDocumento.objects.bulk_create()`.
   * Genera los embeddings, los normaliza ($L_2$) para similitud coseno y los agrega al índice `faiss.IndexIDMap2` usando los IDs primarios de `FRAGMENTO_DOCUMENTO`.
   * Actualiza `VERSION_DOCUMENTO`: `procesamiento = 'PENDIENTE_AUTORIZACION'`, `modelo_embedding` y `version_segmentacion = f"md-v1-cs{cfg.chunk_size}-co{cfg.chunk_overlap}"`.
3. **Acciones de Previsualización, Autorización y Activación en el ViewSet:**
   * `GET /api/v1/admin/document-versions/{id}/preview/`: devuelve la versión y todos sus `FRAGMENTO_DOCUMENTO` ordenados por `orden` para el visor Markdown del Admin.
   * `POST /api/v1/admin/document-versions/{id}/authorize/`: dentro de `transaction.atomic()`, pone `activa = False` en versiones anteriores del mismo `documento_id`, actualiza la versión actual con `procesamiento = 'AUTORIZADO'`, `autorizada_por = request.user`, `autorizada_en = timezone.now()`, `activa = True` y registra el evento en `EVENTO_AUDITORIA`.

### Criterios de Aceptación (DoD - Etapa 3)
* [ ] Las tablas del PDF se almacenan en `FRAGMENTO_DOCUMENTO.contenido` usando sintaxis Markdown (`| col1 | col2 |`).
* [ ] Todos los fragmentos conservan `pagina_inicio` y `pagina_fin` fieles a la paginación del documento PDF.
* [ ] Al desactivar una versión (`POST .../deactivate/`), deja de ser recuperada por FAISS en nuevas consultas, pero las filas existentes en `CITA_ORIENTACION` y `CITA_VERSION_RIESGO` permanecen intactas.

---

## Etapa 4: Endpoints DRF de Incidencias, Evidencias y Atención Humana (N2)
**Dependencias:** Etapas 1 y 2 completadas.

### Tareas de Implementación
1. **Creación Transaccional de Incidencias (`IncidentViewSet.create`):**
   * Exclusivo para rol `REPORTANTE`. Dentro de `transaction.atomic()`:
     1. Crea `INCIDENCIA` y guarda los archivos adjuntos en `EVIDENCIA_INCIDENCIA` calculando `bytes`, `tipo_mime` y `sha256`.
     2. Registra `'REGISTRADA'` y luego `'EN_ANALISIS'` en `HISTORIAL_ESTADO_INCIDENCIA`.
     3. Crea dos filas en `TAREA_IA` (`rama='OPERATIVA'` y `rama='ESTRATEGICA'`) con `estado='PENDIENTE'` y `clave_idempotencia`.
     4. Registra en `transaction.on_commit()` el disparo paralelo de las dos tareas Celery: `run_operational_rag_task.delay(tarea_op.id)` y `run_strategic_risk_task.delay(tarea_est.id)`.
2. **Confirmación del Reportante (`POST /api/v1/incidents/{id}/orientation-feedback/`):**
   * Crea el registro en `RESPUESTA_ORIENTACION` (`resultado = 'FUNCIONO'` o `'NO_FUNCIONO'`).
   * Si `'FUNCIONO'`, inserta `'CERRADA'` en `HISTORIAL_ESTADO_INCIDENCIA`.
   * Si `'NO_FUNCIONO'`, inserta `'PENDIENTE_ATENCION_HUMANA'` en `HISTORIAL_ESTADO_INCIDENCIA`.
3. **ViewSet de Atención Humana (`HumanAttentionViewSet`):**
   * `POST /api/v1/incidents/{id}/start-attention/`: exclusivo para `ANALISTA`, crea `ATENCION_INCIDENCIA` y añade estado `'EN_ATENCION'`.
   * `PATCH /api/v1/incidents/{id}/attention/`: permite guardar avances en `acciones` y `nota_resolucion` sin cerrar el caso, o cerrarlo definitivamente (`cerrar = True`), lo que estampa `cerrada_en` y añade el estado `'CERRADA'`.

### Criterios de Aceptación (DoD - Etapa 4)
* [ ] El endpoint `POST /api/v1/incidents/` responde en $< 300\text{ ms}$ sin esperar a que terminen las tareas de Celery.
* [ ] Si la tarea operativa falla por caída del proveedor de IA, un manejador `on_failure` de Celery registra `error_codigo` y `error_detalle` en `TAREA_IA` y pasa la incidencia automáticamente a `'PENDIENTE_ATENCION_HUMANA'`.

---

## Etapa 5: Workers Celery para RAG Operativo y Análisis Estratégico de Riesgos
**Dependencias:** Etapas 2, 3 y 4 completadas.

### Tareas de Implementación
1. **Tarea Celery `run_operational_rag_task(tarea_ia_id)`:**
   * Recupera los `top_k` vectores en FAISS filtrando únicamente `FRAGMENTO_DOCUMENTO` cuyas `VERSION_DOCUMENTO` cumplan: `activa=True`, `procesamiento='AUTORIZADO'`, `finalidad__in=['OPERATIVA', 'AMBAS']` y `modelo_embedding=cfg.modelo_embedding`.
   * Aplica el filtro de umbral de similitud coseno ($\text{score} \ge \text{cfg.umbral\_similitud}$).
   * Si no hay fragmentos que superen el umbral: guarda `ORIENTACION_OPERATIVA` con `tipo='SIN_RESPALDO'`, marca la tarea como `'COMPLETADA'` y transiciona la incidencia a `'PENDIENTE_ATENCION_HUMANA'`.
   * Si hay fragmentos válidos: invoca al LLM con **Structured Outputs** (validado con Pydantic), guarda `ORIENTACION_OPERATIVA`, `PASO_ORIENTACION` y `CITA_ORIENTACION`.
2. **Tarea Celery `run_strategic_risk_task(tarea_ia_id)`:**
   * Extrae `activo_identificado`, `amenaza`, `vulnerabilidad`, `consecuencia` y `riesgo_candidato` (dejando `'NO_DETERMINADO'` cualquier elemento no sustentado en el reporte) y crea `ANALISIS_INCIDENCIA`.
   * Compara similitud semántica y metadatos contra `ANALISIS_INCIDENCIA` previos y `VERSION_RIESGO` abiertas o vigentes.
   * Consulta en FAISS fragmentos con `finalidad__in=['ESTRATEGICA', 'AMBAS']` (políticas ISO 27002/27005 internas) para sustentar la `recomendacion` y sus citas en `CITA_VERSION_RIESGO`.
   * Crea o asocia `VERSION_RIESGO` y vincula las incidencias en `VERSION_INCIDENCIA` con `similitud`, `motivo_similitud` y `extracto_reporte`.

### Criterios de Aceptación (DoD - Etapa 5)
* [ ] Ninguna cita en `CITA_ORIENTACION` ni `CITA_VERSION_RIESGO` apunta jamás a un fragmento inexistente o no autorizado.
* [ ] Cerrar una incidencia porque la orientación operativa `"FUNCIONO"` no cancela ni detiene la ejecución de `run_strategic_risk_task`.

---

## Etapa 6: ViewSets de Revisión de Agrupaciones, Evaluación en Matriz y Decisiones ISO
**Dependencias:** Etapas 1, 2 y 5 completadas.

### Tareas de Implementación
1. **Gestión de Agrupaciones y Retiro de Falsos Positivos (`RiskVersionViewSet`):**
   * `PATCH /api/v1/risk-versions/{id}/incidents/{analisis_id}/remove/`: cambia `VERSION_INCIDENCIA.estado = 'RETIRADA'`, asigna `revisada_por = request.user`, `revisada_en = timezone.now()` y valida que `motivo_retiro` no esté vacío.
   * `POST /api/v1/risk-versions/{id}/confirm-grouping/`: confirma `activo_confirmado_id`, `agrupacion_confirmada_por` y `agrupacion_confirmada_en`.
2. **Evaluación en Matriz (`POST /api/v1/risk-versions/{id}/evaluate/`):**
   * Recibe `probabilidad` ($1-5$), `impacto` ($1-5$) y `justificacion`. Resuelve la instancia de `CELDA_MATRIZ` sobre la `CONFIGURACION_MATRIZ` activa y guarda/actualiza `EVALUACION_RIESGO`.
3. **Decisión sobre la Versión de Riesgo (`POST /api/v1/risk-versions/{id}/decision/`):**
   * En el `RiskDecisionSerializer.validate()`:
     * Si `accion == 'VALIDADA'`: exige que `activo_confirmado_id` y `agrupacion_confirmada_por` no sean nulos, que exista `EVALUACION_RIESGO` con `justificacion`, y si `CitaVersionRiesgo.objects.filter(version_id=id).count() == 0`, exige `justificacion_sin_fuentes`.
     * Si `accion == 'CORREGIDA'`: registra la decisión en la versión actual y clona una nueva `VERSION_RIESGO` con `numero = version_actual.numero + 1` bajo el mismo `riesgo_id`.
     * Si `accion == 'RECHAZADA'`: exige `observacion` (motivo del rechazo) sin requerir `EVALUACION_RIESGO`.
4. **Endpoint de Registro e Indicadores (`GET /api/v1/risks/dashboard/`):**
   * Utiliza `Subquery` / `DISTINCT ON (riesgo_id)` en el ORM de Django para agregar métricas exclusivamente sobre la última `VERSION_RIESGO` validada de cada `RIESGO`, evitando duplicar conteos cuando un riesgo tiene múltiples versiones.

### Criterios de Aceptación (DoD - Etapa 6)
* [ ] El serializer de DRF rechaza con `HTTP 400 Bad Request` cualquier intento de validar un riesgo sin fuentes estratégicas si `justificacion_sin_fuentes` está vacío.
* [ ] Publicar una nueva `CONFIGURACION_MATRIZ` cambia `activa = True` a la nueva matriz sin alterar los `celda_id` históricos ya registrados en `EVALUACION_RIESGO`.