# Hotfix: Endpoint de Nuevas Localidades para Sincronización Offline

## Descripción

Este hotfix introduce un nuevo endpoint `/api/v1/tickets/new-locations/:event` que permite a la aplicación móvil sincronizar solo las localidades que han sido modificadas desde la última sincronización. Esto evita errores de ticket no encontrado por nuevas emisiones con localidades liberadas.

## Objetivo

El objetivo principal es permitir que la aplicación móvil:
1. Descargue todas las localidades y tickets inicialmente usando `/api/v1/tickets/locations/:event`
2. Guarde el campo `location` de cada ticket en la base de datos local
3. Guarde el `lastTimeSync` recibido en la respuesta
4. Use `/api/v1/tickets/new-locations/:event` con el `lastTimeSync` para obtener solo las localidades modificadas
5. Haga el match de tickets actualizados usando el campo `location` como punto de comparación

---

## Cambios en el Endpoint `/api/v1/tickets/locations/:event`

### Nuevo Campo: `location`

El endpoint `/api/v1/tickets/locations/:event` ahora incluye un nuevo campo `location` en cada ticket de la respuesta. Este campo contiene el ID de la localidad asociada al ticket.

**Ejemplo de respuesta:**

```json
{
  "docs": [
    {
      "_id": "6915fa1f9baf9454d4ffd3ed",
      "ticketCode": "TKT123456",
      "status": "issued",
      "location": "68e674644b573dd40aabae39",
      "seatMapInfo": {
        "seatMap": "",
        "zone": "VIP",
        "section": "",
        "row": "1",
        "seat": "1"
      }
    }
  ],
  "totalDocs": 150,
  "limit": 10000,
  "page": 1,
  "lastTimeSync": "2024-01-15T14:30:45.123Z"
}
```

### Nuevo Campo: `lastTimeSync`

La respuesta ahora incluye un campo `lastTimeSync` que contiene la fecha y hora (en formato ISO 8601 UTC) del momento en que se generó la respuesta. Este valor debe ser guardado en la base de datos local de la aplicación móvil.

**Formato:** `YYYY-MM-DDTHH:mm:ss.sssZ` (ISO 8601 UTC)

**Ejemplo:** `"2024-01-15T14:30:45.123Z"`

---

## Nuevo Endpoint: `/api/v1/tickets/new-locations/:event`

### Descripción

Este endpoint devuelve solo las localidades que han sido modificadas desde la última sincronización. Se utiliza para actualizar la base de datos local de la aplicación móvil de forma incremental.

### Endpoint

```
GET /api/v1/tickets/new-locations/:event
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Parámetros

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `event` | String (Path) | Sí | ID del evento |
| `zones` | String (Query) | No | IDs de zonas separados por comas (opcional) |
| `lastTimeSync` | String (Query) | **Sí** | Fecha y hora de la última sincronización en formato ISO 8601 UTC |

### Ejemplo de Request

```bash
GET /api/v1/tickets/new-locations/68e674644b573dd40aabae39?lastTimeSync=2024-01-15T14:30:45.123Z&zones=zone1,zone2
Authorization: Bearer {token}
```

### Respuesta

```json
{
  "auxLastTimeSync": "2024-01-15T15:45:30.456Z",
  "docs": [
    {
      "_id": "68e674644b573dd40aabae40",
      "ticketCode": "VG068e674644b573dd40aabae39LOC001",
      "status": "pending_issue",
      "location": "68e674644b573dd40aabae40",
      "seatMapInfo": {
        "seat": "A1",
        "row": "1",
        "zone": "VIP",
        "section": "Sección A",
        "sector": "",
        "table": "",
        "chair": ""
      }
    }
  ]
}
```

### Campos de la Respuesta

- **`auxLastTimeSync`**: Nueva fecha y hora de sincronización que debe ser guardada para la próxima petición (formato ISO 8601 UTC)
- **`docs`**: Array de localidades que han sido modificadas desde el `lastTimeSync` proporcionado

**Nota:** El campo se llama `auxLastTimeSync` en la respuesta, pero debe ser guardado como `lastTimeSync` en la base de datos local para usarlo en la próxima petición.

### Comportamiento

- Si se proporciona `lastTimeSync`, el endpoint devuelve solo las localidades cuyo campo `updatedAt` es mayor o igual a `lastTimeSync`
- Si no se proporciona `lastTimeSync`, el endpoint devuelve todas las localidades disponibles
- El campo `lastTimeSync` en la respuesta siempre contiene la fecha y hora actual en formato ISO 8601 UTC

---

## Implementación en la Aplicación Móvil

### Paso 1: Sincronización Inicial

Cuando la aplicación móvil necesita sincronizar por primera vez o hacer una sincronización completa:

```javascript
// 1. Llamar al endpoint de localidades
const response = await fetch('/api/v1/tickets/locations/:eventId', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
})

const data = await response.json()

// 2. Guardar todos los tickets en la BD local
for (const doc of data.docs) {
  if (doc.ticketCode) {
    // Es un ticket
    await localDB.tickets.insert({
      ticketCode: doc.ticketCode,
      location: doc.location,  // ⚠️ IMPORTANTE: Guardar el campo location
      status: doc.status,
      seatMapInfo: doc.seatMapInfo,
      // ... otros campos
    })
  } else {
    // Es una localidad disponible
    await localDB.locations.insert({
      location: doc.location || doc._id,
      seatMapInfo: doc.seatMapInfo,
      status: doc.status,
      // ... otros campos
    })
  }
}

// 3. Guardar el lastTimeSync
await localDB.syncInfo.upsert({
  eventId: eventId,
  lastTimeSync: data.lastTimeSync  // ⚠️ IMPORTANTE: Guardar para próximas sincronizaciones
})
```

### Paso 2: Sincronización Incremental

Para actualizar solo las localidades modificadas:

```javascript
// 1. Obtener el lastTimeSync guardado
const syncInfo = await localDB.syncInfo.findOne({ eventId: eventId })
const lastTimeSync = syncInfo?.lastTimeSync

if (!lastTimeSync) {
  // Si no hay lastTimeSync, hacer sincronización completa
  return await syncInitial()
}

// 2. Llamar al endpoint de nuevas localidades
const response = await fetch(
  `/api/v1/tickets/new-locations/${eventId}?lastTimeSync=${encodeURIComponent(lastTimeSync)}`,
  {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  }
)

const data = await response.json()

// 3. Actualizar localidades en la BD local usando location como clave
for (const doc of data.docs) {
  // Usar location como identificador único para hacer el match
  const existingLocation = await localDB.locations.findOne({
    location: doc.location || doc._id
  })

  if (existingLocation) {
    // Actualizar localidad existente
    await localDB.locations.update(
      { location: doc.location || doc._id },
      {
        seatMapInfo: doc.seatMapInfo,
        status: doc.status,
        // ... otros campos actualizados
      }
    )

    // Si hay un ticket asociado, actualizarlo también
    const associatedTicket = await localDB.tickets.findOne({
      location: doc.location || doc._id
    })

    if (associatedTicket) {
      await localDB.tickets.update(
        { location: doc.location || doc._id },
        {
          status: doc.status,
          seatMapInfo: doc.seatMapInfo,
          // ... otros campos
        }
      )
    }
  } else {
    // Nueva localidad
    await localDB.locations.insert({
      location: doc.location || doc._id,
      seatMapInfo: doc.seatMapInfo,
      status: doc.status,
      // ... otros campos
    })
  }
}

// 4. Actualizar el lastTimeSync
await localDB.syncInfo.upsert({
  eventId: eventId,
  lastTimeSync: data.auxLastTimeSync  // ⚠️ IMPORTANTE: Actualizar con el nuevo valor (el campo en la respuesta es auxLastTimeSync)
})
```

### Estructura de Base de Datos Local

#### Tabla: `tickets`

```sql
CREATE TABLE tickets (
  ticketCode TEXT PRIMARY KEY,
  location TEXT NOT NULL,  -- ⚠️ Campo requerido para hacer match
  status TEXT,
  seatMapInfo JSON,
  -- ... otros campos
  INDEX(location)  -- Índice para búsquedas rápidas
)
```

#### Tabla: `locations`

```sql
CREATE TABLE locations (
  location TEXT PRIMARY KEY,  -- ID de la localidad
  seatMapInfo JSON,
  status TEXT,
  -- ... otros campos
)
```

#### Tabla: `syncInfo`

```sql
CREATE TABLE syncInfo (
  eventId TEXT PRIMARY KEY,
  lastTimeSync TEXT NOT NULL  -- Formato ISO 8601 UTC
)
```

---

## Puntos Importantes

### 1. Campo `location` como Identificador Único

- El campo `location` es el **identificador único** que se debe usar para hacer el match entre tickets y localidades
- Este campo debe ser guardado en la base de datos local junto con cada ticket
- Cuando se reciben nuevas localidades, se debe usar `location` para identificar si es una actualización o una nueva localidad

### 2. `lastTimeSync` Obligatorio

- El parámetro `lastTimeSync` es **obligatorio** en el endpoint `/api/v1/tickets/new-locations/:event`
- Debe enviarse siempre en formato ISO 8601 UTC: `YYYY-MM-DDTHH:mm:ss.sssZ`
- Si no se envía, el endpoint devolverá todas las localidades (comportamiento de sincronización completa)

### 3. Manejo de Errores

Si el `lastTimeSync` proporcionado es inválido o muy antiguo:

```javascript
try {
  const response = await fetch(
    `/api/v1/tickets/new-locations/${eventId}?lastTimeSync=${lastTimeSync}`
  )
  
  if (response.status === 400) {
    // lastTimeSync inválido o muy antiguo
    // Hacer sincronización completa
    await syncInitial()
  }
} catch (error) {
  // Manejar error de red
  console.error('Error de sincronización:', error)
}
```

### 4. Formato de Fecha

El formato de fecha debe ser exactamente:
- **ISO 8601 UTC**: `YYYY-MM-DDTHH:mm:ss.sssZ`
- **Ejemplo**: `"2024-01-15T14:30:45.123Z"`

**En JavaScript:**
```javascript
const lastTimeSync = new Date().toISOString()
// Resultado: "2024-01-15T14:30:45.123Z"
```

**En otros lenguajes:**
- **Python**: `datetime.utcnow().isoformat() + 'Z'`
- **Java**: `Instant.now().toString()`
- **C#**: `DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")`

---

## Flujo Completo de Sincronización

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Sincronización Inicial                                    │
│    GET /api/v1/tickets/locations/:event                      │
│    ↓                                                          │
│    Guardar todos los tickets con campo 'location'            │
│    Guardar 'lastTimeSync'                                    │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Sincronización Incremental (Periódica)                   │
│    GET /api/v1/tickets/new-locations/:event?lastTimeSync=... │
│    ↓                                                          │
│    Recibir solo localidades modificadas                      │
│    ↓                                                          │
│    Hacer match por campo 'location'                         │
│    ↓                                                          │
│    Actualizar localidades existentes                         │
│    Insertar nuevas localidades                              │
│    Actualizar 'lastTimeSync'                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Ejemplos de Uso

### Ejemplo 1: Primera Sincronización

```bash
# Request
GET /api/v1/tickets/locations/68e674644b573dd40aabae39
Authorization: Bearer {token}

# Response
{
  "docs": [
    { "ticketCode": "TKT001", "location": "LOC001", ... },
    { "ticketCode": "TKT002", "location": "LOC002", ... },
    { "location": "LOC003", "status": "pending_issue", ... }
  ],
  "totalDocs": 1000,
  "lastTimeSync": "2024-01-15T14:30:45.123Z"
}
```

### Ejemplo 2: Sincronización Incremental

```bash
# Request (usando el lastTimeSync guardado)
GET /api/v1/tickets/new-locations/68e674644b573dd40aabae39?lastTimeSync=2024-01-15T14:30:45.123Z
Authorization: Bearer {token}

# Response (solo localidades modificadas después de lastTimeSync)
{
  "auxLastTimeSync": "2024-01-15T15:45:30.456Z",
  "docs": [
    { "location": "LOC001", "status": "issued", ... },  // Actualizada
    { "location": "LOC999", "status": "pending_issue", ... }  // Nueva
  ]
}
```

---

## Referencias

- **Endpoint de localidades completo**: `GET /api/v1/tickets/locations/:event`
- **Endpoint de nuevas localidades**: `GET /api/v1/tickets/new-locations/:event`
- **Código fuente**:
  - `src/uses-cases/tickets/build-locations-event.js`
  - `src/uses-cases/tickets/build-new-locations.js`
  - `src/controllers/tickets-controller/build-locations-event.js`
  - `src/models/location-model.js` (método `buildLocations`)

---

## Notas de Implementación

1. **Índices de Base de Datos**: Asegúrate de crear un índice en el campo `location` en la tabla de tickets para optimizar las búsquedas durante el match.

2. **Validación de lastTimeSync**: La aplicación móvil debe validar que el `lastTimeSync` recibido es válido antes de guardarlo. Si es inválido, debe hacer una sincronización completa.

3. **Manejo de Concurrencia**: Si múltiples dispositivos sincronizan al mismo tiempo, cada uno debe mantener su propio `lastTimeSync` por evento.

4. **Límites de Tiempo**: Si el `lastTimeSync` es muy antiguo (por ejemplo, más de 30 días), considera hacer una sincronización completa en lugar de incremental.

5. **Formato de Fecha Consistente**: Asegúrate de que el formato de fecha sea siempre ISO 8601 UTC para evitar problemas de zona horaria.

