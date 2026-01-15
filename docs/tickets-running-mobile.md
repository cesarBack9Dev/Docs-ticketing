# Documentación: Tickets de Eventos Tipo Running - Aplicación Móvil

Esta documentación describe los campos más importantes de los tickets de eventos tipo "running" (carreras) para la aplicación móvil de canje de tickets. El objetivo es que la aplicación móvil pueda agregar y actualizar el dorsal, chip y el estado de entrega del kit de de la carrera.

## Tabla de Contenidos

1. [Campos Importantes para la Aplicación Móvil](#campos-importantes-para-la-aplicación-móvil)
2. [Consulta de Información del Ticket](#consulta-de-información-del-ticket)
3. [Actualización de Información del Ticket](#actualización-de-información-del-ticket)
4. [Estructura de Datos](#estructura-de-datos)
5. [Ejemplos Completos](#ejemplos-completos)

---

## Campos Importantes para la Aplicación Móvil

Para eventos con eventType= "running", los tickets tienen campos especiales que se utilizan durante el proceso de canje:

### Campos Principales

| Campo | Tipo | Ubicación en Respuesta | Descripción |
|-------|------|------------------------|-------------|
| `dorsal` | String | `seatMapInfo.seat` | Número de dorsal asignado al participante. Se muestra como `"Dorsal: {número}"` en la respuesta |
| `chip` | String | `seatMapInfo.row` | Número de chip asignado al participante. Se muestra como `"Chip: {número}"` en la respuesta |
| `equipmentDelivery` | Boolean | `equipmentDelivery` | Indica si se ha entregado el kit de equipamiento al participante (true/false) |

### Notas Importantes

- **Dorsal**: Se almacena directamente en el campo `dorsal` del ticket, pero cuando se consulta la información del ticket para eventos tipo "running", se mapea a `seatMapInfo.seat` con el formato `"Dorsal: {número}"`.

- **Chip**: Se almacena directamente en el campo `chip` del ticket, pero cuando se consulta la información del ticket para eventos tipo "running", se mapea a `seatMapInfo.row` con el formato `"Chip: {número}"`.

- **Equipment Delivery**: Se almacena como un booleano en el campo `equipmentDelivery` del ticket. Indica si el kit de equipamiento (camiseta, número de dorsal físico, etc.) ha sido entregado al participante.

---

## Consulta de Información del Ticket

Existen dos endpoints principales para consultar la información de tickets de eventos tipo "running":

### 1. Consulta Individual de Ticket

#### Endpoint

```
GET /api/v1/tickets/info-v2
```

#### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

#### Parámetros

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `ticketId` | String | Sí | Código del ticket a consultar |
| `eventId` | String | Sí | ID del evento |

#### Ejemplo de Request

```bash
GET /api/v1/tickets/info-v2?ticketId=TKT123456&eventId=6915fa1f9baf9454d4ffd3ed
Authorization: Bearer {token}
```
### Respuesta para Eventos Tipo Running

Cuando el evento es de tipo "running", la respuesta incluye los campos mapeados de forma especial:

```json
[
  {
    "_id": "6915fa1f9baf9454d4ffd3ed",
    "ticketCode": "TKT123456",
    "status": "pending_issue",
    "dorsal": "123",
    "equipmentDelivery": false,
    "seatMapInfo": {
      "seatMap": "",
      "zone": "Categoría Elite",
      "section": "",
      "row": "",
      "seat": "Dorsal: 123 "
    },
    "user": {
      "firstName": "Juan",
      "lastName": "Pérez",
      "email": "juan.perez@example.com"
    },
    "plan": {
      "name": "Categoría Elite"
    },
    "contactInfo": {
      "firstName": "Juan",
      "lastName": "Pérez",
      "phone": "+584121234567",
      "document": "12345678",
      "email": "juan.perez@example.com"
    }
  }
]
```

### Campos en la Respuesta

- **`seatMapInfo.seat`**: Contiene el dorsal con formato `"Dorsal: {número}"` si el dorsal está asignado, o una cadena vacía si no.
- **`seatMapInfo.row`**: Contiene el chip con formato `"Chip: {número}"` si el chip está asignado, o una cadena vacía si no.
- **`equipmentDelivery`**: Booleano que indica si se entregó el kit (true) o no (false).

---


### 2. Extracción de Localidades y Tickets del Evento

#### Endpoint

```
GET /api/v1/tickets/locations/:event
```

Este endpoint se utiliza para extraer todas las localidades y tickets de un evento. Cuando el evento es tipo "running", los tickets incluidos en la respuesta también tienen los campos `dorsal` y `chip` mapeados a `seatMapInfo.seat` y `seatMapInfo.row` respectivamente, de la misma forma que en el endpoint de consulta individual.

#### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

#### Parámetros

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `event` | String (Path) | Sí | ID del evento |
| `zones` | String (Query) | No | IDs de zonas separados por comas (opcional) |

#### Ejemplo de Request

```bash
GET /api/v1/tickets/locations/6915fa1f9baf9454d4ffd3ed?zones=zone1,zone2
Authorization: Bearer {token}
```

#### Respuesta

La respuesta incluye un array `docs` que contiene tanto las localidades disponibles como los tickets emitidos. Para eventos tipo "running", los tickets en la respuesta tienen la misma estructura que se describe a continuación:

```json
{
  "docs": [
    // ... localidades disponibles
    {
      "_id": "6915fa1f9baf9454d4ffd3ed",
      "ticketCode": "TKT12345689",
      "status": "pending_issue",
      "equipmentDelivery": false,
      "seatMapInfo": {
        "seatMap": "",
        "zone": "Categoría Elite",
        "section": "",
        "row": "",
        "seat": ""
      }
    },
    // ... tickets emitidos
    {
      "_id": "6915fa1f9baf9454d4ffd3ed",
      "ticketCode": "TKT123456",
      "status": "pending_issue",
      "dorsal": "123",
      "equipmentDelivery": false,
      "seatMapInfo": {
        "seatMap": "",
        "zone": "Categoría Elite",
        "section": "",
        "row": "",
        "seat": "Dorsal: 123 "
      },
      // ... otros campos del ticket
    }
  ],
  "totalDocs": 150,
  "limit": 10000,
  "page": 1
}
```

**Nota:** Este endpoint se utiliza para tener todos los tickets en la aplicación móvil y hacer el canje offline. La aplicación móvil puede descargar todos los tickets del evento de una vez y realizar el canje sin necesidad de conexión a internet en tiempo real.

## Actualización de Información del Ticket

### Endpoint

```
PUT /api/v1/tickets/update-ticket-info
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Autenticación

Este endpoint requiere autenticación como **EventPlannerGroup**.

### Body Parameters

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `ticketCode` | String | Sí | Código del ticket a actualizar |
| `dorsal` | String | No | Número de dorsal a asignar |
| `removeDorsal` | Boolean | No | Si es `true`, remueve el dorsal (lo establece como cadena vacía) |
| `chip` | String | No | Número de chip a asignar |
| `removeChip` | Boolean | No | Si es `true`, remueve el chip (lo establece como cadena vacía) |
| `equipmentDelivery` | Boolean | No | Indica si se entregó el kit de equipamiento (true) o no (false) |
| `removeEquipmentDelivery` | Boolean | No | Si es `true`, establece `equipmentDelivery` como `false` |

### Ejemplo 1: Agregar Dorsal

```json
{
  "ticketCode": "TKT123456",
  "dorsal": "123"
}
```

**Respuesta Exitosa:**
```json
{
  "message": "Ticket info updated successfully"
}
```

### Ejemplo 2: Agregar Chip

```json
{
  "ticketCode": "TKT123456",
  "chip": "CHP456789"
}
```

### Ejemplo 3: Marcar Kit como Entregado

```json
{
  "ticketCode": "TKT123456",
  "equipmentDelivery": true
}
```

### Ejemplo 4: Actualizar Múltiples Campos

```json
{
  "ticketCode": "TKT123456",
  "dorsal": "123",
  "chip": "CHP456789",
  "equipmentDelivery": true
}
```

### Ejemplo 5: Remover Dorsal

```json
{
  "ticketCode": "TKT123456",
  "removeDorsal": true
}
```

### Ejemplo 6: Remover Chip

```json
{
  "ticketCode": "TKT123456",
  "removeChip": true
}
```

### Ejemplo 7: Marcar Kit como No Entregado

```json
{
  "ticketCode": "TKT123456",
  "removeEquipmentDelivery": true
}
```

### Respuestas de Error

#### Ticket No Encontrado

```json
{
  "error": {
    "message": "Ticket no encontrado"
  }
}
```

**Status Code:** 400

#### Ticket Code Requerido

```json
{
  "error": {
    "message": "Ticket code is required"
  }
}
```

**Status Code:** 400

---

## Estructura de Datos

### Modelo de Ticket (Campos Relevantes)

```javascript
{
  ticketCode: String,           // Código único del ticket
  dorsal: String,               // Número de dorsal (puede ser cadena vacía)
  chip: String,                 // Número de chip (puede ser cadena vacía)
  equipmentDelivery: Boolean,    // Estado de entrega del kit
  status: String,               // Estado del ticket: "pending_issue", "issued", etc.
  user: ObjectId,               // Referencia al usuario
  event: ObjectId,              // Referencia al evento
  contactInfo: {
    firstName: String,
    lastName: String,
    phone: String,
    document: String,
    email: String
  }
}
```

### Respuesta de Consulta (Eventos Running)

Cuando se consulta un ticket de un evento tipo "running", la respuesta transforma los campos:

```javascript
{
  // ... otros campos del ticket
  dorsal: "123",                    // Valor original
  chip: "CHP456789",                // Valor original
  equipmentDelivery: true,          // Valor original
  seatMapInfo: {
    seat: "Dorsal: 123 ",           // Dorsal mapeado aquí
    row: "Chip: CHP456789 ",        // Chip mapeado aquí
    zone: "Categoría Elite",        // Nombre del plan/categoría
    section: "",
    seatMap: ""
  }
}
```

---

## Ejemplos Completos

### Flujo Completo: Canje de Ticket en Aplicación Móvil

#### Paso 1: Consultar Información del Ticket

```bash
GET /api/v1/tickets/info-v2?ticketId=TKT123456&eventId=6915fa1f9baf9454d4ffd3ed
Authorization: Bearer {token}
```

**Respuesta:**
```json
[
  {
    "ticketCode": "TKT123456",
    "status": "issued",
    "dorsal": "",
    "chip": "",
    "equipmentDelivery": false,
    "seatMapInfo": {
      "seat": "",
      "row": "",
      "zone": "Categoría Elite"
    },
    "user": {
      "firstName": "Juan",
      "lastName": "Pérez"
    }
  }
]
```

#### Paso 2: Asignar Dorsal y Chip

```bash
PUT /api/v1/tickets/update-ticket-info
Authorization: Bearer {token}
Content-Type: application/json

{
  "ticketCode": "TKT123456",
  "dorsal": "123",
  "chip": "CHP456789"
}
```

**Respuesta:**
```json
{
  "message": "Ticket info updated successfully"
}
```

#### Paso 3: Marcar Kit como Entregado

```bash
PUT /api/v1/tickets/update-ticket-info
Authorization: Bearer {token}
Content-Type: application/json

{
  "ticketCode": "TKT123456",
  "equipmentDelivery": true
}
```

**Respuesta:**
```json
{
  "message": "Ticket info updated successfully"
}
```

#### Paso 4: Verificar Actualización

```bash
GET /api/v1/tickets/info-v2?ticketId=TKT123456&eventId=6915fa1f9baf9454d4ffd3ed
Authorization: Bearer {token}
```

**Respuesta:**
```json
[
  {
    "ticketCode": "TKT123456",
    "status": "issued",
    "dorsal": "123",
    "chip": "CHP456789",
    "equipmentDelivery": true,
    "seatMapInfo": {
      "seat": "Dorsal: 123 ",
      "row": "Chip: CHP456789 ",
      "zone": "Categoría Elite"
    },
    "user": {
      "firstName": "Juan",
      "lastName": "Pérez"
    }
  }
]
```

---

## Notas Importantes para la Aplicación Móvil

### 1. Formato de Visualización

- **Dorsal**: La aplicación móvil debe mostrar el valor de `seatMapInfo.seat` cuando el evento es tipo "running". Si contiene "Dorsal: ", extraer solo el número para mostrar.
- **Chip**: La aplicación móvil debe mostrar el valor de `seatMapInfo.row` cuando el evento es tipo "running". Si contiene "Chip: ", extraer solo el número para mostrar.
- **Kit**: Mostrar el estado de `equipmentDelivery` con un indicador visual (checkmark, badge, etc.).

### 2. Validaciones

- Antes de actualizar, verificar que el ticket existe y pertenece al evento correcto.
- Validar que el dorsal y chip sean valores válidos (no vacíos, formato correcto, etc.).
- Verificar que el usuario tenga permisos para actualizar el ticket (EventPlannerGroup).

### 3. Logs de Auditoría

Cada actualización genera un registro en el log de reservas (`reservationLogModel`) con:
- Tipo de operación: `"add_dorsal"`, `"remove_dorsal"`, `"add_chip"`, `"remove_chip"`, `"add_equipment"`, `"remove_equipmen"`
- Usuario que realizó la acción
- Fecha y hora de la operación
- Descripción de la acción

### 4. Manejo de Errores

- **400 Bad Request**: Si falta `ticketCode` o el ticket no existe
- **401 Unauthorized**: Si el token es inválido o falta
- **403 Forbidden**: Si el usuario no tiene permisos de EventPlannerGroup

### 5. Campos Opcionales

Todos los campos de actualización son opcionales. Puedes actualizar solo el dorsal, solo el chip, solo el equipmentDelivery, o cualquier combinación de ellos en una sola petición.

---

## Referencias

- **Endpoint de consulta individual**: `GET /api/v1/tickets/info-v2`
- **Endpoint de localidades y tickets**: `GET /api/v1/tickets/locations/:event`
- **Endpoint de actualización**: `PUT /api/v1/tickets/update-ticket-info`
- **Código fuente**: 
  - Consulta individual: `src/uses-cases/tickets/ticket-info-v2.js`
  - Localidades y tickets: `src/uses-cases/tickets/build-locations-event.js`
  - Actualización: `src/uses-cases/tickets/update-ticket-info.js`
  - Controladores: 
    - `src/controllers/tickets-controller/update-ticket-info.js`
    - `src/controllers/tickets-controller/build-locations-event.js`

---

## Preguntas Frecuentes

### ¿Puedo actualizar el dorsal y el chip en la misma petición?

Sí, puedes enviar ambos campos en el mismo request:

```json
{
  "ticketCode": "TKT123456",
  "dorsal": "123",
  "chip": "CHP456789"
}
```

### ¿Qué pasa si intento remover un dorsal que no existe?

No hay problema. El sistema simplemente establecerá el campo `dorsal` como una cadena vacía.

### ¿El campo `equipmentDelivery` tiene un valor por defecto?

Sí, el valor por defecto es `false`. Solo se establece como `true` cuando se envía explícitamente en la actualización.

### ¿Puedo consultar múltiples tickets a la vez?

El endpoint `GET /api/v1/tickets/info-v2` acepta un solo `ticketId` a la vez. Si necesitas consultar múltiples tickets, debes hacer múltiples peticiones.

### ¿Cómo sé si un evento es tipo "running"?

El campo `eventType` del evento debe ser `"running"`. Esto se puede verificar en la respuesta del endpoint de consulta del evento o en la respuesta del ticket si incluye información del evento.

