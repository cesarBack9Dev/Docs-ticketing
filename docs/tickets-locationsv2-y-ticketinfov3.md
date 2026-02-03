# Tickets: `locationsV2` + `ticketV3`

Documentación de los **campos agregados/normalizados** por:

- `src/uses-cases/tickets/build-locations-event-v2.js`
- `src/uses-cases/tickets/ticket-info-v3.js`

y los **endpoints** que los exponen.

---

## Endpoint: locations V2 (sync)

### GET `/api/v1/tickets/locationsV2/:event`

- **Auth**: `AuthUser`
- **Path params**
  - `event`: `eventId`
- **Query params**
  - `zones` (opcional): lista de ObjectIds separados por coma. Ej: `zones=65a...,65b...`
  - `lastTimeSync` (opcional): se acepta en query; el caso de uso devuelve además `lastTimeSync` (generado server-side).

### Response (200)

```json
{
  "docs": [],
  "totalDocs": 0,
  "limit": 10000,
  "page": 1,
  "lastTimeSync": "2026-02-03T12:34:56.789Z"
}
```

### `docs[]` (qué contiene)

`docs` es un array **heterogéneo** que concatena:

- **locations** (construidas por `locationModel.buildLocations(...)` o `buildLocationsWithOutMap(...)`)
- **tickets emitidos** (normalizados)
- **tickets eliminados/cancelados** (normalizados)
- `blackList` (actualmente vacío en este caso de uso)

> Nota: el shape exacto de los objetos **location** depende de `locationModel.buildLocations*`. Este documento se enfoca en los campos que se agregan/normalizan para los **tickets** devueltos por este endpoint.

### Campos agregados/normalizados en tickets (emitidos + cancelados)

Se normalizan tickets emitidos (`ticketModel.emitTickets`) y tickets cancelados (`ticketDeletedModel.aggregate(...)`) para que tengan una estructura común:

- **`ticketRecordStatus`** *(nuevo)*
  - `"active"` para emitidos
  - `"cancelled"` para eliminados/cancelados

- **`seatMapInfo`** *(normalizado)*: objeto con la forma:

```json
{
  "seat": "12",
  "row": "A",
  "zone": "VIP",
  "section": "Grada",
  "sector": ""
}
```

  - Para tickets eliminados, `seatMapInfo` se reconstruye por agregación con `$lookup` a:
    - `zonenames` (`zone.name`) si existe
    - `plans` (`plan.name`) como fallback cuando no existe `zoneName`
    - `sections` (`section.sectionClass.name`)
    - `rows` (`row.rowNumber`)
    - `seats` (`seat.seatNumber`)

- **`userCheck`** *(reducido / normalizado)*:
  - `null` o:

```json
{ "firstName": "Ana", "lastName": "Pérez", "email": "ana@dominio.com" }
```

- **`userDeleted`** *(nuevo en cancelados / normalizado)*:
  - `null` o:

```json
{ "firstName": "Ana", "lastName": "Pérez", "email": "ana@dominio.com" }
```

- **`ticketCode` con soporte de `auxTicketCode`** *(normalización condicional)*
  - Si `event.isWithAuxTicketCode === true`, el `ticketCode` se reemplaza por `auxTicketCode` cuando exista (`auxTicketCode > null`).

### Ajustes especiales para eventos `running`

Si `event.eventType === "running"`:

- Si el ticket tiene `dorsal`, se reescribe:
  - `seatMapInfo.seat = "Dorsal: <dorsal> "`
- Si el ticket tiene `chip`, se reescribe:
  - `seatMapInfo.row = "Chip: <chip> "`
- Se asegura boolean:
  - `equipmentDelivery = equipmentDelivery || false`

---

## Endpoint: ticket info V3

### GET `/api/v1/ticketV3/:event/:ticket`

- **Auth**: `EventPlannerGroup`
- **Path params**
  - `event`: `eventId`
  - `ticket`: `ticketCode`

### Response (200/410/400/404)

Siempre responde con la forma:

```json
{ "message": "string", "statusCode": 200, "ticket": {} }
```

#### Errores de validación

- Si falta `ticketId` (param `:ticket`) o no es string:
  - `{ message: "ticketId requerido", statusCode: 400, ticket: null }`
- Si falta `eventId` (param `:event`):
  - `{ message: "eventId requerido", statusCode: 400, ticket: null }`

#### Ticket no encontrado

- `{ message: "Ticket no encontrado", statusCode: 404, ticket: null }`

#### Ticket cancelado

Si el ticket se encuentra en `ticketDeletedModel`:

- `statusCode: 410`
- `message: "Este ticket se encuentra cancelado"`
- `ticket`: el documento enriquecido (ver campos abajo)

### Acreditaciones (`AC*`)

Si `:ticket` empieza con `"AC"`:

- se consulta en `accreditationModel.findAccreditations({ event: eventId, ticketCode: ticketId })`
- si existe, devuelve:
  - `{ message: "Credencial encontrada", statusCode: 200, ticket: <accreditation> }`

### Campos agregados/normalizados en `ticket`

Para tickets “normales” (no `AC*`), el caso de uso enriquece/normaliza:

- **`user`** *(poblado)*: `{ firstName, lastName, email }`
- **`userDeleted`** *(solo si está cancelado / encontrado en `ticketDeletedModel`)*:
  - `null` o:

```json
{ "firstName": "Ana", "lastName": "Pérez", "email": "ana@dominio.com" }
```
- **`plan`** *(poblado)*: `{ name }`
- **`type`** *(nuevo)*:
  - `"Ticket"` si no es abono
  - `"Abono"` si tiene `seasonTicket`
- **`product`** *(nuevo)*:
  - `"Ticket"`: `event.title`
  - `"Abono"`: `seasonTicket.name`

- **`seatMapInfo`** *(normalizado)*: `{ seatMap, zone, section, row, seat }`
  - Si `location.seatMap` existe: resuelve `zone/section/row/seat` a valores legibles.
  - Si no existe `location.seatMap` pero existen ids en `ticket.seatMapInfo.{zone,section,row,seat}` (típico en eliminados): hace fallback y también devuelve valores legibles.
  - Si no hay info de mapa: usa `plan.name` como `zone` y el resto vacío.

#### Lógica específica para abonos (`seasonTicket`)

Si `ticketFound.seasonTicket` existe:

- valida que el abono pertenezca al evento (`seasonTicket.events` incluye `eventId`)
- deriva el estado para el evento consultado usando `checkEvents[]`:
  - si hay checkEvent para ese `eventId`:
    - `status = "issued"`
    - `issueDate = checkEvent.issueDate`
    - `userCheck = checkEvent.userCheck`
  - si no hay:
    - `status = "pending_issue"`
- elimina `checkEvents` del payload final
- si queda `status === "issued"`, puebla `userCheck` como `{ firstName, lastName, email }`

### Ajustes especiales para eventos `running`

Si `event.eventType === "running"`:

- si existe `dorsal`:
  - `seatMapInfo.seat = "Dorsal: <dorsal> "`
- si existe `chip`:
  - `seatMapInfo.row = "Chip: <chip> "`
- `equipmentDelivery = equipmentDelivery || false`

---

## Nota: endpoint legacy que apunta a V3 sin `eventId`

Existe `GET /api/v1/ticketinfo/:ticket` que usa `ticket-info-v3` internamente, pero **no envía `eventId`** al caso de uso.

Resultado esperado: `400` con `"eventId requerido"` (a menos que exista algún wrapper distinto en el callback/middleware que lo complete).

