# Cuentas de Demostración y Desarrollo — RiesgoTI

Las siguientes cuentas son generadas de forma idempotente por el comando de semilla `seed_riesgoti` (`back/apps/accounts/management/commands/seed_riesgoti.py`) para pruebas y demostraciones en entornos locales. La semilla base solo crea roles, permisos, estas cuentas y los niveles de riesgo; `seed_riesgoti --demo` agrega además activos, matriz, configuración IA/RAG, documentos, incidencias y riesgos de ejemplo.

> **Nota de seguridad:** Estas credenciales corresponden únicamente al entorno de desarrollo/QA. En producción deben desactivarse o rotarse las contraseñas desde el portal de Administración.

---

## Contraseña común de desarrollo

| Entorno | Contraseña |
| :--- | :--- |
| Desarrollo / QA (`seed_riesgoti`) | `RiesgoTI2026!` |

---

## Usuarios semilla por rol

| Nombre completo | Correo institucional | Código de rol | Rol visible | Alcance y permisos principales |
| :--- | :--- | :--- | :--- | :--- |
| **Lucía Ramírez** | `lucia.ramirez@empresa.pe` | `REPORTANTE` | Reportante | Registro, edición y seguimiento de incidencias propias, recepción de orientación operativa IA y confirmación de solución. |
| **Jorge Salas** | `jorge.salas@empresa.pe` | `REPORTANTE` | Reportante | Registro, edición y seguimiento de incidencias propias, recepción de orientación operativa IA y confirmación de solución. |
| **Andrea Quispe** | `andrea.quispe@empresa.pe` | `ANALISTA` | Analista | Atención de incidencias escaladas a N2, revisión de agrupaciones IA, evaluación en matriz 5×5 y decisión de versiones de riesgo (ISO/IEC 27005). |
| **Martín Torres** | `martin.torres@empresa.pe` | `ANALISTA` | Analista | Atención de incidencias escaladas a N2, revisión de agrupaciones IA, evaluación en matriz 5×5 y decisión de versiones de riesgo (ISO/IEC 27005). |
| **Carla Mendoza** | `carla.mendoza@empresa.pe` | `ADMIN` | Administrador | Administración de activos, matriz de riesgo, documentos normativos RAG, configuración IA/RAG, usuarios, roles/permisos y auditoría (sin permisos de evaluación/validación de riesgos por SoD ISO/IEC 27001). |
| **Diego Vargas** | `diego.vargas@empresa.pe` | `ADMIN` | Administrador | Administración de activos, matriz de riesgo, documentos normativos RAG, configuración IA/RAG, usuarios, roles/permisos y auditoría (sin permisos de evaluación/validación de riesgos por SoD ISO/IEC 27001). |
