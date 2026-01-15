# Documentación: Unissue Tickets (Descanjear Tickets)

## Descripción

Este caso de uso permite revertir el canje de tickets, devolviéndolos a un estado pendiente de emisión. 

**Importante**: Se pueden enviar tanto **tickets regulares** como **tickets con abono (seasonTicket)** en la misma petición. Sin embargo, **si se envía al menos un abono, se debe proporcionar el parámetro `event`** para especificar qué evento se desea descanjear del abono.

El sistema maneja dos tipos de tickets de manera diferente:

- **Tickets con Abono (SeasonTicket)**: Elimina el registro del evento específico de los arrays `checkEvents` y `usersSync`. Requiere el parámetro `event`.
- **Tickets sin Abono**: Cambia el estado a `pending_issue` y elimina `userCheck` e `issueDate`. No requiere el parámetro `event`.

## Endpoint

```
POST /api/v1/tickets/unissue
```

## Autenticación

Requiere autenticación. El usuario se obtiene automáticamente de `httpRequest.user`.

## Parámetros de Entrada

### Body (JSON)

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `tickets` | Array\<String\> | Sí | Array de códigos de tickets a descanjear. Puede incluir tickets regulares y tickets con abono (seasonTicket) |
| `event` | String (ObjectId) | Condicional | ID del evento a descanjear. **Obligatorio si se envía al menos un ticket con abono (seasonTicket)** |

### Ejemplo de Request

```json
{
  "tickets": ["TKT001", "TKT002", "TKT003"],
  "event": "507f1f77bcf86cd799439011"
}
```

## Comportamiento

### 1. Tickets con Abono (SeasonTicket)

Cuando un ticket tiene un `seasonTicket` asociado:

- **Validación**: Si hay al menos un ticket con `seasonTicket` en el array de `tickets`, el parámetro `event` es **obligatorio**. El sistema valida esto antes de procesar.
- **Operación**: Se elimina el registro del evento específico (indicado en el parámetro `event`) de:
  - `checkEvents`: Array que contiene los eventos donde el ticket fue canjeado
  - `usersSync`: Array que contiene información de sincronización de usuarios
- **Log**: Se crea un log con tipo `unissue_season_ticket` y descripción que incluye el título del evento descanjeado

### 2. Tickets sin Abono

Para tickets regulares (sin `seasonTicket`):

- **Operación**: 
  - Cambia `status` a `"pending_issue"`
  - Elimina `userCheck` (usuario que canjeó el ticket)
  - Elimina `issueDate` (fecha de canje)
- **Log**: Se crea un log con tipo `unissue_ticket` y descripción "Ticket descanjeado"

## Respuesta Exitosa

### Status Code: 200

```json
{
  "message": "Tickets descajeados exitosamente",
  "code": 200,
  "ticketsWithSeasonTicket": 2,
  "ticketsWithoutSeasonTicket": 1
}
```

### Campos de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `message` | String | Mensaje de confirmación |
| `code` | Number | Código de estado HTTP |
| `ticketsWithSeasonTicket` | Number | Cantidad de tickets con abono procesados |
| `ticketsWithoutSeasonTicket` | Number | Cantidad de tickets regulares procesados |

## Errores

### 400 - Bad Request

#### Error: Event requerido para tickets con abono

```json
{
  "error": {
    "message": "El parámetro event es requerido cuando hay tickets con abono (seasonTicket)",
    "code": 400
  }
}
```

**Causa**: Se intentó descanjear tickets con `seasonTicket` sin proporcionar el parámetro `event`.

## Logs Generados

El caso de uso crea registros en `reservationLogModel` para cada ticket procesado:

### Para tickets con abono:
```javascript
{
  ticketCode: "TKT001",
  user: ObjectId("..."),
  type: "unissue_season_ticket",
  description: "Abono descanjeado event: [Título del Evento]"
}
```

### Para tickets regulares:
```javascript
{
  ticketCode: "TKT002",
  user: ObjectId("..."),
  type: "unissue_ticket",
  description: "Ticket descanjeado"
}
```

## Ejemplos de Uso

### Ejemplo 1: Descanjear tickets regulares

```javascript
// Request
POST /api/v1/tickets/unissue
{
  "tickets": ["TKT001", "TKT002"]
}

// Response
{
  "message": "Tickets descajeados exitosamente",
  "code": 200,
  "ticketsWithSeasonTicket": 0,
  "ticketsWithoutSeasonTicket": 2
}
```

### Ejemplo 2: Descanjear tickets con abono

```javascript
// Request
POST /api/v1/tickets/unissue
{
  "tickets": ["TKT003", "TKT004"],
  "event": "507f1f77bcf86cd799439011"
}

// Response
{
  "message": "Tickets descajeados exitosamente",
  "code": 200,
  "ticketsWithSeasonTicket": 2,
  "ticketsWithoutSeasonTicket": 0
}
```

### Ejemplo 3: Descanjear tickets mixtos (abonos y tickets regulares)

```javascript
// Request
// Nota: TKT003 y TKT004 son tickets con abono, TKT001 es un ticket regular
// Como hay abonos, el parámetro event es obligatorio
POST /api/v1/tickets/unissue
{
  "tickets": ["TKT001", "TKT003", "TKT004"],
  "event": "507f1f77bcf86cd799439011"
}

// Response
{
  "message": "Tickets descajeados exitosamente",
  "code": 200,
  "ticketsWithSeasonTicket": 2,  // TKT003 y TKT004 (abonos)
  "ticketsWithoutSeasonTicket": 1  // TKT001 (ticket regular)
}
```

**Nota**: En este ejemplo, `TKT003` y `TKT004` son tickets con abono, por lo que se requiere el parámetro `event` para especificar qué evento del abono se desea descanjear. `TKT001` es un ticket regular que se procesará independientemente.

## Notas Importantes

1. **Event requerido para abonos**: Si el array de `tickets` incluye al menos un ticket con `seasonTicket` (abono), el parámetro `event` es obligatorio. El `event` especifica qué evento del abono se desea descanjear.

2. **Mezcla de tickets**: Se pueden enviar tickets regulares y tickets con abono en la misma petición. El sistema los procesará según su tipo correspondiente.

2. **Operación atómica**: Los tickets se procesan en lotes separados (con abono y sin abono), pero todas las operaciones se ejecutan en la misma transacción.

3. **Logs**: Se crean logs individuales para cada ticket procesado, permitiendo auditoría completa de la operación.

4. **Búsqueda por código**: Los tickets se buscan por `ticketCode`, no por `_id`.

5. **No destructivo para abonos**: Para tickets con abono, solo se elimina el registro del evento específico, no se afecta el estado general del ticket ni otros eventos del abono.

## Dependencias

- `ticketModel`: Modelo de tickets
- `reservationLogModel`: Modelo de logs de reservaciones
- `eventModel`: Modelo de eventos (para obtener el título del evento)

## Ubicación del Código

- **Caso de uso**: `src/uses-cases/tickets/unissue-tickets.js`
- **Controlador**: `src/controllers/tickets-controller/unissue-tickets.js`
- **Ruta**: `src/index.js` (línea ~556)

