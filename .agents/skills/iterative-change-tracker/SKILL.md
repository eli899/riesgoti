---
name: iterative-change-tracker
description: Protocolo obligatorio de 3 etapas (Análisis + Brain + Contratos/Mappers -> Puerta de Aprobación Previa -> Ejecución + Changelog + Brain) para cualquier modificación, ajuste visual, corrección de bug o mejora iterativa sobre archivos existentes del proyecto RiesgoTI.
---

# Skill: Control Estricto de Iteraciones y Changelogs (`iterative-change-tracker`)

## Propósito y Activación Automática
Esta skill se activa **automáticamente y sin excepción** cada vez que el usuario solicita cualquier modificación, ajuste de interfaz, cambio de validación, refactorización, corrección de error o mejora iterativa (por pequeña que sea) sobre archivos existentes del proyecto.

> **PROHIBICIÓN ESTRICTA:** El agente **TIENE PROHIBIDO** editar o modificar archivos de código directamente en el primer turno de una solicitud de cambio. Debe ejecutar secuencialmente las **3 Etapas** descritas a continuación.

---

## Regla Crítica de Normalización y Corrección Ortográfica
Cuando el usuario escriba instrucciones rápidas en el chat, puede incluir errores de tipeo, omisión de tildes, abreviaturas o redacción coloquial (ej. *"boto"* en lugar de *"botón"*, *"qita el camp"* en lugar de *"elimina el campo"*).
1. **PROHIBIDO** transcribir o guardar errores tipográficos literalmente en `/changelogs/INDEX.md`, en `/changelogs/entries/CHG-XXXX_*.md`, en nombres de archivos (`slug`) o en `./agents/brain/project_memory.json`.
2. El agente **DEBE corregir ortográficamente, limpiar y redactar con lenguaje técnico profesional** la solicitud antes de presentarla en la Etapa 2 y antes de persistirla en el Changelog o en el Brain, manteniendo intacta la intención original del usuario.

---

## Protocolo Obligatorio de 3 Etapas

### ETAPA 1: Consulta del Brain, Inspección de Código y Evaluación de Contratos/Mappers (Solo Lectura)
1. **Consultar el Brain (`./agents/brain/project_memory.json`):**
   * Identificar el módulo (`MOD-01` a `MOD-10`), los componentes relacionados, las dependencias en Backend/Frontend/BD (`bd.sql`), las 5 invariantes de dominio y cualquier cambio previo (`CHG-XXXX`) registrado sobre esos archivos.
2. **Inspección del Código Fuente y Cumplimiento de Clean Architecture (`/agents/rules/clean_code_and_architecture.md`):**
   * Localizar y leer los archivos exactos que serían intervenidos para comprender las líneas afectadas y su impacto colateral.
   * Verificar que la solución propuesta respete las 5 capas de Frontend (`Servicio API -> Mapper/Adapter -> Custom Hook -> Página Contenedora -> Componente Presentacional`), HTML5 semántico + SEO (`<PageMeta />`, un único `<h1>`) y DTOs/Serializers de Entrada/Salida en Backend.
3. **Evaluación Obligatoria de Impacto en Contratos API / Mappers (Política Dual):**
   * Determinar explícitamente bajo cuál de los dos escenarios arquitectónicos se ejecutará el cambio:
     - **Escenario A (Modificación Aislada — Preservar Retrocompatibilidad):** El cambio es interno a una capa o afecta un componente sin alterar el contrato compartido; el `Mapper/Adapter` (`fromApiToDomain` / `fromDomainToApi`) o el `OutputSerializer` absorbe la transformación para no romper otros consumidores.
     - **Escenario B (Evolución de Funcionalidad — Cambio Sincronizado Full-Stack):** El usuario solicita agregar, modificar o eliminar campos de negocio de extremo a extremo; se deben actualizar en cadena `Modelo -> Input/Output DTO Serializer -> Servicio/ViewSet -> Contrato API TS + Zod -> Mapper/Adapter -> Custom Hook -> Página Contenedora -> Componente UI Presentacional`.

---

### ETAPA 2: Puerta de Aprobación (`STOP & CONFIRM`)
**PROHIBIDO editar archivos de código en este paso.** El agente debe detenerse y responder al usuario en el chat utilizando exactamente esta estructura:

```markdown
### Análisis Previo de Cambio Iterativo (`iterative-change-tracker`)

* **Requerimiento Normalizado (Entendimiento):** [Solicitud del usuario corregida ortográficamente y expresada con precisión técnica y profesional].
* **Contexto del Brain:** [Módulos, tablas de `bd.sql`, invariantes o relaciones detectadas en `./agents/brain/project_memory.json` que interactúan con este cambio].
* **Estrategia de Contrato y Mappers (Política Dual):** [Indicar explícitamente si aplica **Escenario A (Retrocompatibilidad vía Mapper/Adapter)** o **Escenario B (Evolución Sincronizada Full-Stack en cadena)** y detallar qué DTOs/Mappers se ajustarán].
* **Archivos Afectados y Justificación:**
  1. `[ruta/exacta/archivo_1]`: [Explicación técnica de por qué es necesario modificar este archivo].
  2. `[ruta/exacta/archivo_2]`: [Explicación técnica de por qué es necesario modificar este archivo].
* **Impacto Colateral:** [Detalle de si afecta contratos de API REST, esquemas Zod, Mappers, serializers DRF, tablas de `bd.sql` o estado compartido en Redux/RTK Query].

¿Confirmas que proceda con estos cambios?
```
*(El agente debe finalizar su turno aquí y esperar la confirmación explícita del usuario antes de pasar a la Etapa 3).*

---

### ETAPA 3: Ejecución, Generación de Changelog y Actualización del Brain
Únicamente después de recibir la confirmación explícita del usuario:

1. **Aplicar el Cambio en el Código:**
   * Realizar las ediciones exactas acordadas en la Etapa 2 cumpliendo `/agents/rules/clean_code_and_architecture.md` y verificar que no existan errores de sintaxis o tipos.
2. **Crear el Archivo de Detalle en `/changelogs/entries/`:**
   * Determinar el siguiente número secuencial de 4 dígitos (`CHG-0001`, `CHG-0002`, ...) consultando `/changelogs/INDEX.md`.
   * Construir el nombre del archivo con el formato limpio y sin faltas ortográficas:
     `changelogs/entries/CHG-[XXXX]_[YYYY-MM-DD]_[slug_limpio_y_descriptivo].md`
     *(Ejemplo: `changelogs/entries/CHG-0001_2026-10-22_aumentar_tamano_boton_guardar.md`)*.
   * Redactar el documento siguiendo estrictamente `/changelogs/TEMPLATE.md`, incluyendo:
     * **ID del Cambio, Fecha y Módulo afectado.**
     * **Descripción del Cambio Solicitado:** Redacción corregida, clara y profesional (jamás copiar errores de tipeo del prompt original).
     * **Motivo Técnico, Escenario de Contrato (A o B) y Archivos Modificados:** Tabla con ruta de cada archivo y explicación de por qué se intervino.
     * **Comparativa de Código (Antes vs. Después):** Bloques de *Código Anterior*, *Código Posterior* y bloque `diff` mostrando las líneas exactas modificadas en cada archivo.
     * **Impacto en el Sistema:** Cambios en UI, payloads, Mappers, esquemas o reglas de negocio.
3. **Actualizar `/changelogs/INDEX.md`:**
   * Agregar la nueva fila en la tabla cronológica maestra (`ID | Fecha/Hora | Tipo | Resumen Profesional del Cambio | Archivo de Detalle`).
4. **Actualizar el Brain (`./agents/brain/project_memory.json`):**
   * Registrar el nodo del cambio (`CHG-XXXX`) dentro de la colección `changelogs` del Brain usando terminología técnica limpia y estandarizada, vinculándolo a los archivos y módulos modificados.
