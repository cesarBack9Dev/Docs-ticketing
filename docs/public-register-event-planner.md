# Documentación: Public Register Event Planner (MVP)

## Endpoint

```
POST /api/v1/eventplanners/public/register
```

- **Auth**: No requiere autenticación (público)
- **Objetivo**: crear un **usuario organizador** (`userRoll: "event_planner"`) y su **Event Planner** en un solo paso, devolviendo un **token de sesión (JWT)**.

---

## Payload

### Campos requeridos

| Campo | Tipo | Descripción |
|------|------|-------------|
| `firstName` | String | Nombre del usuario |
| `lastName` | String | Apellido del usuario |
| `password` | String | Contraseña del usuario (se hashea) |
| `phone` | String | Teléfono |
| `document` | String | Documento / identificación |
| `email` | String | Email del usuario (se normaliza a lowercase) |
| `organizationName` | String | Nombre del Event Planner (también acepta `eventPlannerName` o `name`) |

### Campos opcionales

| Campo | Tipo | Descripción |
|------|------|-------------|
| `eventPlannerName` | String | Alternativa a `organizationName` |
| `name` | String | Alternativa a `organizationName` |
| `code` | String | Código del Event Planner (si no se envía, se genera) |
| `contactEmail` | String | Email de contacto del Event Planner |
| `logo` | String | Logo del Event Planner |

### Ejemplo mínimo

```json
{
  "firstName": "Juan",
  "lastName": "Pérez",
  "password": "MiPassword123",
  "phone": "04141234567",
  "document": "V12345678",
  "email": "juan@example.com",
  "organizationName": "Mi Organización de Eventos"
}
```

---

## Generación automática del `code` (si NO se envía)

- Se usa el nombre: `organizationName || eventPlannerName || name`
- Se normaliza a **mayúsculas**, sin acentos y sin símbolos/espacios
- Se toma una base de **6 caracteres**
- Si hay colisión, se intenta: `BASE`, `BASE1`, `BASE2`, ... (hasta 10 intentos)

---

## Respuesta

### Éxito (200)

Retorna:
- `eventPlanner`
- `user` (sin `password`)
- `token` (JWT)

```json
{
  "message": "Okey",
  "eventPlanner": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Mi Organización de Eventos",
    "code": "MIORGA",
    "adminId": "507f1f77bcf86cd799439012"
  },
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "firstName": "Juan",
    "lastName": "Pérez",
    "phone": "04141234567",
    "document": "V12345678",
    "email": "juan@example.com",
    "userRoll": "event_planner"
  },
  "token": "<JWT>"
}
```

---

## Errores comunes

- `400`: faltan datos requeridos
- `409`: `code` ya registrado (si lo envías) o email duplicado (índice único)

---

## Nota (MVP)

La verificación explícita de “email ya existe” está **comentada** intencionalmente en el caso de uso para mostrar un MVP; si el email ya existe, MongoDB puede disparar un `duplicate key` que se responde como **409**.

