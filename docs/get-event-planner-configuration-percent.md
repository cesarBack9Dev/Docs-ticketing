# Documentación: Get Event Planner Configuration Percent

## Descripción

Este endpoint retorna el **porcentaje de configuración** (0–100) del Event Planner autenticado y un **breakdown** por ítems de un checklist default (cada ítem tiene `weight` y `done`).

Se usa para mostrar “qué tan completo” está el perfil/configuración del organizador.

## Endpoint

```
GET /api/v1/eventplanner/configuration-percent
```

## Autenticación

Requiere autenticación por `EventPlannerGroup`.

### Header requerido

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Authorization` | String | Sí | `Bearer <token>` |

El middleware agrega `req.user.eventPlanner` (desde el token / usuario) y el endpoint usa `eventPlanner._id`.

## Parámetros de entrada

No recibe body ni query params.

## Checklist default (100 puntos)

El checklist es fijo y suma 100:

| key | weight | Criterio `done` |
|-----|--------|------------------|
| `name` | 15 | `eventPlanner.name` no vacío |
| `logo` | 10 | `eventPlanner.logo` no vacío |
| `code` | 5 | `eventPlanner.code` no vacío |
| `domain` | 10 | `eventPlanner.domain` no vacío |
| `contactEmail` | 10 | `eventPlanner.contactEmail` no vacío |
| `currency` | 5 | `eventPlanner.currency` no vacío |
| `domains` | 5 | `eventPlanner.domains.length > 0` |
| `gatewaysConfigExists` | 20 | existe documento `GateWaysConfig` para el EP |
| `gatewaysSelected` | 20 | `GateWaysConfig.gateWays.length > 0` |

## Cálculo del percent

1. Se suma el total de pesos (\(totalWeight = 100\)).
2. Se suma el peso de los ítems en `done=true` (\(doneWeight\)).
3. `percent = round((doneWeight / totalWeight) * 100)`.

## Respuesta exitosa

### Status Code: 200

```json
{
  "percent": 65,
  "breakdown": [
    { "key": "name", "label": "Nombre", "weight": 15, "done": true },
    { "key": "logo", "label": "Logo", "weight": 10, "done": false },
    { "key": "code", "label": "Código", "weight": 5, "done": true },
    { "key": "domain", "label": "Dominio principal", "weight": 10, "done": true },
    { "key": "contactEmail", "label": "Email de contacto", "weight": 10, "done": true },
    { "key": "currency", "label": "Moneda", "weight": 5, "done": true },
    { "key": "domains", "label": "Lista de dominios", "weight": 5, "done": false },
    { "key": "gatewaysConfigExists", "label": "Configuración de pasarelas creada", "weight": 20, "done": true },
    { "key": "gatewaysSelected", "label": "Pasarelas seleccionadas", "weight": 20, "done": false }
  ]
}
```

## Errores

### 401 - Not authorized

Si falta `Authorization` o el token es inválido, el middleware responde:

```json
{ "message": "Not authorized to access this resource" }
```

### 404 - Eventplanner no encontrado

Si el Event Planner no existe (por ejemplo, el middleware no inyectó `eventPlanner` o el `_id` no existe en DB):

```json
{
  "error": {
    "message": "Eventplanner no encontrado.",
    "code": 404
  }
}
```

### 400 - Bad Request

Cualquier otra excepción cae en `400` desde el controller:

```json
{
  "error": {}
}
```

## Notas importantes

- Si **no existe** `GateWaysConfig` para el EP, **no es error**: `gatewaysConfigExists=false` y `gatewaysSelected=false`.
- El endpoint recarga el Event Planner desde base de datos con `.lean()` para evaluar el checklist.

## Ejemplos

```bash
curl -X GET "http://localhost:3000/api/v1/eventplanner/configuration-percent" ^
  -H "Authorization: Bearer <TOKEN>"
```

## Ubicación del código

- **Caso de uso**: `src/uses-cases/event-planner/get-event-planner-config-percent.js`
- **Controlador**: `src/controllers/event-planner-controller/get-event-planner-config-percent.js`
- **Middleware**: `src/middleware/event-planner-group.js`
- **Ruta**: `src/index.js` (`/api/v1/eventplanner/configuration-percent`)

