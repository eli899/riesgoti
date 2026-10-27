---
name: constants-no-magic-strings
description: "Prohibición de magic strings y números mágicos: los valores reutilizados cuyo cambio puede romper la aplicación (estados, roles, permisos, rutas, endpoints, límites, tiempos) se centralizan en *.constants.ts con as const. El texto visible de la interfaz permanece en el componente."
---

# Erradicación de Strings Mágicos y Centralización de Constantes

Esta habilidad prohíbe los *magic strings* y los números mágicos dispersos por el código fuente, imponiendo una arquitectura de constantes inmutables y fuertemente tipadas.

**Qué se centraliza (valores):** todo lo que se reutiliza y cuyo cambio puede romper la aplicación si una de sus copias queda desalineada:

- estados y enumeraciones de dominio (`bd.sql`, API);
- roles y códigos de permiso;
- rutas de navegación y endpoints REST;
- *tags* de caché, acciones y modos o identificadores de interfaz;
- números con significado: límites de campos y archivos, intervalos de *polling*, tiempos de espera y tamaños;
- claves de almacenamiento, eventos y códigos de error.

**Qué NO se centraliza (texto visible):** títulos, párrafos, etiquetas, `aria-*`, `title`, `placeholder`, *toasts* y `<PageMeta>` permanecen escritos en el componente, la página o el *hook* que los muestra. Así se identifica de un vistazo de qué trata cada vista sin abrir una constante, y cambiar un texto no rompe la aplicación. Se admiten dos excepciones porque dependen de un valor:

- los mapas de etiquetas indexados por una enumeración (`Record<EstadoX, string>`, que obliga a cubrir cada valor nuevo);
- los mensajes de validación que interpolan un límite centralizado.

En RiesgoTI estas excepciones viven en `labels.constants.ts` y `validation.constants.ts` (decisión AUD-0001 R-03, CHG-0033).

---

## 1. Lo que está ESTRICTAMENTE PROHIBIDO

- ❌ Strings de estado en lógica condicional: `if (status === 'active')` o `order.status = 'PENDING'`.
- ❌ Nombres de eventos Socket o canales IPC duplicados a mano: `socket.emit('order:created', ...)` o `ipcRenderer.invoke('printer:print', ...)`.
- ❌ Códigos de error y mensajes de validación hardcodeados en llamadas o respuestas: `res.status(409).json({ code: 'ALREADY_EXISTS' })`.
- ❌ Rutas de navegación dispersas: `<Link to="/admin/catalogs/products">`.
- ❌ Números mágicos sin contexto: `if (attempts > 5)` o `setTimeout(fn, 86400000)`.
- ❌ Endpoints REST escritos a mano: `fetch('/api/v1/incidencias/')` o `url: 'incidencias/'` en un servicio.

> El texto visible no entra en esta lista: `<h1>Registro de riesgos</h1>` o `toast.success('Usuario creado')` son correctos.

---

## 2. Centralización en Archivos `*.constants.ts` con `as const`

Toda constante debe definirse en el archivo correspondiente de su módulo (`auth.constants.ts`, `order.constants.ts`) asegurando tipado de solo lectura:

```typescript
// 1. Constantes de Estado y Roles
export const USER_ROLES = {
  ADMIN: 'ADMIN',
  OPERATOR: 'OPERATOR',
  VIEWER: 'VIEWER',
} as const;

export type UserRole = typeof USER_ROLES[keyof typeof USER_ROLES];

// 2. Canales IPC o Eventos Socket Simétricos
export const ORDER_SOCKET_EVENTS = {
  EMIT: 'order:create',
  RESPONSE: 'order:create-response',
  STATUS_CHANGED: 'order:status-changed',
} as const;

// 3. Códigos de Error Semánticos
export const ERROR_CODES = {
  ALREADY_EXISTS: 'ALREADY_EXISTS',
  NOT_FOUND: 'NOT_FOUND',
  CONCURRENCY_CONFLICT: 'CONCURRENCY_CONFLICT',
  UNAUTHENTICATED: 'UNAUTHENTICATED',
  FORBIDDEN: 'FORBIDDEN',
} as const;

// 4. Parámetros de Tiempo y Límites
export const TIME_LIMITS = {
  ACCESS_TOKEN_EXPIRES_IN_SEC: 900,       // 15 minutos
  REFRESH_TOKEN_EXPIRES_IN_SEC: 604800,   // 7 días
  MAX_LOGIN_ATTEMPTS: 5,
} as const;
```

---

## 3. Beneficios Inmediatos

- **Refactorización Segura**: Un renombre mediante el Language Server (LSP) actualiza todo el proyecto sin dejar referencias rotas.
- **Autocompletado Total**: Los desarrolladores y agentes de IA obtienen sugerencias exactas de tipos sin errores tipográficos.
