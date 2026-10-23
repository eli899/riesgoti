# RiesgoTI

Plataforma de gestión de incidencias operativas y gobierno de riesgos de Seguridad de la Información, alineada con ISO/IEC 27005, 27001 y 27002. Cada incidencia reportada recibe una orientación asistida por IA (RAG sobre documentos normativos) y alimenta en segundo plano el análisis de riesgos que validan los Analistas.

| Capa | Tecnología | Carpeta |
| --- | --- | --- |
| Backend | Django 5.2 + DRF + Celery, PostgreSQL 16, Redis 7, FAISS | `back/` |
| Frontend | React 19 + TypeScript + Vite 8 + RTK Query + Tailwind | `front/` |
| Infraestructura | Docker Compose (`db`, `redis`, `pgadmin`, `backend`, `celery_worker`) | `docker-compose.yml` |

El backend y sus servicios corren en contenedores. El frontend corre en tu máquina con Node.js.

`back/` y `front/` son submódulos de Git con su propio repositorio. Clona el proyecto con `git clone --recurse-submodules` (ver [sección 10](#10-control-de-versiones-superproyecto-y-submódulos)); sin los submódulos, `docker compose` no encuentra `./back`.

---

## 1. Requisitos previos

| Herramienta | Versión | Para qué |
| --- | --- | --- |
| [Git](https://git-scm.com/) | 2.34 o superior | Clonar el superproyecto y sus submódulos `back/` y `front/` |
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) (con Docker Compose v2) | 24 o superior | Base de datos, Redis, pgAdmin, API y worker de Celery |
| [Node.js](https://nodejs.org/) | 20.19+ o 22.12+ (lo exige Vite 8) | Servidor de desarrollo del frontend |
| npm o [pnpm](https://pnpm.io/) | npm 10+ / pnpm 9+ | Dependencias del frontend (el repositorio incluye `pnpm-lock.yaml`) |

Puertos que deben estar libres: `5432` (PostgreSQL), `6379` (Redis), `5050` (pgAdmin), `8000` (API) y `5173` (frontend).

No necesitas Python instalado en tu máquina: todo el backend se ejecuta dentro del contenedor.

---

## 2. Configurar las variables de entorno

### 2.1 Backend (`back/.env`)

Copia la plantilla:

```bash
cp back/.env.example back/.env
```

En Windows PowerShell:

```powershell
Copy-Item back/.env.example back/.env
```

Luego reemplaza estos dos valores en `back/.env`:

| Variable | Qué es | Cómo generarla |
| --- | --- | --- |
| `SECRET_KEY` | Clave de Django | Cualquier cadena aleatoria larga. |
| `RIESGOTI_MASTER_KEY` | Clave maestra AES-256-GCM que cifra las API keys de la IA. Debe ser de 32 bytes en Base64. **Obligatoria:** sin ella el backend no arranca. | Ver comandos abajo. |

Generar `RIESGOTI_MASTER_KEY` sin instalar Python:

```bash
# Con Docker (cualquier sistema)
docker run --rm python:3.12-slim python -c "import os, base64; print(base64.b64encode(os.urandom(32)).decode())"
```

```powershell
# Con PowerShell
$b = New-Object byte[] 32; [Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($b); [Convert]::ToBase64String($b)
```

> No cambies `RIESGOTI_MASTER_KEY` después de guardar API keys desde la aplicación: las claves cifradas con la anterior dejarán de poder leerse.

El resto de valores (`DB_*`, `PGADMIN_*`, `CELERY_*`, `CORS_ALLOWED_ORIGINS`) ya funcionan para desarrollo local. `back/.env` contiene secretos: no lo subas al repositorio.

### 2.2 Frontend (`front/.env`)

```bash
cp front/.env.example front/.env
```

| Variable | Valor por defecto | Qué es |
| --- | --- | --- |
| `VITE_API_BASE_URL` | `http://localhost:8000/api/v1` | URL de la API. |
| `VITE_APP_NAME` | `RiesgoTI — …` | Nombre que muestra la aplicación. |
| `VITE_DEMO_PASSWORD` | (comentada) | Opcional, solo desarrollo. Si la defines con la contraseña de los usuarios semilla, el login muestra botones para entrar con un clic con cada perfil. No la subas al repositorio. |

---

## 3. Levantar el backend

Desde la raíz del proyecto:

```bash
# 1. Construir las imágenes y levantar los 5 servicios
docker compose up -d --build

# 2. Comprobar que estén en estado "running"
docker compose ps

# 3. Crear las tablas de docs/bd.sql, las PK compuestas y los triggers
docker compose exec backend python manage.py migrate

# 4. Cargar los datos semilla
docker compose exec backend python manage.py seed_riesgoti
```

La primera construcción tarda varios minutos porque instala PyTorch (CPU), PyMuPDF y FAISS. El primer análisis de documentos también descarga el modelo de embeddings `paraphrase-multilingual-mpnet-base-v2`; queda guardado en el volumen `huggingface_cache` y no se vuelve a descargar.

`seed_riesgoti` se puede ejecutar varias veces sin duplicar datos. Carga roles, permisos, 6 usuarios de prueba, activos, la matriz 5×5, la configuración IA/RAG inicial, documentos de ejemplo, incidencias y riesgos de demostración.

Verifica que la API responda:

```bash
curl http://localhost:8000/api/v1/health/
```

Debe devolver `"status": "ok"`, `"database": "connected"` y `"master_key_loaded": true`.

---

## 4. Levantar el frontend

```bash
cd front
pnpm install      # o: npm install
pnpm dev          # o: npm run dev
```

Abre [http://localhost:5173](http://localhost:5173).

---

## 5. Direcciones y usuarios

| Servicio | URL | Acceso |
| --- | --- | --- |
| Aplicación web | http://localhost:5173 | Usuarios semilla (tabla siguiente) |
| API REST | http://localhost:8000/api/v1/ | JWT (`POST /api/v1/auth/login/`) |
| Documentación OpenAPI (Swagger) | http://localhost:8000/api/docs/ | Pública en desarrollo |
| Esquema OpenAPI | http://localhost:8000/api/schema/ | Pública en desarrollo |
| pgAdmin 4 | http://localhost:5050 | `PGADMIN_DEFAULT_EMAIL` / `PGADMIN_DEFAULT_PASSWORD` de `back/.env`. El servidor «RiesgoTI PostgreSQL 16 (Docker)» ya aparece configurado. |
| PostgreSQL | `localhost:5432` | `DB_USER` / `DB_PASSWORD` de `back/.env` |

Usuarios semilla:

| Rol | Correos |
| --- | --- |
| Reportante | `lucia.ramirez@empresa.pe`, `jorge.salas@empresa.pe` |
| Analista | `andrea.quispe@empresa.pe`, `martin.torres@empresa.pe` |
| Administrador | `carla.mendoza@empresa.pe`, `diego.vargas@empresa.pe` |

Todos comparten la contraseña semilla definida en `SEED_PASSWORD` dentro de `back/apps/accounts/management/commands/seed_riesgoti.py`. Úsala solo en desarrollo.

Por segregación de funciones (ISO/IEC 27001), el Administrador configura el sistema pero no puede evaluar ni validar riesgos: para probar ese flujo entra como Analista.

---

## 6. Activar la IA con un proveedor real (opcional)

La configuración semilla usa Google Gemini (`gemini-2.5-flash`) para redactar y Sentence Transformers (local) para los embeddings, pero **no incluye API key**. Sin clave, la orientación y el análisis de riesgos funcionan en modo local limitado.

Para usar el modelo real:

1. Entra como Administrador y abre **Configuración IA/RAG**.
2. Escribe tu API key en **API key del LLM**.
3. Pulsa **Probar conexión** y comprueba que el modelo de lenguaje responda.
4. Pulsa **Publicar nueva versión**.

La clave se guarda cifrada con `RIESGOTI_MASTER_KEY` y la aplicación solo muestra una máscara.

---

## 7. Comandos útiles

| Tarea | Comando |
| --- | --- |
| Ver logs de la API | `docker compose logs -f backend` |
| Ver logs del worker de IA | `docker compose logs -f celery_worker` |
| Ejecutar las pruebas del backend | `docker compose exec backend pytest` |
| Re-encolar tareas de IA pendientes | `docker compose exec backend python manage.py requeue_ai_tasks` |
| Ver qué tareas se re-encolarían | `docker compose exec backend python manage.py requeue_ai_tasks --dry-run` |
| Liberar tareas atascadas en PROCESANDO | `docker compose exec backend python manage.py requeue_ai_tasks --stale-minutes 10` |
| Abrir una consola de Django | `docker compose exec backend python manage.py shell` |
| Reiniciar el worker tras cambiar código de tareas | `docker compose restart celery_worker` |
| Revisar tipos, lint y compilar el frontend | `cd front && pnpm build && pnpm lint` |
| Detener todo (conserva los datos) | `docker compose down` |
| Detener y **borrar** BD, Redis y caché de modelos | `docker compose down -v` |

La API se recarga sola al editar `back/` porque el código está montado en el contenedor. El worker de Celery no: reinícialo después de cambiar tareas.

---

## 8. Solución de problemas

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| El backend se reinicia en bucle con `RIESGOTI_MASTER_KEY es obligatoria` | Falta la clave en `back/.env` | Genérala (sección 2.1) y ejecuta `docker compose up -d`. |
| `port is already allocated` al levantar Docker | Otro programa usa 5432, 6379, 5050 u 8000 | Detén ese servicio o cambia `DB_PORT_HOST`, `REDIS_PORT_HOST` o `PGADMIN_PORT` en `back/.env`. |
| El login muestra un error de red | La API no está arriba o la URL no coincide | Revisa `docker compose ps` y `VITE_API_BASE_URL` en `front/.env`. |
| Error de CORS en el navegador | El frontend corre en otro puerto u host | Agrega el origen a `CORS_ALLOWED_ORIGINS` en `back/.env` y reinicia `backend`. |
| `relation "USUARIO" does not exist` | No se aplicaron las migraciones | Ejecuta `migrate` y luego `seed_riesgoti` (sección 3). |
| Una incidencia se queda en «Preparando una orientación» | El worker no está corriendo o hay tareas sin encolar | Revisa `docker compose logs celery_worker` y ejecuta `requeue_ai_tasks`. |
| Los documentos quedan en «Procesando PDF» | El worker sigue descargando el modelo de embeddings la primera vez | Espera unos minutos y revisa los logs del worker. |
| `npm run dev` falla con un error de versión de Node | Node.js anterior a 20.19 | Actualiza Node.js. |

---

## 9. Estructura del repositorio

| Ruta | Contenido |
| --- | --- |
| `back/` | API Django: `apps/` (accounts, assets_matrix, ai_config, documents_rag, incidents, risks, audit), `config/`, `storage/` (PDFs, evidencias, índices FAISS) |
| `front/` | Aplicación React: `src/app/api` (RTK Query), `src/mappers`, `src/hooks`, `src/pages`, `src/components`, `src/schemas` (Zod) |
| `back/roadmap/`, `front/roadmap/` | Hojas de ruta de implementación por versión (`v0.0` a `v0.6`); solo locales, no se versionan |
| `docker-compose.yml` | Servicios de desarrollo |
| `docs/bd.sql` | Esquema de base de datos de referencia (los modelos Django lo replican 1:1) |
| `docs/hoja_de_ruta_implementacion_backend.md`, `docs/hoja_de_ruta_implementacion_frontend.md` | Hojas de ruta maestras de implementación (visión general de la que derivan los roadmaps por versión) |
| [`docs/manuales/`](docs/manuales/README.md) | Manuales de usuario en Markdown por rol: [Reportante](docs/manuales/manual_usuario_reportante.md), [Analista](docs/manuales/manual_usuario_analista.md) y [Administrador](docs/manuales/manual_usuario_administrador.md) |
| `docs/manuales/capturas/` | Capturas de pantalla usadas en los manuales de usuario |
| `changelogs/` | Registro de cambios iterativos (`INDEX.md` y `entries/`); solo local, no se versiona |
| `agents/`, `CLAUDE.md`, `AGENTS.md` | Reglas, Brain (`agents/brain/project_memory.json`) y skills del ecosistema de agentes |

---

## 10. Control de versiones (superproyecto y submódulos)

El repositorio raíz es un **superproyecto** que agrupa dos submódulos con historial propio:

| Repositorio | Ruta | Contenido versionado |
| --- | --- | --- |
| `riesgoti` (superproyecto) | `/` | `docker-compose.yml`, `docs/`, `agents/`, `.claude/`, `.agents/`, `CLAUDE.md`, `AGENTS.md`, `README.md`, `.mcp.json` y las referencias a los submódulos (`.gitmodules`) |
| `riesgoti-back` | `back/` | API Django, Dockerfile, configuración y `storage/` vacío (solo `.gitkeep`) |
| `riesgoti-front` | `front/` | Aplicación React y su configuración |

No se versionan: `.env` (solo `.env.example`), `desktop.ini` (Google Drive), `node_modules/`, el contenido de `back/storage/`, los roadmaps (`back/roadmap/`, `front/roadmap/`) ni `changelogs/`. Las reglas están en el `.gitignore` de cada repositorio, y los `.gitattributes` fijan el fin de línea en LF porque `back/` se monta en contenedores Linux.

### 10.1 Clonar el proyecto

```bash
git clone --recurse-submodules https://github.com/<usuario>/riesgoti.git
# Si ya clonaste sin submódulos:
git submodule update --init --recursive
```

### 10.2 Trabajo diario

Haz el commit primero dentro del submódulo y después en la raíz, para que el superproyecto apunte al commit nuevo:

```bash
cd back && git add -A && git commit -m "..." && git push && cd ..
git add back && git commit -m "Actualiza back" && git push
```

Para traer los últimos cambios de ambos submódulos: `git submodule update --remote --merge`.
