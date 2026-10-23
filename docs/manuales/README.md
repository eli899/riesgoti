# Manuales de usuario de RiesgoTI

Un manual por rol. Cada uno explica las pantallas a las que ese rol tiene acceso, con capturas reales del sistema, y cierra con los estados, los mensajes de error frecuentes y las buenas prácticas.

| Manual | Para quién | Qué cubre |
| --- | --- | --- |
| [Reportante](manual_usuario_reportante.md) | Personal que reporta problemas de TI | Inicio de sesión, reporte de incidencias con evidencias, sugerencia de solución generada por IA y sus citas normativas, y seguimiento de sus casos. |
| [Analista](manual_usuario_analista.md) | Analistas de soporte N2 y de riesgos | Atención de incidencias escaladas, propuestas de riesgo, agrupación de incidencias, evaluación en la matriz 5×5, decisiones (validar, corregir, rechazar) y registro de riesgos. |
| [Administrador](manual_usuario_administrador.md) | Administradores de la plataforma | Configuración de IA/RAG, carga y autorización de documentos normativos, matriz de evaluación, activos, usuarios, roles y permisos, y auditoría. |

Las capturas están en [`capturas/`](capturas/). Los prefijos `rep-`, `ana-` y `adm-` indican el manual al que pertenece cada una; `login.png` y `mi-cuenta.png` son compartidas. Los diagramas de estados están escritos en Mermaid, que GitHub y la vista previa de Markdown de VS Code muestran como gráfico.
