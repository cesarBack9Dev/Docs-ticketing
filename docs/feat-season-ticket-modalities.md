# Modalidades de Season Tickets

## Resumen

El abono (season ticket) tiene un nuevo campo **`modality`** (modalidad) que define el tipo de canje. Por defecto es `byEvent` (comportamiento actual).

Implementación de dos nuevas modalidades además del modelo actual:

1. **Por uso (byUse)** – Abono con N usos canjeables en cualquier evento del abono
2. **Por saldo (byBalance)** – Abono con saldo/crédito que se descuenta al canjear eventos

---

## 1. Modalidad actual: Por evento (byEvent)

- Un ticket por cada evento del abono
- `checkEvents` registra cada evento canjeado
- No se puede canjear el mismo evento dos veces

---

## 2. Modalidad: Por uso (byUse)

### Descripción

- Season ticket con **N usos** definidos en `seasonTicket.usesCount` (ej: 10 usos)
- Cada canje consume 1 uso
- Puede usarse en **cualquier evento** de `seasonTicket.events`
- Un mismo evento puede canjearse varias veces si quedan usos
- **No puede tener mapa de asientos** (`product.seatMap` debe ser falsy)

### Ejemplo

- Abono "10 entradas flexibles" → 10 usos
- Usuario va al evento A → quedan 9 usos
- Usuario va al evento B → quedan 8 usos
- Puede repetir evento A si tiene usos disponibles

### Modelos

**season-ticket-model.js**

```javascript
modality: { type: String, enum: ['byEvent', 'byUse', 'byBalance'], default: 'byEvent' },
usesCount: { type: Number }  // ej: 10 - solo para modality === 'byUse'
```

**reservation-model.js** (locations[])

```javascript
isMultiCheck: { type: Boolean },
quantityChecks: { type: Number }  // del seasonTicket.usesCount
```

**ticket-model.js**

```javascript
isMultiCheck: { type: Boolean },
quantityChecks: { type: Number },  // del seasonTicket.usesCount
numberChecked: { type: Number }    // incrementa en cada check
```

### Flujo de reserva (add-reservation-v2)

- Usa el **primer plan** del `plansGroup` (no requiere `withOutLimit`)
- Construye locations desde `locationModel.findOne({ plan })`
- Omite validación de asientos (`withOutLimit = true`)
- Enriquece `locationsMap` con `isMultiCheck: true` y `quantityChecks: seasonTicket.usesCount`

### Lógica de canje (check-ticket.js, fast-check.js)

- No validar "ya canjeado para este evento"
- Validar `numberChecked < quantityChecks`
- El evento debe estar en `seasonTicket.events`
- Al canjear: `$inc: { numberChecked: 1 }` y `$push` en `checkEvents`

---

## 3. Modalidad: Por saldo (byBalance)

### Descripción

- Season ticket con **saldo/crédito** que proviene del **precio del plan** (`plan.price`) al comprar
- El saldo inicial = `plan.price` (sin fees; se guarda en `location.price.initialBalance`)
- Al canjear un evento se descuenta el precio del evento del saldo
- No está ligado a eventos específicos; puede usarse en eventos del eventPlanner que acepten este tipo de abono
- El canje es por **valor**, no por evento
- **No puede tener mapa de asientos**

### Ejemplo

- Abono "Saldo $100" → balance inicial $100 (precio del plan)
- Evento A cuesta $15 → canje → saldo $85
- Evento B cuesta $30 → canje → saldo $55
- Puede repetir evento A si tiene saldo suficiente

### Modelos

**season-ticket-model.js**

```javascript
modality: { type: String, enum: ['byEvent', 'byUse', 'byBalance'], default: 'byEvent' },
initialBalance: { type: Number }  // fallback si plan no tiene precio
```

**reservation-model.js** (locations[].price)

```javascript
initialBalance: { type: Number }  // plan.price - saldo sin fees
```

**ticket-model.js**

```javascript
balance: { type: Number },  // saldo restante del ticket
checkEvents: [{ event, issueDate, userCheck, amountUsed }]  // amountUsed = monto descontado
```

### Origen del balance

1. `location.price.initialBalance` – precio del plan guardado al crear la reserva (sin fees)
2. `plan.price` – fallback
3. `seasonTicket.initialBalance` – último recurso

**No usar** `price.amount` ni `price.totalAmount` porque pueden incluir fees.

### Flujo de reserva (add-reservation-v2)

- Usa el **primer plan** del `plansGroup` (debe tener `price`)
- Valida que el plan tenga precio
- Enriquece `locationsMap` con `price.initialBalance: Number(loc.plan.price)`

### Lógica de canje (check-ticket.js, fast-check.js)

- Validar `ticket.balance >= precioDelEvento` (desde `eventPriceMap` o precio del plan del evento)
- Rechazar si saldo insuficiente
- Descontar: `balance -= precioDelEvento`
- Registrar en `checkEvents` con `amountUsed: precioDelEvento`

---

## 4. Plan y locations (byUse y byBalance)

- **Plan**: Se usa el **primer plan** del `plansGroup` del season ticket
- **Sin withOutLimit**: No se requiere que el plan tenga `withOutLimit`
- **Locations**: Se obtiene una location del plan (`locationModel.findOne({ plan })`) y se repite según `ticketsQuantity`
- **Sin asientos**: Se omite `validateSeatsV2`

---

## 5. Archivos modificados

| Archivo | Cambios |
|---------|---------|
| `season-ticket-model.js` | `modality`, `usesCount`, `initialBalance`, `eventPriceMap` |
| `season-ticket.js` (entity) | Validación modality, usesCount, initialBalance |
| `reservation-model.js` | `locations[].isMultiCheck`, `quantityChecks`, `price.initialBalance` |
| `ticket-model.js` | `isMultiCheck`, `quantityChecks`, `numberChecked`, `balance`, `checkEvents[].amountUsed` |
| `add-reservation-v2.js` | Flujo byUse/byBalance: primer plan, enriquecimiento locations |
| `generate-tickets-v2.js` | Mismo flujo, enriquecimiento, creación tickets con balance/quantityChecks |
| `issue-tickets.js` | byUse: multiCheckInfo desde locations o product.usesCount; byBalance: balance desde initialBalance o plan.price |
| `check-ticket.js` | Lógica byUse (usos) y byBalance (saldo) |
| `fast-check.js` | Misma lógica |
| `check-tickets-in-event.js` | Validación bulk para byUse y byBalance |
| `electronic-ticket-default.js` | Mostrar "Usos: X de N" (byUse) y "Saldo: $X" (byBalance) |
| `ticket-info.js` | `usesRemaining`, `balanceRemaining` |
| `get-user-ticket.js` | Incluir `balance`, `quantityChecks`, `numberChecked`, `seasonTicket` |
| `ticket-transaction.js` | Use case para consultar transacciones de tickets multi-check |
| `build-locations-event.js` / `build-locations-event-v2.js` | Incluir `isMultiCheck`, `quantityChecks`, `numberChecked`; status `pending_issue` si hay canjes disponibles |
| `tickets-users.js` / `userTickets` | Los tickets incluyen `quantityChecks`, `numberChecked`, `isMultiCheck` (no se excluyen en el pipeline) |

---

## 6. Perfil de usuario

En el perfil de usuario (endpoint `userTickets` / `tickets-users`), los tickets con **`isMultiCheck: true`** deben mostrar:

- **Usos totales:** `quantityChecks`
- **Usos disponibles:** `quantityChecks - numberChecked` (o 0 si ya no quedan)

Para tickets **byBalance**, mostrar además el **saldo restante:** `balance`.

---

## 7. Endpoint: Ticket Transaction

Consulta las transacciones (canjes) de un ticket multi-check.

**GET** `/api/v1/tickets/transaction/:ticketCode`

**Autenticación:** `EventPlannerGroup`

**Descripción:** Retorna la información de usos y transacciones de un ticket con `isMultiCheck: true`.

**Respuesta 200:**

```json
{
  "quantityChecks": 10,
  "numberChecked": 3,
  "transactions": [
    {
      "event": { "_id": "...", "title": "...", "date": "..." },
      "userCheck": { "_id": "...", "name": "...", "email": "..." },
      "issueDate": "...",
      "amountUsed": 1
    }
  ]
}
```

**Errores:**

- **404** – Ticket no encontrado
- **400** – El ticket no es de tipo multi-check (`isMultiCheck !== true`)

---

## 8. Flujos

### Web: add-reservation-v2 → pago → issue-tickets

1. Usuario crea reserva (add-reservation-v2)
2. Locations enriquecidas con `isMultiCheck`/`quantityChecks` (byUse) o `initialBalance` (byBalance)
3. Usuario paga
4. issue-tickets crea tickets con los datos de la reserva

### Taquilla: generate-tickets-v2

1. Reserva + tickets en un solo paso
2. Misma lógica de plan y enriquecimiento

---

## 9. Campo modality (modalidad)

El season ticket incluye el campo **`modality`** con los valores:

- `byEvent` (default) – Un canje por evento
- `byUse` – N usos canjeables
- `byBalance` – Saldo que se descuenta por evento

Si no se indica, el valor por defecto es `byEvent`, manteniendo el comportamiento anterior.

---

## 10. Compatibilidad

- Season tickets existentes: `modality` default `'byEvent'` → sin cambios
- No requiere migración de datos
