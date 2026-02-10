# Feature: Tasa fija por `PlanGroup` (precios en VES)

## Resumen

Este feature permite **tener precios en bolívares (VES) diferentes a la tasa de cambio del sistema**, usando una **tasa fija configurada en el `PlanGroup`** (`plansGroup.tasa`, expresada como *Bs por 1 USD*).

Aplica para:

- Consultas de **Evento** y **Abono (Season Ticket)** con `?currency=VES`.
- Consultas de **Seat Map** (evento y abono) con `?currency=VES`.
- **Creación de reservas** enviando `currency: "VES"` en el body.
- **Checkout**: si la reserva es VES, mostrar **precios en bolívares de inmediato** y solo **métodos de pago en bolívares**.

---

## 1) Crear un `PlanGroup` con `tasa`

### ¿Qué es `tasa`?

- `tasa`: número mayor a 0.
- Representa **Bs por 1 USD**.
- Si existe, **overridea la tasa global** del sistema para conversiones a bolívares.

### Endpoints

Existen dos endpoints relacionados con plan groups:

1) **Crear `PlanGroup` (create)**  
`POST /api/v1/plansgroup/create`

2) **Crear/actualizar `PlanGroup` (bulk upsert)**  
`POST /api/v1/plans/group`  
Si envías `_id`, el backend hace `updateOne`; si no, hace `insertOne`.

### Body de ejemplo (crear)

```json
{
  "name": "Precios Temporada 2026",
  "tasa": 75.35,
  "plans": [
    { "plan": "PLAN_ID_1" },
    { "plan": "PLAN_ID_2" }
  ]
}
```

Notas:

- `tasa` es opcional; si se incluye debe ser válida (> 0).
- `plans` debe ser un array.

---

## 2) Consultar Evento con precios en VES (query param `currency`)

### Endpoint

`GET /api/v1/event/:event`

### Query params

- `currency`: usar `VES` para recibir precios en bolívares cuando el evento tenga `plansGroup.tasa`.

### Ejemplo

`GET /api/v1/event/<EVENT_ID>?currency=VES`

### Comportamiento

- Si `currency=VES` y el evento tiene `plansGroup.tasa`, el backend **multiplica** el `plan.price` por la tasa y retorna el precio ya convertido a VES.
- En el response, el backend **elimina** `plansGroup.tasa` (no se expone la tasa en ese payload).

---

## 3) Consultar Abono (Season Ticket) con precios en VES (query param `currency`)

### Endpoint

`GET /api/v1/season-ticket/:seasonTicket`

### Query params

- `currency`: usar `VES` para recibir precios en bolívares cuando el abono tenga `plansGroup.tasa`.

### Ejemplo

`GET /api/v1/season-ticket/<SEASON_TICKET_ID>?currency=VES`

### Comportamiento

- Igual que en Evento: si `currency=VES` y existe `plansGroup.tasa`, los precios retornan en VES.
- Se elimina `plansGroup.tasa` del response.

---

## 4) Seat Maps: traer precios en VES (Event y Season Ticket)

### 4.1) Seat map de Evento (V3)

**Endpoint**

`GET /api/v1/event/seat-map-v3/:event`

**Query params**

- `currency=VES`: retorna precios del mapa en bolívares.
- `ticketOffice=1` (opcional): en flujo ticket-office, usa `plansGroupTicketOffice` si el evento lo tiene.

**Ejemplos**

- `GET /api/v1/event/seat-map-v3/<EVENT_ID>?currency=VES`
- `GET /api/v1/event/seat-map-v3/<EVENT_ID>?currency=VES&ticketOffice=1`

**Importante**

- Si se solicita `currency=VES`, los **precios del mapa** ya vienen en bolívares.  
  El frontend **no debe reconvertir** ni aplicar tasa local sobre estos valores.

### 4.2) Seat map de Abono (Season Ticket) (V3)

**Endpoint**

`GET /api/v1/season-tickets/seat-map-v3/:seasonTicket`

**Query params**

- `currency=VES`: retorna precios del mapa en bolívares.
- `mapNumber` (opcional): para abonos con múltiples mapas; selecciona el mapa adicional.
- `ticketOffice`, `web` (opcionales): flags de consumo según contexto.
- `locations` (opcional): lista separada por comas. Ej: `locations=loc1,loc2,loc3`

**Ejemplos**

- `GET /api/v1/season-tickets/seat-map-v3/<SEASON_TICKET_ID>?currency=VES`
- `GET /api/v1/season-tickets/seat-map-v3/<SEASON_TICKET_ID>?currency=VES&mapNumber=2`
- `GET /api/v1/season-tickets/seat-map-v3/<SEASON_TICKET_ID>?currency=VES&locations=loc1,loc2`

---

## 5) Frontend: selector de moneda (USD/VES)

### Reglas

- La moneda para bolívares es **`VES`**.
- Cuando el usuario seleccione **VES**, el frontend debe:
  - Consultar el producto (evento/abono) con `?currency=VES` para que los precios vengan en bolívares.
  - Consultar el seat map con `?currency=VES`.

### Nota de consistencia

Si el backend está devolviendo precios en VES (por `currency=VES`), el frontend debe tratar esos precios como **finales** para UI (sin aplicar tasa local).

---

## 6) Crear Reserva en VES (body `currency`)

### Endpoint

`POST /api/v1/reservationsV2`

### Requisito

Para crear una reserva en bolívares se debe enviar en el body:

- `currency: "VES"`

### Ejemplo mínimo (referencial)

```json
{
  "event": "<EVENT_ID>",
  "locations": ["<LOCATION_ID_1>", "<LOCATION_ID_2>"],
  "currency": "VES"
}
```

---

## 7) Checkout (UI/UX): reservas con `currency=VES`

Si la reserva tiene `currency === "VES"`:

- **Precios**:
  - Mostrar los precios **en bolívares de inmediato**.
  - No mostrar los precios en dólares como precio principal (evitar “primero USD y luego VES”).

- **Métodos de pago**:
  - Mostrar únicamente los métodos de pago **en bolívares**.
  - En general son los métodos VES (ejemplos comunes: `bss`, `pdv`, `bio`, `vesc`, `transferencia`, `TDCM`, `TDCV`, y otros equivalentes definidos para VES en tu integración).

---

## Checklist rápido (QA)

- Un `PlanGroup` con `tasa` configurada permite visualizar precios en VES en:
  - `GET /api/v1/event/:id?currency=VES`
  - `GET /api/v1/season-ticket/:id?currency=VES`
  - `GET /api/v1/event/seat-map-v3/:id?currency=VES`
  - `GET /api/v1/season-tickets/seat-map-v3/:id?currency=VES`
- Al crear reserva con `currency: "VES"`, el checkout debe:
  - mostrar VES directamente,
  - y limitar métodos de pago a VES.

