# Feature: Eventos por Donaciones (`eventType: donation`)

## Resumen

Este feature permite que un `Event` defina un flujo alterno para reservas cuando su campo `eventType` sea `"donation"`.

- `eventType: "normal"` (default): **flujo actual**, sin cambios.
- `eventType: "donation"`: **flujo de donación**.

---

## 1) Crear un evento de donación

### Endpoint

- `POST /api/v1/events` (requiere permisos de Event Planner)

### Body (ejemplo)

```json
{
  "title": "Donaciones - Fundación X",
  "description": "Apoya nuestra causa",
  "start": { "date": "2026-02-03T10:00:00.000Z", "timezone": "America/Caracas" },
  "end": { "date": "2026-03-03T10:00:00.000Z", "timezone": "America/Caracas" },
  "eventPlanner": "EVENT_PLANNER_ID",
  "eventType": "donation",
  "currency": "USD"
}
```

### Nota

Si no envías `eventType`, el evento queda como `"normal"` por default.

---

## 2) Crear una reserva de donación

### Endpoint

- `POST /api/v1/reservationsV2` (AuthUser)

### Body mínimo recomendado

```json
{
  "event": "EVENT_ID",
  "donationAmount": 25,
  "aditionalsUsers": [
    {
      "firstName": "Ana",
      "lastName": "Pérez",
      "phone": "0000",
      "document": "V123",
      "email": "ana@email.com"
    }
  ]
}
```

### Reglas del flujo `donation`

Si el evento es `eventType === "donation"`:

- Valida `donationAmount` (number > 0).
- Crea la reserva **sin seatmap / sin plan / sin locations**:
  - `locations: []`
  - `taxes: []`
  - `tickets.quantity = 1`
  - `currency = event.currency || eventPlanner.currency || "USD"`
  - `price` fijo:

```js
price: {
  amount: donationAmount,
  fee: 0,
  feePaymentGateWay: 0,
  feeTicketsIssued: 0,
  b9Fee: 0,
  ancillariesAmount: 0,
  totalAmount: donationAmount
}
```

Si el evento es `normal` (o no tiene `eventType`), se ejecuta el flujo existente sin cambios.

---

## 3) `reprice` en donaciones (sin fees / sin taxes)

Cuando una reserva pertenece a un evento `donation`:

- **No** se aplican impuestos/fees (paymentGateway / IVA / IGTF / B9).
- **No** se muta `reservation.taxes`.
- **Sí** se calculan los campos VES necesarios (para flujos como BNCPM):
  - `amountVES`
  - `feeVES`
  - `totalAmountVES`
  - `feePaymentGateWayVES`

---

## 4) Emisión/confirmación (donaciones NO generan tickets)

Cuando se intenta emitir tickets (`issue-tickets`) para una reserva de donación:

- **No se crean tickets**.
- Se envía un **email de confirmación de donación**.
- El use-case retorna `[]`.

### Email

- Template: `files/donation-confirmation-template.html`
- Sender: `src/send-mail/send-mail-donation-confirmation.js`

---

## 5) Endpoints de Dashboard (Donaciones)

> Protegidos con `EventPlannerGroup` (similar a otros endpoints de dashboard).

### 5.1 Listado paginado

- `GET /api/v1/dashboard/donations`

#### Query params

- `event` (opcional): `eventId`
- `from` (opcional): fecha ISO
- `to` (opcional): fecha ISO
- `currency` (opcional): `USD` | `VES` | `EUR` (o lo que use la reserva)
- `paymentStatus` (opcional)
- `reservationStatus` (opcional)
- `page` (opcional, default `1`)
- `limit` (opcional, default `50`, max `200`)

#### Respuesta

Devuelve `{ docs, totalDocs, page, limit, totalPages }`.

---

### 5.2 Resumen / métricas

- `GET /api/v1/dashboard/donations/summary`

#### Query params

- `event` (opcional)
- `from` (opcional)
- `to` (opcional)
- `currency` (opcional)
- `paymentStatus` (opcional)
- `reservationStatus` (opcional)

#### Respuesta

- `overall`: `{ count, totalAmount }`
- `totalsByCurrency`: lista de totales por moneda
- `seriesByDay`: serie diaria (`YYYY-MM-DD`) por moneda

---

## 6) Verificación rápida (manual)

- Crear/editar un evento con `eventType: "donation"`.
- `POST /api/v1/reservationsV2` con `donationAmount`.
  - Verificar: `locations=[]`, `taxes=[]`, `tickets.quantity=1`, `price.totalAmount=donationAmount`.
- Ejecutar un flujo de pago que invoque `reprice` (ej: BNCPM):
  - Verificar: no agrega fees/taxes, pero sí retorna `totalAmountVES`.
- Emitir tickets (issue) para la reserva:
  - Verificar: no crea tickets y envía email de confirmación.
- Consultar dashboard:
  - `GET /api/v1/dashboard/donations`
  - `GET /api/v1/dashboard/donations/summary`

