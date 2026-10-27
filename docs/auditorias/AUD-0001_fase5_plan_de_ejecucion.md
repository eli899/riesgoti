# AUD-0001 — Plan de ejecución de la Fase 5 (E2E con copia y guardianes)

| Campo | Valor |
| :--- | :--- |
| **Auditoría de origen** | [AUD-0001_2026-10-25_magic_strings_y_constantes.md](AUD-0001_2026-10-25_magic_strings_y_constantes.md) (hallazgo F-12 y §6 «Guardián propuesto») |
| **Fecha del análisis** | 2026-10-26 |
| **Estado** | **Cerrado el 2026-10-26.** Solo se ejecutó el lote 5a, y limitado a los 11 códigos de permiso (`CHG-0032`). Durante la ejecución el usuario decidió que el texto visible permanezca en los componentes (AUD-0001, R-03): la Fase 4 se revirtió en `CHG-0033` y los lotes 5b a 5e no se ejecutan. Este documento se conserva como registro del plan original. |
| **Changelogs previstos** | CHG-0032 (lotes 5a y 5b), CHG-0033 (lote 5c), CHG-0034 (lotes 5d y 5e) |
| **Fases previas** | 1 a 4 aplicadas en CHG-0024 a CHG-0031. Nada está commiteado todavía (submódulos `back/` y `front/` ni la raíz). |

---

## 1. Instrucciones para la nueva sesión

1. Consultar el Brain (`agents/brain/project_memory.json`, nodo `project_status.constants_governance` y changelogs CHG-0024 a CHG-0031).
2. Presentar este plan al usuario y **esperar su confirmación** («Procede con los cambios»). La preferencia expresada en fases anteriores fue ejecutar los lotes seguidos y detenerse solo si alguno falla.
3. Ejecutar lote por lote. Tras cada lote: verificación completa (§5), changelog en `changelogs/entries/`, fila en `changelogs/INDEX.md` y nodo en el Brain.
4. Al terminar: marcar la Fase 5 como aplicada en AUD-0001 (línea «Estado de la remediación» y fila de la tabla de fases) y actualizar `constants_governance.status` en el Brain.

---

## 2. Medición previa (2026-10-26, sin modificar archivos)

| Sondeo | Resultado |
| :--- | :--- |
| Guardián ESLint candidato sobre `front/src` (excluido `constants/**`) | Comparaciones de estado y `switch` sobre literales: **0**. *Toasts* con literal: **0**. Quedan **57** infracciones no detectadas por la Fase 4 (§2.1). |
| Backend: comparaciones y *kwargs* (`estado=`, `procesamiento=`, `accion=`, `rama=`, `tipo=`, `resultado=`, `origen=`, `finalidad=`, `nivel=`, `caso=`) contra literales en mayúsculas, excluidos `constants.py`, `rbac.py`, `messages.py`, `migrations/`, pruebas y `seed_*` | **0** |
| Literales de la E2E en posición de localizador (`name:`, `hasText:`, `label:`, `getByText`, `getByLabel`, `getByPlaceholder`, `toHaveText`, `toContainText`, `dialog`, `goTo`) | **390**: **326** coinciden exactamente con un valor de `copy/*` o `labels.constants.ts` (68 ambiguos, presentes en más de una clave) y **64** sin equivalente (§2.2). |
| Pares de enumeraciones frontend ↔ backend | 19 pares, más `ROLES`, `PERMISSIONS` y `AUDIT_ACTIONS` (§4, lote 5e). |

### 2.1 Infracciones en `front/src`

* **11 códigos de permiso** en atributos JSX (la Fase 1 no los detectó):
  * `pages/admin/AdminPortal.tsx` líneas 64, 79, 87, 90, 93, 97 y 101: `riesgo.consultar`, `incidencia.ver_todas`, `activo.administrar`, `matriz.administrar`, `documento.administrar`, `usuario.administrar`, `rol.administrar`.
  * `pages/staff/AnalystPortal.tsx` líneas 55, 64, 72 y 80: `riesgo.consultar`, `incidencia.ver_todas`, `atencion.ver_cola`, `riesgo.revisar_agrupacion`.
  * Corrección: `permission={PERMISSIONS.X}`.
* **16 `label="..."` de `<AsyncContent>`** (frases en minúscula que el codemod descartó): `AccountPage`, `AdminDashboardPage`, `DocsPage`, `MatrixPage`, `RolesPage`, `UsersPage`, `IncidentDetailPage`, `MyIncidentsPage`, `ReporterHomePage`, `RiskDetailPage`, `RiskProposalsQueuePage`, `RiskRegistryPage`, `RiskWorkspacePage`, `AnalystSummaryPage`, `SupportDetailPage`, `SupportListPage`.
* **28 fragmentos en JSX** entre expresiones. Pasan a plantillas tipadas en la copia del módulo:
  * `components/admin/`: `DocumentUploadModal:122` (« MB»), `RolePermissionsMatrixView:54,135,172` («de … activos», «usuarios con este rol.»), `SystemStatusArticle:40,51`.
  * `components/incidents/OperationalStrategicStatusAside:90,95`.
  * `components/risk/`: `DecisionReadinessArticle:48`, `EvaluationFormFieldset:88`, `InteractiveRiskMatrix5x5Grid:64,70`, `ProposalCardArticle:22,37,38`, `RiskRegistryTable:55,57`.
  * `components/staff/`: `HumanAttentionFormAside:61,93` («desde»), `PendingProposalsArticle:16,27`.
  * `components/ui.tsx:272` («{page} de {pages}»).
  * `pages/admin/AuditPage:46` («eventos encontrados»), `pages/admin/MatrixPage:33`.
  * `pages/risk/RiskDetailPage:68`, `pages/risk/RiskWorkspacePage:87`.
* **2 *placeholders***: `"https://"` (`AIConfigFormView:60`) y `"AUDITOR_TI"` (`RoleFormModal:53`).

> Las líneas son las del 2026-10-26; volver a ejecutar el sondeo (§3) antes de editar.

### 2.2 Literales de la E2E sin equivalente exacto (64)

| Tipo | Ejemplos | Tratamiento |
| :--- | :--- | :--- |
| Mensajes de validación | «El título debe describir brevemente el problema (mínimo 5 caracteres)», «Para cerrar el caso debe registrar la nota de resolución (mínimo 10 caracteres)», «La justificación de evaluación en matriz debe tener al menos 15 caracteres», «El solapamiento (chunk_overlap) debe ser estrictamente menor…» | Importar las funciones/mensajes de `src/constants/validation.constants.ts` (o del esquema Zod correspondiente) con el límite de `FIELD_LIMITS`. |
| Textos generados por plantilla | «Quitar error.log», «Descargar captura-vpn.png», «Página 1 de», «Fuentes estratégicas (0)», «1 celda modificada», «Intento 2», «Atender INC-0015: …», «Fecha ↑» | Llamar a la función de copia correspondiente con el argumento. |
| Datos de la semilla | «ERP Contable», «VPN corporativa», «Correo corporativo», «Servidor de archivos», «Degradación del servicio del ERP Contable», «Impresora lenta al imprimir reportes», «Acceso remoto no autorizado a la VPN», «No puedo conectarme a la VPN», «Casi segura», «Severo», «Aplicación» | Nuevo módulo `e2e/support/seed.ts` con los datos de `seed_riesgoti` usados por la suite. |
| Subcadenas parciales | «completado», «capa de texto», «pendiente», «Registrada el», «no solucionaron el problema», «el análisis de riesgo continúa de forma independiente» | Si la copia tiene el texto completo, usarlo; si no, construir una `RegExp` a partir de la copia o mantenerla documentada como excepción. |
| Rol de los usuarios de prueba | `role: 'Reportante'` en `e2e/support/users.ts` | `ROLE_LABEL[ROLES.X]`. |

---

## 3. Herramientas de apoyo

Las herramientas de la sesión anterior estaban en el *scratchpad* de esa sesión (`C:\Users\JEFFER~1\AppData\Local\Temp\claude\c--Users-JEFFERSON-Documents-Jeniffer-riesgo-ti\88677c75-4db1-4f12-bd9c-f0ca2e0549cc\scratchpad\`) y pueden no existir ya. Si faltan, reconstruirlas:

* **Sondeo ESLint** (`guard_rules.mjs` + configuración temporal): crear en `front/` un `eslint.guard-probe.config.mjs` que importe `./eslint.config.js`, añada las reglas de §4 (lote 5b) para `src/**/*.{ts,tsx}` con `ignores: ['src/constants/**']`, y ejecutar `npx eslint src -c eslint.guard-probe.config.mjs -f json`. Borrar la configuración temporal al terminar. Nota: ESLint 10 ya no incluye el formateador `unix`; usar `json`.
* **Medición de la E2E** (`e2e_locators.cjs`): transpilar con `typescript.transpileModule` cada `src/constants/copy/*.constants.ts` y `labels.constants.ts` (sustituyendo las importaciones por un *stub* `var PERMISSIONS = new Proxy({}, { get: (_, k) => k })`), aplanar los valores de texto a un mapa `texto → [rutas]` y recorrer el AST de `e2e/**/*.ts` comparando los literales en posición de localizador. Filtrar solo `.ts` (hay `desktop.ini` en las carpetas).
* **Sondeo del backend**: script `ast` ejecutado con `docker compose exec -T backend python <script>` (descrito en §2, fila 2).
* **Codemod y verificación posicional** de la Fase 4 (`copy_codemod.cjs`, `verify_positions.cjs`): la técnica está descrita en CHG-0027 y CHG-0028.

---

## 4. Plan por lotes

### Lote 5a — Corregir las 57 infracciones de `src` (CHG-0032)

* Permisos a `PERMISSIONS.X` (11).
* Etiquetas de `AsyncContent` a la copia de cada módulo, por ejemplo `COPY.LOADING_LABEL` (16).
* Fragmentos JSX a plantillas tipadas en la copia del módulo; por ejemplo `COMMON_COPY.UI.PAGE_OF(page, pages)` o `ADMIN_COPY.AUDIT_PAGE.EVENTS_FOUND(total)` (28).
* *Placeholders* a la copia (2).
* Ningún texto visible cambia; verificar por posición.

### Lote 5b — Guardián ESLint de `src` (CHG-0032)

Añadir a `front/eslint.config.js` un bloque para `src/**/*.{ts,tsx}` con `ignores: ['src/constants/**']` y `no-restricted-syntax` (con `L = '[A-Za-zÁÉÍÓÚÜÑáéíóúüñ]'`):

1. `BinaryExpression[operator=/^[!=]==$/] > Literal[value=/^[A-Z][A-Z0-9_]{2,}$/]`: estados comparados contra literales.
2. `SwitchCase > Literal[value=/^[A-Z][A-Z0-9_]{2,}$/]`.
3. `Literal[value=/^(incidencia|atencion|riesgo|activo|documento|matriz|usuario|rol|auditoria|configuracion|dashboard|orientacion)\.[a-z_]+$/]`: códigos de permiso.
4. `JSXText` con dos o más letras seguidas (se admiten abreviaturas de una letra como «v», «P», «I» y separadores como «·»).
5. `JSXAttribute[name.name=/^(aria-label|aria-description|title|placeholder|alt|label|description)$/] > Literal` con letras.
6. `CallExpression[callee.object.name='toast'] > Literal` y `> TemplateLiteral`.

Cada regla con un mensaje en español que indique a qué archivo de constantes llevar el texto. `pnpm lint` debe quedar en 0.

### Lote 5c — Suite E2E con copia (CHG-0033)

* Codemod sobre `e2e/**/*.ts` que reemplace los 326 literales por su referencia. En los 68 ambiguos, preferir el objeto del módulo del *spec* (`admin.spec` → `ADMIN_COPY`, `analyst.spec` → `STAFF_COPY`, `risk-workflow.spec` → `RISK_COPY`, `reporter.spec` → `REPORTER_COPY`/`INCIDENT_COPY`, `auth.spec` → `AUTH_COPY`) y, en su defecto, `COMMON_COPY.SHARED`.
* Resolver a mano los 64 de §2.2 y crear `e2e/support/seed.ts`.
* Añadir `"paths": { "@/*": ["./src/*"] }` a `tsconfig.e2e.json` (`labels.constants.ts` importa tipos con el alias; Playwright resuelve los `paths` del tsconfig).
* Importar la copia desde un único punto (`e2e/support/copy.ts`, que reexporte `../../src/constants/copy/*`), como ya hace `e2e/support/routes.ts` con `ROUTES`.
* Regla ESLint para `e2e/**`: prohibir literales con letras en posición de localizador (`getByText`, `getByLabel`, `getByPlaceholder`, `toHaveText`, `toContainText` y las propiedades `name`/`hasText`/`label`). Se permiten expresiones regulares, los textos que se escriben en formularios y `e2e/support/seed.ts`.
* No tocar los títulos de `test`/`describe` (son descripciones, no UI).

### Lote 5d — Guardián del backend (CHG-0034)

* Prueba de arquitectura en `pytest` que recorra `apps/**/*.py` con `ast` y falle ante:
  * comparaciones (`==`, `!=`, `in` con tuplas/listas/conjuntos) contra literales `^[A-Z][A-Z0-9_]{2,}$`;
  * *kwargs* `estado`, `procesamiento`, `accion`, `rama`, `tipo`, `resultado`, `origen`, `finalidad`, `nivel`, `estado_vinculo`, `caso` (incluidos los *lookups* `estado__in`) con literal `str`.
* Exclusiones: `constants.py`, `rbac.py`, `messages.py`, `migrations/`, `tests/`, comandos `seed_*`.
* Ubicación: junto a las pruebas transversales existentes (revisar dónde viven antes de crearla). Hoy da 0; solo previene regresiones.

### Lote 5e — Contrato frontend ↔ backend (CHG-0034)

`front/scripts/check-domain-contract.mjs`: lee los valores de `src/constants/{domain,rbac,audit}.constants.ts` y de los `StrEnum` de `../back/apps/**/constants.py` y `../back/apps/accounts/rbac.py`, y compara cada par. Si `../back` no existe (submódulo clonado por separado), termina con aviso sin fallar. Encadenarlo en `pnpm lint`.

| Frontend | Backend |
| :--- | :--- |
| `ROLES` | `accounts/rbac.py: RolCodigo` |
| `PERMISSIONS` | `accounts/rbac.py: CodigoPermiso` |
| `AUDIT_ACTIONS` | `audit/constants.py: AccionAuditoria` |
| `LLM_PROVIDER` / `EMBEDDING_PROVIDER` | `ai_config/constants.py: ProveedorLlm` / `ProveedorEmbedding` |
| `DOCUMENT_PROCESSING_STATUS` / `DOCUMENT_PURPOSE` | `documents_rag/constants.py: ProcesamientoDocumento` / `FinalidadDocumento` |
| `INCIDENT_STATUS` / `GUIDANCE_RESULT` / `INCIDENT_INBOXES` | `incidents/constants.py: EstadoIncidencia` / `ResultadoOrientacion` / `BandejaIncidencia` |
| `AI_TASK_BRANCH` / `AI_TASK_STATUS` / `GUIDANCE_TYPE` / `INCIDENT_LINK_STATUS` | `risks/constants.py: RamaTareaIA` / `EstadoTareaIA` / `TipoOrientacion` / `EstadoVinculo` |
| `RISK_DECISION_ACTION` / `RISK_VALIDATION_STATUS` / `RISK_VERSION_ORIGIN` / `RISK_INBOXES` | `risks/constants.py: AccionDecisionRiesgo` / `EstadoVersionRiesgo` / `OrigenVersionRiesgo` / `BandejaRiesgo` |

Revisar al implementar: `RISK_LEVEL`, `RISK_REGISTRY_CATEGORY`, `AUDIT_TARGET_KIND`, `DOCUMENT_DISPLAY_STATE` son solo del frontend o derivados (no comparar), y `OrigenRecomendacion` y `CasoAgrupacion` son solo del backend. Confirmar si algún par es un subconjunto intencional (por ejemplo, bandejas) y documentarlo en el script.

---

## 5. Verificación por lote

* `cd front`: `npx tsc -b`, `pnpm lint` (con los guardianes ya activos, en 0) y `pnpm build`.
* Backend: `docker compose exec backend pytest` (152 pruebas antes de la Fase 5, más la nueva), `python manage.py check` y `makemigrations --check`.
* Lotes 5a y 5c: verificación posicional (mismos textos en el mismo lugar) e inventario por AST en 0.
* E2E: `cd front; pnpm test:e2e` sobre el stack aislado recién creado (46/46). **Nunca** usar el stack de desarrollo. Si Vite no arranca en `:5174` (`ERR_CONNECTION_REFUSED`), repetir la ejecución y documentarlo.

---

## 6. Impacto y alcance

* **Contratos API, esquemas Zod, serializers y `bd.sql`:** sin cambios. Escenario A en todos los lotes.
* **Interfaz:** ningún texto visible cambia.
* **Contrapartida asumida (es lo que pide F-12):** con la E2E consumiendo la copia, un cambio de texto ya no rompe la suite, pero tampoco lo detecta. Esa revisión queda en los archivos `copy/*` y en las capturas de los manuales (`docs/manuales/capturas/`).
* **Fuera de alcance:** R-02 (mínimos validados solo en el frontend: activo, rol, matriz e incidencia) sigue pendiente de decisión del usuario; alinearlo en los serializers es un cambio de regla (Escenario B).
* **Convenciones:** redacción profesional en español sin emojis en la documentación; changelogs con la plantilla `changelogs/TEMPLATE.md`; commits solo si el usuario los pide (primero en los submódulos y luego en la raíz).
