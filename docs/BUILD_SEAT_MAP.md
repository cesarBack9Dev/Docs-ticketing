# API - Crear o Editar Mapa de Asientos

## Endpoint

```
POST /api/v1/seat-map/create-or-update
```

## Descripción

Este endpoint permite **crear un nuevo mapa de asientos** o **editar un mapa existente** usando el mismo endpoint. La diferencia entre crear y editar se determina por la presencia del campo `_id` en el body de la petición.

## Funcionamiento

- **Crear**: Si NO se envía el campo `_id`, se crea un nuevo mapa de asientos
- **Editar**: Si se envía el campo `_id`, se actualiza el mapa existente con ese ID

---

## Crear Mapa de Asientos

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | String | Nombre del mapa de asientos (requerido) |
| `viewBox` | String | ViewBox del SVG del mapa (requerido) |
| `imgUrlMap` | String | URL de la imagen del mapa (requerido) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `eventPlanners` | Array[String] | Array de IDs de event planners (opcional) |

### Ejemplo de Request - Crear

```json
{
  "name": "Mapa Principal - Estadio",
  "viewBox": "0 0 800 600",
  "imgUrlMap": "https://example.com/maps/stadium.svg",
  "eventPlanners": [
    "507f1f77bcf86cd799439011",
    "507f1f77bcf86cd799439012"
  ]
}
```

### Ejemplo de Response - Crear (200 OK)

```json
{
  "message": "Mapa de asientos creado exitosamente",
  "seatMap": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Mapa Principal - Estadio",
    "viewBox": "0 0 800 600",
    "imgUrlMap": "https://example.com/maps/stadium.svg",
    "eventPlanners": [
      "507f1f77bcf86cd799439011",
      "507f1f77bcf86cd799439012"
    ],
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

## Editar Mapa de Asientos

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `_id` | String | ID del mapa de asientos a editar (requerido para edición) |

### Campos Opcionales (Solo enviar los que se desean actualizar)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | String | Nuevo nombre del mapa |
| `viewBox` | String | Nuevo viewBox del SVG |
| `imgUrlMap` | String | Nueva URL de la imagen |
| `eventPlanners` | String o Array[String] | **Comportamiento especial** (ver abajo) |

### Comportamiento de `eventPlanners` en Edición

El campo `eventPlanners` tiene un comportamiento especial cuando se edita:

- **Si se envía un solo `_id` (String)**: Se agrega ese event planner al array existente (push)
- **Si se envía un Array**: Se agregan todos los event planners del array al array existente (push múltiple)

**Importante**: Los event planners siempre se **agregan** al array existente, nunca se reemplazan.

### Ejemplos de Request - Editar

#### 1. Editar solo el nombre

```json
{
  "_id": "507f1f77bcf86cd799439013",
  "name": "Mapa Principal - Estadio Actualizado"
}
```

#### 2. Editar múltiples campos

```json
{
  "_id": "507f1f77bcf86cd799439013",
  "name": "Mapa Principal - Estadio Actualizado",
  "viewBox": "0 0 1000 800",
  "imgUrlMap": "https://example.com/maps/stadium-v2.svg"
}
```

#### 3. Agregar un solo event planner (push)

```json
{
  "_id": "507f1f77bcf86cd799439013",
  "eventPlanners": "507f1f77bcf86cd799439014"
}
```

#### 4. Agregar múltiples event planners (push múltiple)

```json
{
  "_id": "507f1f77bcf86cd799439013",
  "eventPlanners": [
    "507f1f77bcf86cd799439014",
    "507f1f77bcf86cd799439015",
    "507f1f77bcf86cd799439016"
  ]
}
```

#### 5. Editar campos y agregar event planners

```json
{
  "_id": "507f1f77bcf86cd799439013",
  "name": "Nuevo Nombre",
  "eventPlanners": [
    "507f1f77bcf86cd799439014",
    "507f1f77bcf86cd799439015"
  ]
}
```

### Ejemplo de Response - Editar (200 OK)

```json
{
  "message": "Mapa de asientos actualizado exitosamente",
  "seatMap": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Mapa Principal - Estadio Actualizado",
    "viewBox": "0 0 1000 800",
    "imgUrlMap": "https://example.com/maps/stadium-v2.svg",
    "eventPlanners": [
      "507f1f77bcf86cd799439011",
      "507f1f77bcf86cd799439012",
      "507f1f77bcf86cd799439014",
      "507f1f77bcf86cd799439015"
    ],
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| 200 | Operación exitosa (creación o actualización) |
| 400 | Error de validación o datos inválidos |

---

## Errores Comunes

### Error: Campos requeridos faltantes (Creación)

**Request:**
```json
{
  "name": "Mapa Principal"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "La imagen del mapa de asientos es requerida"
  }
}
```

### Error: Mapa no existe (Edición)

**Request:**
```json
{
  "_id": "507f1f77bcf86cd799439999",
  "name": "Nuevo Nombre"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "El mapa de asientos no existe"
  }
}
```

### Error: No se enviaron campos para actualizar

**Request:**
```json
{
  "_id": "507f1f77bcf86cd799439013"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar al menos un campo para actualizar"
  }
}
```

### Error: eventPlanners debe ser array (Solo en creación)

**Request:**
```json
{
  "name": "Mapa Principal",
  "viewBox": "0 0 800 600",
  "imgUrlMap": "https://example.com/maps/stadium.svg",
  "eventPlanners": "507f1f77bcf86cd799439011"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "Los event planners deben ser un array"
  }
}
```

---

## Notas Importantes

1. **En edición, solo se actualizan los campos que se envían**. No es necesario enviar todos los campos.

2. **Los `eventPlanners` siempre se agregan** al array existente cuando se edita. Si necesitas reemplazar completamente el array, deberás usar otro endpoint o enviar el array completo con todos los valores deseados.

3. **En creación, `eventPlanners` debe ser un array** si se envía. En edición, puede ser un string (para agregar uno) o un array (para agregar varios).

4. **El campo `_id` solo se usa para edición**. Si se envía en una creación, será tratado como un intento de edición y fallará si el ID no existe.

---

## Ejemplos de Uso en JavaScript/TypeScript

### Crear un mapa

```javascript
const createSeatMap = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      name: 'Mapa Principal - Estadio',
      viewBox: '0 0 800 600',
      imgUrlMap: 'https://example.com/maps/stadium.svg',
      eventPlanners: ['507f1f77bcf86cd799439011']
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Editar solo el nombre

```javascript
const updateSeatMapName = async (seatMapId, newName) => {
  const response = await fetch('/api/v1/seat-map/create-or-update', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      _id: seatMapId,
      name: newName
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Agregar un event planner

```javascript
const addEventPlanner = async (seatMapId, eventPlannerId) => {
  const response = await fetch('/api/v1/seat-map/create-or-update', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      _id: seatMapId,
      eventPlanners: eventPlannerId  // String, no array
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Agregar múltiples event planners

```javascript
const addMultipleEventPlanners = async (seatMapId, eventPlannerIds) => {
  const response = await fetch('/api/v1/seat-map/create-or-update', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      _id: seatMapId,
      eventPlanners: eventPlannerIds  // Array
    })
  });
  
  const data = await response.json();
  return data;
};
```

---

## Resumen de Validaciones

### Creación
- ✅ `name` es requerido
- ✅ `viewBox` es requerido
- ✅ `imgUrlMap` es requerido
- ✅ `eventPlanners` debe ser array si se envía

### Edición
- ✅ `_id` es requerido
- ✅ Debe enviar al menos un campo para actualizar
- ✅ El mapa debe existir
- ✅ `eventPlanners` puede ser String o Array

---

# API - Crear o Editar Zonas (Bulk)

## Endpoint

```
POST /api/v1/seat-map/create-or-update-zone
```

## Descripción

Este endpoint permite **crear múltiples zonas** o **editar múltiples zonas** en una sola petición usando operaciones en lote (bulk). Puedes enviar un array de zonas y el sistema procesará todas las operaciones de manera eficiente, devolviendo información detallada de cada elemento procesado.

## Funcionamiento

- **Crear**: Si NO se envía el campo `_id` en un elemento, se crea una nueva zona
- **Editar**: Si se envía el campo `_id` en un elemento, se actualiza la zona existente con ese ID
- **Bulk**: Puedes mezclar creaciones y ediciones en la misma petición
- **Orden**: Los resultados se devuelven en el mismo orden que se enviaron

---

## Formatos de Request

El endpoint acepta múltiples formatos para facilitar su uso:

### Formato 1: Array directo (Recomendado)

```json
[
  { "name": "Zona Principal" },
  { "_id": "507f1f77bcf86cd799439011", "name": "Zona VIP Actualizada" },
  { "name": "Zona General" }
]
```

### Formato 2: Objeto con propiedad `zones`

```json
{
  "zones": [
    { "name": "Zona Principal" },
    { "_id": "507f1f77bcf86cd799439011", "name": "Zona VIP Actualizada" }
  ]
}
```

### Formato 3: Objeto único (Compatibilidad)

```json
{
  "name": "Zona Principal"
}
```

O para editar:

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "Zona Principal Actualizada"
}
```

---

## Campos

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | String | Nombre de la zona (requerido para cada elemento) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `_id` | String | ID de la zona a editar (opcional, solo si es edición) |

---

## Ejemplos de Request

### 1. Crear múltiples zonas

```json
[
  { "name": "Zona Principal" },
  { "name": "Zona VIP" },
  { "name": "Zona General" }
]
```

### 2. Editar múltiples zonas

```json
[
  { "_id": "507f1f77bcf86cd799439011", "name": "Zona Principal Actualizada" },
  { "_id": "507f1f77bcf86cd799439012", "name": "Zona VIP Actualizada" }
]
```

### 3. Crear y editar en la misma petición

```json
[
  { "name": "Zona Nueva 1" },
  { "_id": "507f1f77bcf86cd799439011", "name": "Zona Existente Actualizada" },
  { "name": "Zona Nueva 2" }
]
```

---

## Ejemplo de Response (200 OK)

```json
{
  "message": "Procesadas 3 zona(s): 2 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "zoneName": {
        "_id": "507f1f77bcf86cd799439013",
        "name": "Zona Nueva 1",
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "updated",
      "zoneName": {
        "_id": "507f1f77bcf86cd799439011",
        "name": "Zona Existente Actualizada",
        "createdAt": "2024-01-10T08:00:00.000Z"
      }
    },
    {
      "operation": "created",
      "zoneName": {
        "_id": "507f1f77bcf86cd799439014",
        "name": "Zona Nueva 2",
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 3,
    "created": 2,
    "updated": 1,
    "errors": 0
  }
}
```

---

## Estructura de Response

### Campos de Response

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `message` | String | Mensaje descriptivo del resultado |
| `results` | Array | Array de resultados, uno por cada elemento procesado |
| `summary` | Object | Resumen con contadores de operaciones |

### Estructura de cada elemento en `results`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `operation` | String | Tipo de operación: `"created"`, `"updated"`, o `"error"` |
| `zoneName` | Object | Objeto de la zona (solo si la operación fue exitosa) |
| `error` | String | Mensaje de error (solo si `operation` es `"error"`) |
| `_id` | String | ID de la zona (solo si hay error) |

### Estructura de `summary`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `total` | Number | Total de elementos procesados |
| `created` | Number | Cantidad de zonas creadas |
| `updated` | Number | Cantidad de zonas actualizadas |
| `errors` | Number | Cantidad de errores |

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| 200 | Operación exitosa (puede tener algunos errores individuales) |
| 400 | Error de validación o datos inválidos |

---

## Errores Comunes

### Error: Array vacío

**Request:**
```json
[]
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar al menos una zona"
  }
}
```

### Error: Formato inválido

**Request:**
```json
{
  "invalid": "data"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar un array de zonas o un objeto de zona"
  }
}
```

### Error: Nombre faltante en algún elemento

**Request:**
```json
[
  { "name": "Zona 1" },
  { "_id": "507f1f77bcf86cd799439011" },
  { "name": "Zona 3" }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El nombre de la zona en el índice 1 es requerido"
  }
}
```

### Error: Zona no existe (en edición)

Si intentas editar una zona que no existe, el resultado individual mostrará un error, pero el proceso continuará con los demás elementos:

**Request:**
```json
[
  { "_id": "507f1f77bcf86cd799439999", "name": "Zona Inexistente" },
  { "name": "Zona Nueva" }
]
```

**Response (200):**
```json
{
  "message": "Procesadas 2 zona(s): 1 creada(s), 0 actualizada(s), 1 error(es)",
  "results": [
    {
      "operation": "error",
      "error": "La zona no existe",
      "_id": "507f1f77bcf86cd799439999"
    },
    {
      "operation": "created",
      "zoneName": {
        "_id": "507f1f77bcf86cd799439014",
        "name": "Zona Nueva",
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "updated": 0,
    "errors": 1
  }
}
```

---

## Notas Importantes

1. **Operaciones en lote**: Todas las operaciones se procesan en una sola transacción usando `bulkWrite` de MongoDB, lo que es mucho más eficiente que hacer múltiples peticiones individuales.

2. **Orden preservado**: Los resultados se devuelven en el mismo orden que se enviaron los elementos en el request.

3. **Tolerancia a errores**: Si un elemento falla, el proceso continúa con los demás elementos. Revisa el campo `operation` en cada resultado para identificar errores.

4. **Compatibilidad**: El endpoint mantiene compatibilidad con el formato anterior (objeto único) para facilitar la migración.

5. **Validación**: Cada elemento se valida individualmente. Si algún elemento tiene un error de validación, toda la petición falla antes de procesar cualquier elemento.

---

## Ejemplos de Uso en JavaScript/TypeScript

### Crear múltiples zonas

```javascript
const createMultipleZones = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-zone', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify([
      { name: 'Zona Principal' },
      { name: 'Zona VIP' },
      { name: 'Zona General' }
    ])
  });
  
  const data = await response.json();
  console.log(`Creadas: ${data.summary.created}`);
  return data;
};
```

### Editar múltiples zonas

```javascript
const updateMultipleZones = async (zonesToUpdate) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-zone', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(zonesToUpdate)
  });
  
  const data = await response.json();
  
  // Filtrar solo las actualizadas exitosamente
  const updated = data.results.filter(r => r.operation === 'updated');
  return updated;
};
```

### Crear y editar en la misma petición

```javascript
const createAndUpdateZones = async (newZones, zonesToUpdate) => {
  const zones = [
    ...newZones.map(zone => ({ name: zone.name })),
    ...zonesToUpdate.map(zone => ({ _id: zone._id, name: zone.name }))
  ];
  
  const response = await fetch('/api/v1/seat-map/create-or-update-zone', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(zones)
  });
  
  const data = await response.json();
  
  // Manejar errores
  const errors = data.results.filter(r => r.operation === 'error');
  if (errors.length > 0) {
    console.error('Errores:', errors);
  }
  
  return data;
};
```

### Procesar resultados con manejo de errores

```javascript
const processZonesWithErrorHandling = async (zones) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-zone', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(zones)
  });
  
  const data = await response.json();
  
  // Separar resultados por tipo
  const created = data.results.filter(r => r.operation === 'created');
  const updated = data.results.filter(r => r.operation === 'updated');
  const errors = data.results.filter(r => r.operation === 'error');
  
  return {
    success: {
      created: created.map(r => r.zoneName),
      updated: updated.map(r => r.zoneName)
    },
    errors: errors.map(r => ({
      _id: r._id,
      error: r.error
    }))
  };
};
```

---

## Resumen de Validaciones

- ✅ Debe enviar al menos una zona
- ✅ Cada zona debe tener el campo `name`
- ✅ Si se envía `_id`, la zona debe existir (si no existe, se reporta como error pero no detiene el proceso)
- ✅ El formato puede ser array directo, objeto con propiedad `zones`, o objeto único

---

# API - Crear o Editar Secciones (Bulk)

## Endpoint

```
POST /api/v1/seat-map/create-or-update-section
```

## Descripción

Este endpoint permite **crear múltiples secciones** o **editar múltiples secciones** en una sola petición usando operaciones en lote (bulk). Puedes enviar un array de secciones y el sistema procesará todas las operaciones de manera eficiente, devolviendo información detallada de cada elemento procesado.

Las secciones pueden ser de dos tipos:
- **Por aforo**: Secciones con capacidad máxima (`isWithCapacity: true` y `maxCapacity` requerido)
- **Numerada**: Secciones con cantidad de asientos (`seatQuantity` requerido)

## Funcionamiento

- **Crear**: Si NO se envía el campo `_id` en un elemento, se crea una nueva sección
- **Editar**: Si se envía el campo `_id` en un elemento, se actualiza la sección existente con ese ID
- **Bulk**: Puedes mezclar creaciones y ediciones en la misma petición
- **Orden**: Los resultados se devuelven en el mismo orden que se enviaron

---

## Formatos de Request

El endpoint acepta múltiples formatos para facilitar su uso:

### Formato 1: Array directo (Recomendado)

```json
[
  {
    "sectionClass": { "code": "A", "name": "Sección A" },
    "svgUrl": "https://example.com/section-a.svg",
    "sectionFill": "#FF0000",
    "seatQuantity": 50,
    "gate": "Puerta 1"
  },
  {
    "_id": "507f1f77bcf86cd799439011",
    "sectionClass": { "code": "VIP", "name": "VIP" },
    "svgUrl": "https://example.com/vip.svg",
    "sectionFill": "#FFD700",
    "isWithCapacity": true,
    "maxCapacity": 200
  }
]
```

### Formato 2: Objeto con propiedad `sections`

```json
{
  "sections": [
    {
      "sectionClass": { "code": "A", "name": "Sección A" },
      "svgUrl": "https://example.com/section-a.svg",
      "sectionFill": "#FF0000",
      "seatQuantity": 50
    }
  ]
}
```

### Formato 3: Objeto único (Compatibilidad)

```json
{
  "sectionClass": { "code": "A", "name": "Sección A" },
  "svgUrl": "https://example.com/section-a.svg",
  "sectionFill": "#FF0000",
  "seatQuantity": 50,
  "gate": "Puerta 1"
}
```

---

## Campos

### Campos Requeridos (Todos)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `sectionClass` | Object | Clase de la sección (requerido) |
| `sectionClass.code` | String | Código de la sección (requerido) |
| `sectionClass.name` | String | Nombre de la sección (requerido) |
| `svgUrl` | String | URL del SVG de la sección (requerido) |
| `sectionFill` | String | Color de relleno de la sección (requerido) |

### Campos Requeridos Condicionales

#### Si es sección por aforo (`isWithCapacity: true`)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `maxCapacity` | Number | Capacidad máxima de la sección (requerido si `isWithCapacity: true`) |

#### Si es sección numerada (`isWithCapacity` no es `true`)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `seatQuantity` | Number | Cantidad de asientos de la sección (requerido si no es por aforo) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `_id` | String | ID de la sección a editar (opcional, solo si es edición) |
| `gate` | String | Puerta de acceso a la sección |
| `isWithCapacity` | Boolean | Indica si la sección es por aforo (true) o numerada (false/undefined) |
| `firstRow` | Number | Primera fila de la sección (default: 1) |
| `lastRow` | Number | Última fila de la sección (default: 1) |
| `disabled` | Boolean | Indica si la sección está deshabilitada |
| `parkingSection` | Boolean | Indica si es una sección de estacionamiento |

---

## Ejemplos de Request

### 1. Crear sección numerada

```json
{
  "sectionClass": { "code": "GEN", "name": "General" },
  "svgUrl": "https://example.com/gen.svg",
  "sectionFill": "#0000FF",
  "seatQuantity": 500,
  "gate": "Puerta Principal"
}
```

### 2. Crear sección por aforo

```json
{
  "sectionClass": { "code": "VIP", "name": "VIP" },
  "svgUrl": "https://example.com/vip.svg",
  "sectionFill": "#FFD700",
  "isWithCapacity": true,
  "maxCapacity": 200,
  "gate": "Puerta VIP"
}
```

### 3. Crear múltiples secciones (mezclando tipos)

```json
[
  {
    "sectionClass": { "code": "A", "name": "Sección A" },
    "svgUrl": "https://example.com/section-a.svg",
    "sectionFill": "#FF0000",
    "seatQuantity": 50,
    "gate": "Puerta 1"
  },
  {
    "sectionClass": { "code": "VIP", "name": "VIP" },
    "svgUrl": "https://example.com/vip.svg",
    "sectionFill": "#FFD700",
    "isWithCapacity": true,
    "maxCapacity": 200,
    "gate": "Puerta VIP"
  },
  {
    "sectionClass": { "code": "B", "name": "Sección B" },
    "svgUrl": "https://example.com/section-b.svg",
    "sectionFill": "#00FF00",
    "seatQuantity": 100
  }
]
```

### 4. Editar múltiples secciones

```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "sectionClass": { "code": "A", "name": "Sección A Actualizada" },
    "svgUrl": "https://example.com/section-a-v2.svg",
    "sectionFill": "#FF0000",
    "seatQuantity": 60
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "sectionClass": { "code": "VIP", "name": "VIP Actualizada" },
    "svgUrl": "https://example.com/vip-v2.svg",
    "sectionFill": "#FFD700",
    "isWithCapacity": true,
    "maxCapacity": 250
  }
]
```

### 5. Crear y editar en la misma petición

```json
[
  {
    "sectionClass": { "code": "C", "name": "Sección Nueva" },
    "svgUrl": "https://example.com/section-c.svg",
    "sectionFill": "#FFFF00",
    "seatQuantity": 75
  },
  {
    "_id": "507f1f77bcf86cd799439011",
    "sectionClass": { "code": "A", "name": "Sección A Actualizada" },
    "svgUrl": "https://example.com/section-a-v2.svg",
    "sectionFill": "#FF0000",
    "seatQuantity": 60
  }
]
```

---

## Ejemplo de Response (200 OK)

```json
{
  "message": "Procesadas 3 sección(es): 2 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "section": {
        "_id": "507f1f77bcf86cd799439013",
        "sectionClass": {
          "code": "A",
          "name": "Sección A"
        },
        "svgUrl": "https://example.com/section-a.svg",
        "sectionFill": "#FF0000",
        "seatQuantity": 50,
        "gate": "Puerta 1",
        "firstRow": 1,
        "lastRow": 1,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "created",
      "section": {
        "_id": "507f1f77bcf86cd799439014",
        "sectionClass": {
          "code": "VIP",
          "name": "VIP"
        },
        "svgUrl": "https://example.com/vip.svg",
        "sectionFill": "#FFD700",
        "isWithCapacity": true,
        "maxCapacity": 200,
        "gate": "Puerta VIP",
        "firstRow": 1,
        "lastRow": 1,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "updated",
      "section": {
        "_id": "507f1f77bcf86cd799439011",
        "sectionClass": {
          "code": "B",
          "name": "Sección B Actualizada"
        },
        "svgUrl": "https://example.com/section-b-v2.svg",
        "sectionFill": "#00FF00",
        "seatQuantity": 100,
        "firstRow": 1,
        "lastRow": 1,
        "createdAt": "2024-01-10T08:00:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 3,
    "created": 2,
    "updated": 1,
    "errors": 0
  }
}
```

---

## Estructura de Response

### Campos de Response

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `message` | String | Mensaje descriptivo del resultado |
| `results` | Array | Array de resultados, uno por cada elemento procesado |
| `summary` | Object | Resumen con contadores de operaciones |

### Estructura de cada elemento en `results`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `operation` | String | Tipo de operación: `"created"`, `"updated"`, o `"error"` |
| `section` | Object | Objeto de la sección (solo si la operación fue exitosa) |
| `error` | String | Mensaje de error (solo si `operation` es `"error"`) |
| `_id` | String | ID de la sección (solo si hay error) |

### Estructura de `summary`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `total` | Number | Total de elementos procesados |
| `created` | Number | Cantidad de secciones creadas |
| `updated` | Number | Cantidad de secciones actualizadas |
| `errors` | Number | Cantidad de errores |

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| 200 | Operación exitosa (puede tener algunos errores individuales) |
| 400 | Error de validación o datos inválidos |

---

## Errores Comunes

### Error: Array vacío

**Request:**
```json
[]
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar al menos una sección"
  }
}
```

### Error: Campos requeridos faltantes

**Request:**
```json
[
  {
    "sectionClass": { "code": "A", "name": "Sección A" },
    "svgUrl": "https://example.com/section-a.svg"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El sectionFill de la sección en el índice 0 es requerido"
  }
}
```

### Error: sectionClass incompleto

**Request:**
```json
[
  {
    "sectionClass": { "code": "A" },
    "svgUrl": "https://example.com/section-a.svg",
    "sectionFill": "#FF0000",
    "seatQuantity": 50
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El nombre del sectionClass de la sección en el índice 0 es requerido"
  }
}
```

### Error: Sección por aforo sin maxCapacity

**Request:**
```json
[
  {
    "sectionClass": { "code": "VIP", "name": "VIP" },
    "svgUrl": "https://example.com/vip.svg",
    "sectionFill": "#FFD700",
    "isWithCapacity": true
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "La sección en el índice 0 es por aforo (isWithCapacity: true), por lo tanto maxCapacity es requerido"
  }
}
```

### Error: Sección numerada sin seatQuantity

**Request:**
```json
[
  {
    "sectionClass": { "code": "A", "name": "Sección A" },
    "svgUrl": "https://example.com/section-a.svg",
    "sectionFill": "#FF0000"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "La sección en el índice 0 es numerada (isWithCapacity no es true), por lo tanto seatQuantity es requerido"
  }
}
```

### Error: Sección no existe (en edición)

Si intentas editar una sección que no existe, el resultado individual mostrará un error, pero el proceso continuará con los demás elementos:

**Request:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439999",
    "sectionClass": { "code": "A", "name": "Sección Inexistente" },
    "svgUrl": "https://example.com/section-a.svg",
    "sectionFill": "#FF0000",
    "seatQuantity": 50
  },
  {
    "sectionClass": { "code": "B", "name": "Sección Nueva" },
    "svgUrl": "https://example.com/section-b.svg",
    "sectionFill": "#00FF00",
    "seatQuantity": 100
  }
]
```

**Response (200):**
```json
{
  "message": "Procesadas 2 sección(es): 1 creada(s), 0 actualizada(s), 1 error(es)",
  "results": [
    {
      "operation": "error",
      "error": "La sección no existe",
      "_id": "507f1f77bcf86cd799439999"
    },
    {
      "operation": "created",
      "section": {
        "_id": "507f1f77bcf86cd799439014",
        "sectionClass": {
          "code": "B",
          "name": "Sección Nueva"
        },
        "svgUrl": "https://example.com/section-b.svg",
        "sectionFill": "#00FF00",
        "seatQuantity": 100,
        "firstRow": 1,
        "lastRow": 1,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "updated": 0,
    "errors": 1
  }
}
```

---

## Notas Importantes

1. **Operaciones en lote**: Todas las operaciones se procesan en una sola transacción usando `bulkWrite` de MongoDB, lo que es mucho más eficiente que hacer múltiples peticiones individuales.

2. **Orden preservado**: Los resultados se devuelven en el mismo orden que se enviaron los elementos en el request.

3. **Tolerancia a errores**: Si un elemento falla, el proceso continúa con los demás elementos. Revisa el campo `operation` en cada resultado para identificar errores.

4. **Compatibilidad**: El endpoint mantiene compatibilidad con el formato anterior (objeto único) para facilitar la migración.

5. **Validación**: Cada elemento se valida individualmente. Si algún elemento tiene un error de validación, toda la petición falla antes de procesar cualquier elemento.

6. **Tipos de sección**: 
   - **Por aforo**: Debe tener `isWithCapacity: true` y `maxCapacity` requerido
   - **Numerada**: No debe tener `isWithCapacity: true` (o debe ser `false`) y `seatQuantity` requerido

7. **Campos opcionales**: Los campos como `gate`, `firstRow`, `lastRow`, `disabled`, y `parkingSection` son opcionales y solo se actualizarán si se envían en el request.

---

## Ejemplos de Uso en JavaScript/TypeScript

### Crear sección numerada

```javascript
const createNumberedSection = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-section', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      sectionClass: { code: 'GEN', name: 'General' },
      svgUrl: 'https://example.com/gen.svg',
      sectionFill: '#0000FF',
      seatQuantity: 500,
      gate: 'Puerta Principal'
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear sección por aforo

```javascript
const createCapacitySection = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-section', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      sectionClass: { code: 'VIP', name: 'VIP' },
      svgUrl: 'https://example.com/vip.svg',
      sectionFill: '#FFD700',
      isWithCapacity: true,
      maxCapacity: 200,
      gate: 'Puerta VIP'
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear múltiples secciones

```javascript
const createMultipleSections = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-section', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify([
      {
        sectionClass: { code: 'A', name: 'Sección A' },
        svgUrl: 'https://example.com/section-a.svg',
        sectionFill: '#FF0000',
        seatQuantity: 50,
        gate: 'Puerta 1'
      },
      {
        sectionClass: { code: 'VIP', name: 'VIP' },
        svgUrl: 'https://example.com/vip.svg',
        sectionFill: '#FFD700',
        isWithCapacity: true,
        maxCapacity: 200,
        gate: 'Puerta VIP'
      }
    ])
  });
  
  const data = await response.json();
  console.log(`Creadas: ${data.summary.created}`);
  return data;
};
```

### Editar múltiples secciones

```javascript
const updateMultipleSections = async (sectionsToUpdate) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-section', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(sectionsToUpdate)
  });
  
  const data = await response.json();
  
  // Filtrar solo las actualizadas exitosamente
  const updated = data.results.filter(r => r.operation === 'updated');
  return updated;
};
```

### Procesar resultados con manejo de errores

```javascript
const processSectionsWithErrorHandling = async (sections) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-section', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(sections)
  });
  
  const data = await response.json();
  
  // Separar resultados por tipo
  const created = data.results.filter(r => r.operation === 'created');
  const updated = data.results.filter(r => r.operation === 'updated');
  const errors = data.results.filter(r => r.operation === 'error');
  
  return {
    success: {
      created: created.map(r => r.section),
      updated: updated.map(r => r.section)
    },
    errors: errors.map(r => ({
      _id: r._id,
      error: r.error
    }))
  };
};
```

---

## Resumen de Validaciones

### Campos Requeridos (Todos)
- ✅ `sectionClass` (con `code` y `name`)
- ✅ `svgUrl`
- ✅ `sectionFill`

### Campos Requeridos Condicionales
- ✅ Si `isWithCapacity === true`: `maxCapacity` es requerido
- ✅ Si `isWithCapacity !== true`: `seatQuantity` es requerido

### Otros
- ✅ Debe enviar al menos una sección
- ✅ Si se envía `_id`, la sección debe existir (si no existe, se reporta como error pero no detiene el proceso)
- ✅ El formato puede ser array directo, objeto con propiedad `sections`, o objeto único

---

# API - Crear o Editar Filas (Bulk)

## Endpoint

```
POST /api/v1/seat-map/create-or-update-row
```

## Descripción

Este endpoint permite **crear múltiples filas** o **editar múltiples filas** en una sola petición usando operaciones en lote (bulk). Puedes enviar un array de filas y el sistema procesará todas las operaciones de manera eficiente, devolviendo información detallada de cada elemento procesado.

Las filas pueden tener características especiales:
- **Mesa**: Si `isTable: true`, indica que la fila representa una mesa
- **Fila completa**: Si `isCompleteRow: true`, indica que la fila se vende completa (no por asientos individuales)

## Funcionamiento

- **Crear**: Si NO se envía el campo `_id` en un elemento, se crea una nueva fila
- **Editar**: Si se envía el campo `_id` en un elemento, se actualiza la fila existente con ese ID
- **Bulk**: Puedes mezclar creaciones y ediciones en la misma petición
- **Orden**: Los resultados se devuelven en el mismo orden que se enviaron

---

## Formatos de Request

El endpoint acepta múltiples formatos para facilitar su uso:

### Formato 1: Array directo (Recomendado)

```json
[
  {
    "rowNumber": "1",
    "isTable": false,
    "isCompleteRow": false
  },
  {
    "_id": "507f1f77bcf86cd799439011",
    "rowNumber": "M1",
    "isTable": true,
    "isCompleteRow": false
  }
]
```

### Formato 2: Objeto con propiedad `rows`

```json
{
  "rows": [
    {
      "rowNumber": "1",
      "isTable": false,
      "isCompleteRow": false
    }
  ]
}
```

### Formato 3: Objeto único (Compatibilidad)

```json
{
  "rowNumber": "1",
  "isTable": false,
  "isCompleteRow": false
}
```

---

## Campos

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `rowNumber` | String | Número o identificador de la fila (requerido) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `_id` | String | ID de la fila a editar (opcional, solo si es edición) |
| `isTable` | Boolean | Indica si la fila es una mesa (opcional) |
| `isCompleteRow` | Boolean | Indica si la fila se vende completa (opcional) |
| `disabled` | Boolean | Indica si la fila está deshabilitada (opcional) |

---

## Ejemplos de Request

### 1. Crear fila normal

```json
{
  "rowNumber": "1",
  "isTable": false,
  "isCompleteRow": false
}
```

### 2. Crear fila que es mesa

```json
{
  "rowNumber": "M1",
  "isTable": true,
  "isCompleteRow": false
}
```

### 3. Crear fila que se vende completa

```json
{
  "rowNumber": "VIP-1",
  "isTable": false,
  "isCompleteRow": true
}
```

### 4. Crear múltiples filas

```json
[
  {
    "rowNumber": "1",
    "isTable": false,
    "isCompleteRow": false
  },
  {
    "rowNumber": "M1",
    "isTable": true,
    "isCompleteRow": false
  },
  {
    "rowNumber": "VIP-1",
    "isTable": false,
    "isCompleteRow": true
  }
]
```

### 5. Editar múltiples filas

```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "rowNumber": "1",
    "isTable": false,
    "isCompleteRow": true
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "rowNumber": "M1",
    "isTable": true,
    "isCompleteRow": false
  }
]
```

### 6. Crear y editar en la misma petición

```json
[
  {
    "rowNumber": "2",
    "isTable": false,
    "isCompleteRow": false
  },
  {
    "_id": "507f1f77bcf86cd799439011",
    "rowNumber": "1",
    "isTable": false,
    "isCompleteRow": true
  }
]
```

---

## Ejemplo de Response (200 OK)

```json
{
  "message": "Procesadas 3 fila(s): 2 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "row": {
        "_id": "507f1f77bcf86cd799439013",
        "rowNumber": "1",
        "isTable": false,
        "isCompleteRow": false,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "created",
      "row": {
        "_id": "507f1f77bcf86cd799439014",
        "rowNumber": "M1",
        "isTable": true,
        "isCompleteRow": false,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "updated",
      "row": {
        "_id": "507f1f77bcf86cd799439011",
        "rowNumber": "VIP-1",
        "isTable": false,
        "isCompleteRow": true,
        "createdAt": "2024-01-10T08:00:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 3,
    "created": 2,
    "updated": 1,
    "errors": 0
  }
}
```

---

## Estructura de Response

### Campos de Response

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `message` | String | Mensaje descriptivo del resultado |
| `results` | Array | Array de resultados, uno por cada elemento procesado |
| `summary` | Object | Resumen con contadores de operaciones |

### Estructura de cada elemento en `results`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `operation` | String | Tipo de operación: `"created"`, `"updated"`, o `"error"` |
| `row` | Object | Objeto de la fila (solo si la operación fue exitosa) |
| `error` | String | Mensaje de error (solo si `operation` es `"error"`) |
| `_id` | String | ID de la fila (solo si hay error) |

### Estructura de `summary`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `total` | Number | Total de elementos procesados |
| `created` | Number | Cantidad de filas creadas |
| `updated` | Number | Cantidad de filas actualizadas |
| `errors` | Number | Cantidad de errores |

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| 200 | Operación exitosa (puede tener algunos errores individuales) |
| 400 | Error de validación o datos inválidos |

---

## Errores Comunes

### Error: Array vacío

**Request:**
```json
[]
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar al menos una fila"
  }
}
```

### Error: Formato inválido

**Request:**
```json
{
  "invalid": "data"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar un array de filas o un objeto de fila"
  }
}
```

### Error: rowNumber faltante

**Request:**
```json
[
  {
    "isTable": false,
    "isCompleteRow": false
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El rowNumber de la fila en el índice 0 es requerido"
  }
}
```

### Error: Fila no existe (en edición)

Si intentas editar una fila que no existe, el resultado individual mostrará un error, pero el proceso continuará con los demás elementos:

**Request:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439999",
    "rowNumber": "1",
    "isTable": false,
    "isCompleteRow": false
  },
  {
    "rowNumber": "2",
    "isTable": false,
    "isCompleteRow": false
  }
]
```

**Response (200):**
```json
{
  "message": "Procesadas 2 fila(s): 1 creada(s), 0 actualizada(s), 1 error(es)",
  "results": [
    {
      "operation": "error",
      "error": "La fila no existe",
      "_id": "507f1f77bcf86cd799439999"
    },
    {
      "operation": "created",
      "row": {
        "_id": "507f1f77bcf86cd799439014",
        "rowNumber": "2",
        "isTable": false,
        "isCompleteRow": false,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "updated": 0,
    "errors": 1
  }
}
```

---

## Notas Importantes

1. **Operaciones en lote**: Todas las operaciones se procesan en una sola transacción usando `bulkWrite` de MongoDB, lo que es mucho más eficiente que hacer múltiples peticiones individuales.

2. **Orden preservado**: Los resultados se devuelven en el mismo orden que se enviaron los elementos en el request.

3. **Tolerancia a errores**: Si un elemento falla, el proceso continúa con los demás elementos. Revisa el campo `operation` en cada resultado para identificar errores.

4. **Compatibilidad**: El endpoint mantiene compatibilidad con el formato anterior (objeto único) para facilitar la migración.

5. **Validación**: Cada elemento se valida individualmente. Si algún elemento tiene un error de validación, toda la petición falla antes de procesar cualquier elemento.

6. **Campos booleanos**: Los campos `isTable` e `isCompleteRow` son opcionales. Si no se envían, se guardarán como `undefined` o `false` según el comportamiento del modelo.

7. **rowNumber**: Puede ser cualquier string (números, letras, combinaciones). Ejemplos: "1", "A", "VIP-1", "M1", etc.

---

## Ejemplos de Uso en JavaScript/TypeScript

### Crear fila normal

```javascript
const createRow = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-row', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      rowNumber: '1',
      isTable: false,
      isCompleteRow: false
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear fila que es mesa

```javascript
const createTableRow = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-row', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      rowNumber: 'M1',
      isTable: true,
      isCompleteRow: false
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear fila que se vende completa

```javascript
const createCompleteRow = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-row', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      rowNumber: 'VIP-1',
      isTable: false,
      isCompleteRow: true
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear múltiples filas

```javascript
const createMultipleRows = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-row', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify([
      {
        rowNumber: '1',
        isTable: false,
        isCompleteRow: false
      },
      {
        rowNumber: 'M1',
        isTable: true,
        isCompleteRow: false
      },
      {
        rowNumber: 'VIP-1',
        isTable: false,
        isCompleteRow: true
      }
    ])
  });
  
  const data = await response.json();
  console.log(`Creadas: ${data.summary.created}`);
  return data;
};
```

### Editar múltiples filas

```javascript
const updateMultipleRows = async (rowsToUpdate) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-row', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(rowsToUpdate)
  });
  
  const data = await response.json();
  
  // Filtrar solo las actualizadas exitosamente
  const updated = data.results.filter(r => r.operation === 'updated');
  return updated;
};
```

### Procesar resultados con manejo de errores

```javascript
const processRowsWithErrorHandling = async (rows) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-row', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(rows)
  });
  
  const data = await response.json();
  
  // Separar resultados por tipo
  const created = data.results.filter(r => r.operation === 'created');
  const updated = data.results.filter(r => r.operation === 'updated');
  const errors = data.results.filter(r => r.operation === 'error');
  
  return {
    success: {
      created: created.map(r => r.row),
      updated: updated.map(r => r.row)
    },
    errors: errors.map(r => ({
      _id: r._id,
      error: r.error
    }))
  };
};
```

---

## Resumen de Validaciones

- ✅ Debe enviar al menos una fila
- ✅ Cada fila debe tener el campo `rowNumber`
- ✅ Si se envía `_id`, la fila debe existir (si no existe, se reporta como error pero no detiene el proceso)
- ✅ El formato puede ser array directo, objeto con propiedad `rows`, o objeto único
- ✅ `isTable` e `isCompleteRow` son opcionales (booleanos)

---

# API - Crear o Editar Asientos (Bulk)

## Endpoint

```
POST /api/v1/seat-map/create-or-update-seat
```

## Descripción

Este endpoint permite **crear múltiples asientos** o **editar múltiples asientos** en una sola petición usando operaciones en lote (bulk). Puedes enviar un array de asientos y el sistema procesará todas las operaciones de manera eficiente, devolviendo información detallada de cada elemento procesado.

Cada asiento debe tener información de posición SVG (`svgInfo`) que define su ubicación y forma en el mapa.

## Funcionamiento

- **Crear**: Si NO se envía el campo `_id` en un elemento, se crea un nuevo asiento
- **Editar**: Si se envía el campo `_id` en un elemento, se actualiza el asiento existente con ese ID
- **Bulk**: Puedes mezclar creaciones y ediciones en la misma petición
- **Orden**: Los resultados se devuelven en el mismo orden que se enviaron

---

## Formatos de Request

El endpoint acepta múltiples formatos para facilitar su uso:

### Formato 1: Array directo (Recomendado)

```json
[
  {
    "seatNumber": "1",
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    }
  },
  {
    "_id": "507f1f77bcf86cd799439011",
    "seatNumber": "2",
    "svgInfo": {
      "svgX": "1400.5",
      "svgY": "2300.10",
      "svgR": "5.92837",
      "svgTag": "circle"
    },
    "column": "B"
  }
]
```

### Formato 2: Objeto con propiedad `seats`

```json
{
  "seats": [
    {
      "seatNumber": "1",
      "svgInfo": {
        "svgX": "1359.2",
        "svgY": "2278.74",
        "svgR": "5.92837",
        "svgTag": "circle"
      }
    }
  ]
}
```

### Formato 3: Objeto único (Compatibilidad)

```json
{
  "seatNumber": "1",
  "svgInfo": {
    "svgX": "1359.2",
    "svgY": "2278.74",
    "svgR": "5.92837",
    "svgTag": "circle"
  },
  "column": "A"
}
```

---

## Campos

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `seatNumber` | String | Número o identificador del asiento (requerido) |
| `svgInfo` | Object | Información de posición SVG del asiento (requerido) |
| `svgInfo.svgX` | String/Number | Coordenada X en el SVG (requerido) |
| `svgInfo.svgY` | String/Number | Coordenada Y en el SVG (requerido) |
| `svgInfo.svgR` | String/Number | Radio del elemento SVG (requerido) |
| `svgInfo.svgTag` | String | Tipo de elemento SVG (ej: "circle", "rect") (requerido) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `_id` | String | ID del asiento a editar (opcional, solo si es edición) |
| `column` | String | Columna del asiento (opcional, default: "A") |
| `isAvailableSeat` | Boolean | Indica si el asiento está disponible (opcional) |

---

## Ejemplos de Request

### 1. Crear un asiento

```json
{
  "seatNumber": "1",
  "svgInfo": {
    "svgX": "1359.2",
    "svgY": "2278.74",
    "svgR": "5.92837",
    "svgTag": "circle"
  },
  "column": "A",
  "isAvailableSeat": true
}
```

### 2. Crear múltiples asientos

```json
[
  {
    "seatNumber": "1",
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    },
    "column": "A"
  },
  {
    "seatNumber": "2",
    "svgInfo": {
      "svgX": "1400.5",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    },
    "column": "A"
  },
  {
    "seatNumber": "3",
    "svgInfo": {
      "svgX": "1441.8",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    },
    "column": "A"
  }
]
```

### 3. Editar múltiples asientos

```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "seatNumber": "1",
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74",
      "svgR": "6.0",
      "svgTag": "circle"
    },
    "column": "A",
    "isAvailableSeat": false
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "seatNumber": "2",
    "svgInfo": {
      "svgX": "1400.5",
      "svgY": "2300.10",
      "svgR": "5.92837",
      "svgTag": "circle"
    },
    "column": "B"
  }
]
```

### 4. Crear y editar en la misma petición

```json
[
  {
    "seatNumber": "10",
    "svgInfo": {
      "svgX": "1500.0",
      "svgY": "2400.0",
      "svgR": "5.92837",
      "svgTag": "circle"
    },
    "column": "C"
  },
  {
    "_id": "507f1f77bcf86cd799439011",
    "seatNumber": "1",
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74",
      "svgR": "6.0",
      "svgTag": "circle"
    },
    "isAvailableSeat": false
  }
]
```

---

## Ejemplo de Response (200 OK)

```json
{
  "message": "Procesados 3 asiento(s): 2 creado(s), 1 actualizado(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "seat": {
        "_id": "507f1f77bcf86cd799439013",
        "seatNumber": "1",
        "column": "A",
        "svgInfo": {
          "svgX": "1359.2",
          "svgY": "2278.74",
          "svgR": "5.92837",
          "svgTag": "circle"
        },
        "isAvailableSeat": true,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "created",
      "seat": {
        "_id": "507f1f77bcf86cd799439014",
        "seatNumber": "2",
        "column": "A",
        "svgInfo": {
          "svgX": "1400.5",
          "svgY": "2278.74",
          "svgR": "5.92837",
          "svgTag": "circle"
        },
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "updated",
      "seat": {
        "_id": "507f1f77bcf86cd799439011",
        "seatNumber": "3",
        "column": "B",
        "svgInfo": {
          "svgX": "1441.8",
          "svgY": "2278.74",
          "svgR": "6.0",
          "svgTag": "circle"
        },
        "isAvailableSeat": false,
        "createdAt": "2024-01-10T08:00:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 3,
    "created": 2,
    "updated": 1,
    "errors": 0
  }
}
```

---

## Estructura de Response

### Campos de Response

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `message` | String | Mensaje descriptivo del resultado |
| `results` | Array | Array de resultados, uno por cada elemento procesado |
| `summary` | Object | Resumen con contadores de operaciones |

### Estructura de cada elemento en `results`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `operation` | String | Tipo de operación: `"created"`, `"updated"`, o `"error"` |
| `seat` | Object | Objeto del asiento (solo si la operación fue exitosa) |
| `error` | String | Mensaje de error (solo si `operation` es `"error"`) |
| `_id` | String | ID del asiento (solo si hay error) |

### Estructura de `summary`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `total` | Number | Total de elementos procesados |
| `created` | Number | Cantidad de asientos creados |
| `updated` | Number | Cantidad de asientos actualizados |
| `errors` | Number | Cantidad de errores |

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| 200 | Operación exitosa (puede tener algunos errores individuales) |
| 400 | Error de validación o datos inválidos |

---

## Errores Comunes

### Error: Array vacío

**Request:**
```json
[]
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar al menos un asiento"
  }
}
```

### Error: Formato inválido

**Request:**
```json
{
  "invalid": "data"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar un array de asientos o un objeto de asiento"
  }
}
```

### Error: seatNumber faltante

**Request:**
```json
[
  {
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    }
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El seatNumber del asiento en el índice 0 es requerido"
  }
}
```

### Error: svgInfo faltante

**Request:**
```json
[
  {
    "seatNumber": "1"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El svgInfo del asiento en el índice 0 es requerido"
  }
}
```

### Error: svgInfo incompleto

**Request:**
```json
[
  {
    "seatNumber": "1",
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74"
    }
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El svgInfo.svgR del asiento en el índice 0 es requerido"
  }
}
```

### Error: svgInfo no es un objeto

**Request:**
```json
[
  {
    "seatNumber": "1",
    "svgInfo": "invalid"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El svgInfo del asiento en el índice 0 debe ser un objeto"
  }
}
```

### Error: Asiento no existe (en edición)

Si intentas editar un asiento que no existe, el resultado individual mostrará un error, pero el proceso continuará con los demás elementos:

**Request:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439999",
    "seatNumber": "1",
    "svgInfo": {
      "svgX": "1359.2",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    }
  },
  {
    "seatNumber": "2",
    "svgInfo": {
      "svgX": "1400.5",
      "svgY": "2278.74",
      "svgR": "5.92837",
      "svgTag": "circle"
    }
  }
]
```

**Response (200):**
```json
{
  "message": "Procesados 2 asiento(s): 1 creado(s), 0 actualizado(s), 1 error(es)",
  "results": [
    {
      "operation": "error",
      "error": "El asiento no existe",
      "_id": "507f1f77bcf86cd799439999"
    },
    {
      "operation": "created",
      "seat": {
        "_id": "507f1f77bcf86cd799439014",
        "seatNumber": "2",
        "column": "A",
        "svgInfo": {
          "svgX": "1400.5",
          "svgY": "2278.74",
          "svgR": "5.92837",
          "svgTag": "circle"
        },
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "updated": 0,
    "errors": 1
  }
}
```

---

## Notas Importantes

1. **Operaciones en lote**: Todas las operaciones se procesan en una sola transacción usando `bulkWrite` de MongoDB, lo que es mucho más eficiente que hacer múltiples peticiones individuales.

2. **Orden preservado**: Los resultados se devuelven en el mismo orden que se enviaron los elementos en el request.

3. **Tolerancia a errores**: Si un elemento falla, el proceso continúa con los demás elementos. Revisa el campo `operation` en cada resultado para identificar errores.

4. **Compatibilidad**: El endpoint mantiene compatibilidad con el formato anterior (objeto único) para facilitar la migración.

5. **Validación**: Cada elemento se valida individualmente. Si algún elemento tiene un error de validación, toda la petición falla antes de procesar cualquier elemento.

6. **svgInfo**: Este objeto es crítico para la visualización del asiento en el mapa SVG. Todos sus campos son requeridos:
   - `svgX`: Coordenada X (puede ser string o number)
   - `svgY`: Coordenada Y (puede ser string o number)
   - `svgR`: Radio del elemento (puede ser string o number)
   - `svgTag`: Tipo de elemento SVG (ej: "circle", "rect", "ellipse")

7. **column**: Si no se envía, el valor por defecto es "A".

---

## Ejemplos de Uso en JavaScript/TypeScript

### Crear un asiento

```javascript
const createSeat = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-seat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      seatNumber: '1',
      svgInfo: {
        svgX: '1359.2',
        svgY: '2278.74',
        svgR: '5.92837',
        svgTag: 'circle'
      },
      column: 'A',
      isAvailableSeat: true
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear múltiples asientos

```javascript
const createMultipleSeats = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-seat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify([
      {
        seatNumber: '1',
        svgInfo: {
          svgX: '1359.2',
          svgY: '2278.74',
          svgR: '5.92837',
          svgTag: 'circle'
        },
        column: 'A'
      },
      {
        seatNumber: '2',
        svgInfo: {
          svgX: '1400.5',
          svgY: '2278.74',
          svgR: '5.92837',
          svgTag: 'circle'
        },
        column: 'A'
      },
      {
        seatNumber: '3',
        svgInfo: {
          svgX: '1441.8',
          svgY: '2278.74',
          svgR: '5.92837',
          svgTag: 'circle'
        },
        column: 'A'
      }
    ])
  });
  
  const data = await response.json();
  console.log(`Creados: ${data.summary.created}`);
  return data;
};
```

### Editar múltiples asientos

```javascript
const updateMultipleSeats = async (seatsToUpdate) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-seat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(seatsToUpdate)
  });
  
  const data = await response.json();
  
  // Filtrar solo los actualizados exitosamente
  const updated = data.results.filter(r => r.operation === 'updated');
  return updated;
};
```

### Crear asientos desde coordenadas

```javascript
const createSeatsFromCoordinates = async (coordinates) => {
  // coordinates es un array de { seatNumber, x, y, r, tag, column }
  const seats = coordinates.map(coord => ({
    seatNumber: coord.seatNumber,
    svgInfo: {
      svgX: coord.x.toString(),
      svgY: coord.y.toString(),
      svgR: coord.r.toString(),
      svgTag: coord.tag || 'circle'
    },
    column: coord.column || 'A',
    isAvailableSeat: coord.isAvailableSeat
  }));
  
  const response = await fetch('/api/v1/seat-map/create-or-update-seat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(seats)
  });
  
  const data = await response.json();
  return data;
};
```

### Procesar resultados con manejo de errores

```javascript
const processSeatsWithErrorHandling = async (seats) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-seat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(seats)
  });
  
  const data = await response.json();
  
  // Separar resultados por tipo
  const created = data.results.filter(r => r.operation === 'created');
  const updated = data.results.filter(r => r.operation === 'updated');
  const errors = data.results.filter(r => r.operation === 'error');
  
  return {
    success: {
      created: created.map(r => r.seat),
      updated: updated.map(r => r.seat)
    },
    errors: errors.map(r => ({
      _id: r._id,
      error: r.error
    }))
  };
};
```

---

## Resumen de Validaciones

### Campos Requeridos
- ✅ `seatNumber` es requerido
- ✅ `svgInfo` es requerido (debe ser un objeto)
- ✅ `svgInfo.svgX` es requerido
- ✅ `svgInfo.svgY` es requerido
- ✅ `svgInfo.svgR` es requerido
- ✅ `svgInfo.svgTag` es requerido

### Otros
- ✅ Debe enviar al menos un asiento
- ✅ Si se envía `_id`, el asiento debe existir (si no existe, se reporta como error pero no detiene el proceso)
- ✅ El formato puede ser array directo, objeto con propiedad `seats`, o objeto único
- ✅ `column` es opcional (default: "A")
- ✅ `isAvailableSeat` es opcional

---

# API - Crear o Editar Locations (Bulk)

## Endpoint

```
POST /api/v1/seat-map/create-or-update-location
```

## Descripción

Este endpoint permite **crear múltiples locations** o **editar múltiples locations** en una sola petición usando operaciones en lote (bulk). Puedes enviar un array de locations y el sistema procesará todas las operaciones de manera eficiente, devolviendo información detallada de cada elemento procesado.

**Características especiales:**
- **Secciones por aforo**: Si una sección es por aforo (`isWithCapacity: true`), el sistema automáticamente asigna valores por defecto para `row` y `seat`, ya que estas secciones no tienen filas ni asientos individuales.
- **Generación automática de locationCode**: Si no se envía `locationCode`, se genera automáticamente usando `generateLocationCode()`.

## Funcionamiento

- **Crear**: Si NO se envía el campo `_id` en un elemento, se crea una nueva location
- **Editar**: Si se envía el campo `_id` en un elemento, se actualiza la location existente con ese ID
- **Bulk**: Puedes mezclar creaciones y ediciones en la misma petición
- **Orden**: Los resultados se devuelven en el mismo orden que se enviaron
- **Detección automática de secciones por aforo**: Si no se envía `isWithCapacity`, el sistema verifica la sección en la base de datos para determinar si es por aforo

---

## Formatos de Request

El endpoint acepta múltiples formatos para facilitar su uso:

### Formato 1: Array directo (Recomendado)

```json
[
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439015"
  },
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439016",
    "isWithCapacity": true
  }
]
```

### Formato 2: Objeto con propiedad `locations`

```json
{
  "locations": [
    {
      "seatMap": "507f1f77bcf86cd799439011",
      "zone": "507f1f77bcf86cd799439012",
      "section": "507f1f77bcf86cd799439013",
      "row": "507f1f77bcf86cd799439014",
      "seat": "507f1f77bcf86cd799439015"
    }
  ]
}
```

### Formato 3: Objeto único (Compatibilidad)

```json
{
  "seatMap": "507f1f77bcf86cd799439011",
  "zone": "507f1f77bcf86cd799439012",
  "section": "507f1f77bcf86cd799439013",
  "row": "507f1f77bcf86cd799439014",
  "seat": "507f1f77bcf86cd799439015"
}
```

---

## Campos

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `seatMap` | String (ObjectId) | ID del mapa de asientos (requerido) |
| `zone` | String (ObjectId) | ID de la zona (requerido) |
| `section` | String (ObjectId) | ID de la sección (requerido) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `_id` | String (ObjectId) | ID de la location a editar (opcional, solo si es edición) |
| `plan` | String (ObjectId) | ID del plan (opcional) |
| `row` | String (ObjectId) | ID de la fila (opcional, requerido para secciones numeradas) |
| `seat` | String (ObjectId) | ID del asiento (opcional, requerido para secciones numeradas) |
| `table` | String (ObjectId) | ID de la mesa (opcional) |
| `chair` | String (ObjectId) | ID de la silla (opcional) |
| `locationCode` | String | Código de la location (opcional, se genera automáticamente si no se envía) |
| `disabled` | Boolean | Indica si la location está deshabilitada (opcional) |
| `isWithCapacity` | Boolean | Indica si la sección es por aforo (opcional, se detecta automáticamente si no se envía) |

### Notas Importantes sobre Campos

- **Secciones por aforo**: Si `isWithCapacity: true` o si la sección es por aforo, los campos `row` y `seat` se asignan automáticamente con valores por defecto:
  - `row`: `"64bfceb738589419f3cb5ba4"`
  - `seat`: `"64bfcdce38589419f3cb5b9f"`
- **Secciones numeradas**: Si la sección NO es por aforo, los campos `row` y `seat` son requeridos.
- **locationCode**: Si no se envía, se genera automáticamente usando `generateLocationCode()`.

---

## Ejemplos de Request

### 1. Crear location para sección numerada

```json
{
  "seatMap": "507f1f77bcf86cd799439011",
  "zone": "507f1f77bcf86cd799439012",
  "section": "507f1f77bcf86cd799439013",
  "row": "507f1f77bcf86cd799439014",
  "seat": "507f1f77bcf86cd799439015",
  "plan": "507f1f77bcf86cd799439016"
}
```

### 2. Crear location para sección por aforo (con flag explícito)

```json
{
  "seatMap": "507f1f77bcf86cd799439011",
  "zone": "507f1f77bcf86cd799439012",
  "section": "507f1f77bcf86cd799439013",
  "isWithCapacity": true,
  "plan": "507f1f77bcf86cd799439016"
}
```

### 3. Crear location para sección por aforo (detección automática)

```json
{
  "seatMap": "507f1f77bcf86cd799439011",
  "zone": "507f1f77bcf86cd799439012",
  "section": "507f1f77bcf86cd799439013",
  "plan": "507f1f77bcf86cd799439016"
}
```

El sistema verificará automáticamente si la sección es por aforo y asignará los valores por defecto para `row` y `seat`.

### 4. Crear múltiples locations (mezclando secciones numeradas y por aforo)

```json
[
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439015"
  },
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439016",
    "isWithCapacity": true
  },
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439017",
    "row": "507f1f77bcf86cd799439018",
    "seat": "507f1f77bcf86cd799439019",
    "locationCode": "CUSTOM123"
  }
]
```

### 5. Editar múltiples locations

```json
[
  {
    "_id": "507f1f77bcf86cd799439020",
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439015",
    "disabled": true
  },
  {
    "_id": "507f1f77bcf86cd799439021",
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439016",
    "isWithCapacity": true,
    "locationCode": "UPDATED456"
  }
]
```

### 6. Crear y editar en la misma petición

```json
[
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439015"
  },
  {
    "_id": "507f1f77bcf86cd799439020",
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439015",
    "disabled": true
  }
]
```

---

## Ejemplo de Response (200 OK)

```json
{
  "message": "Procesadas 3 location(es): 2 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "location": {
        "_id": "507f1f77bcf86cd799439022",
        "seatMap": "507f1f77bcf86cd799439011",
        "zone": "507f1f77bcf86cd799439012",
        "section": "507f1f77bcf86cd799439013",
        "row": "507f1f77bcf86cd799439014",
        "seat": "507f1f77bcf86cd799439015",
        "locationCode": "ABC123XYZ",
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "created",
      "location": {
        "_id": "507f1f77bcf86cd799439023",
        "seatMap": "507f1f77bcf86cd799439011",
        "zone": "507f1f77bcf86cd799439012",
        "section": "507f1f77bcf86cd799439016",
        "row": "64bfceb738589419f3cb5ba4",
        "seat": "64bfcdce38589419f3cb5b9f",
        "locationCode": "DEF456UVW",
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "updated",
      "location": {
        "_id": "507f1f77bcf86cd799439020",
        "seatMap": "507f1f77bcf86cd799439011",
        "zone": "507f1f77bcf86cd799439012",
        "section": "507f1f77bcf86cd799439013",
        "row": "507f1f77bcf86cd799439014",
        "seat": "507f1f77bcf86cd799439015",
        "locationCode": "UPDATED456",
        "disabled": true,
        "createdAt": "2024-01-10T08:00:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 3,
    "created": 2,
    "updated": 1,
    "errors": 0
  }
}
```

---

## Estructura de Response

### Campos de Response

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `message` | String | Mensaje descriptivo del resultado |
| `results` | Array | Array de resultados, uno por cada elemento procesado |
| `summary` | Object | Resumen con contadores de operaciones |

### Estructura de cada elemento en `results`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `operation` | String | Tipo de operación: `"created"`, `"updated"`, o `"error"` |
| `location` | Object | Objeto de la location (solo si la operación fue exitosa) |
| `error` | String | Mensaje de error (solo si `operation` es `"error"`) |
| `_id` | String | ID de la location (solo si hay error) |

### Estructura de `summary`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `total` | Number | Total de elementos procesados |
| `created` | Number | Cantidad de locations creadas |
| `updated` | Number | Cantidad de locations actualizadas |
| `errors` | Number | Cantidad de errores |

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| 200 | Operación exitosa (puede tener algunos errores individuales) |
| 400 | Error de validación o datos inválidos |

---

## Errores Comunes

### Error: Array vacío

**Request:**
```json
[]
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar al menos una location"
  }
}
```

### Error: Formato inválido

**Request:**
```json
{
  "invalid": "data"
}
```

**Response (400):**
```json
{
  "error": {
    "message": "Debe enviar un array de locations o un objeto de location"
  }
}
```

### Error: seatMap faltante

**Request:**
```json
[
  {
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El seatMap de la location en el índice 0 es requerido"
  }
}
```

### Error: zone faltante

**Request:**
```json
[
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "section": "507f1f77bcf86cd799439013"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El zone de la location en el índice 0 es requerido"
  }
}
```

### Error: section faltante

**Request:**
```json
[
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012"
  }
]
```

**Response (400):**
```json
{
  "error": {
    "message": "El section de la location en el índice 0 es requerido"
  }
}
```

### Error: Location no existe (en edición)

Si intentas editar una location que no existe, el resultado individual mostrará un error, pero el proceso continuará con los demás elementos:

**Request:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439999",
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439015"
  },
  {
    "seatMap": "507f1f77bcf86cd799439011",
    "zone": "507f1f77bcf86cd799439012",
    "section": "507f1f77bcf86cd799439013",
    "row": "507f1f77bcf86cd799439014",
    "seat": "507f1f77bcf86cd799439016"
  }
]
```

**Response (200):**
```json
{
  "message": "Procesadas 2 location(es): 1 creada(s), 0 actualizada(s), 1 error(es)",
  "results": [
    {
      "operation": "error",
      "error": "La location no existe",
      "_id": "507f1f77bcf86cd799439999"
    },
    {
      "operation": "created",
      "location": {
        "_id": "507f1f77bcf86cd799439024",
        "seatMap": "507f1f77bcf86cd799439011",
        "zone": "507f1f77bcf86cd799439012",
        "section": "507f1f77bcf86cd799439013",
        "row": "507f1f77bcf86cd799439014",
        "seat": "507f1f77bcf86cd799439016",
        "locationCode": "GHI789RST",
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "updated": 0,
    "errors": 1
  }
}
```

---

## Notas Importantes

1. **Operaciones en lote**: Todas las operaciones se procesan en una sola transacción usando `bulkWrite` de MongoDB, lo que es mucho más eficiente que hacer múltiples peticiones individuales.

2. **Orden preservado**: Los resultados se devuelven en el mismo orden que se enviaron los elementos en el request.

3. **Tolerancia a errores**: Si un elemento falla, el proceso continúa con los demás elementos. Revisa el campo `operation` en cada resultado para identificar errores.

4. **Compatibilidad**: El endpoint mantiene compatibilidad con el formato anterior (objeto único) para facilitar la migración.

5. **Validación**: Cada elemento se valida individualmente. Si algún elemento tiene un error de validación, toda la petición falla antes de procesar cualquier elemento.

6. **Secciones por aforo**: 
   - Si `isWithCapacity: true` o si la sección es por aforo, los campos `row` y `seat` se asignan automáticamente con valores por defecto.
   - Si no se envía `isWithCapacity`, el sistema verifica la sección en la base de datos para determinar si es por aforo.
   - Los valores por defecto son:
     - `row`: `"64bfceb738589419f3cb5ba4"`
     - `seat`: `"64bfcdce38589419f3cb5b9f"`

7. **locationCode**: Si no se envía, se genera automáticamente usando `generateLocationCode()`. Puedes enviar un `locationCode` personalizado si lo necesitas.

8. **Secciones numeradas**: Para secciones que NO son por aforo, los campos `row` y `seat` son requeridos (aunque no se validan explícitamente, deben enviarse para que la location tenga sentido).

---

## Ejemplos de Uso en JavaScript/TypeScript

### Crear location para sección numerada

```javascript
const createLocation = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-location', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      seatMap: '507f1f77bcf86cd799439011',
      zone: '507f1f77bcf86cd799439012',
      section: '507f1f77bcf86cd799439013',
      row: '507f1f77bcf86cd799439014',
      seat: '507f1f77bcf86cd799439015',
      plan: '507f1f77bcf86cd799439016'
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear location para sección por aforo

```javascript
const createLocationWithCapacity = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-location', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      seatMap: '507f1f77bcf86cd799439011',
      zone: '507f1f77bcf86cd799439012',
      section: '507f1f77bcf86cd799439013',
      isWithCapacity: true,
      plan: '507f1f77bcf86cd799439016'
    })
  });
  
  const data = await response.json();
  return data;
};
```

### Crear múltiples locations

```javascript
const createMultipleLocations = async () => {
  const response = await fetch('/api/v1/seat-map/create-or-update-location', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify([
      {
        seatMap: '507f1f77bcf86cd799439011',
        zone: '507f1f77bcf86cd799439012',
        section: '507f1f77bcf86cd799439013',
        row: '507f1f77bcf86cd799439014',
        seat: '507f1f77bcf86cd799439015'
      },
      {
        seatMap: '507f1f77bcf86cd799439011',
        zone: '507f1f77bcf86cd799439012',
        section: '507f1f77bcf86cd799439016',
        isWithCapacity: true
      }
    ])
  });
  
  const data = await response.json();
  console.log(`Creadas: ${data.summary.created}`);
  return data;
};
```

### Crear locations para sección por aforo (múltiples)

```javascript
const createLocationsForCapacitySection = async (seatMap, zone, section, quantity) => {
  const locations = [];
  
  for (let i = 0; i < quantity; i++) {
    locations.push({
      seatMap,
      zone,
      section,
      isWithCapacity: true
    });
  }
  
  const response = await fetch('/api/v1/seat-map/create-or-update-location', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(locations)
  });
  
  const data = await response.json();
  return data;
};
```

### Editar múltiples locations

```javascript
const updateMultipleLocations = async (locationsToUpdate) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-location', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(locationsToUpdate)
  });
  
  const data = await response.json();
  
  // Filtrar solo las actualizadas exitosamente
  const updated = data.results.filter(r => r.operation === 'updated');
  return updated;
};
```

### Procesar resultados con manejo de errores

```javascript
const processLocationsWithErrorHandling = async (locations) => {
  const response = await fetch('/api/v1/seat-map/create-or-update-location', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(locations)
  });
  
  const data = await response.json();
  
  // Separar resultados por tipo
  const created = data.results.filter(r => r.operation === 'created');
  const updated = data.results.filter(r => r.operation === 'updated');
  const errors = data.results.filter(r => r.operation === 'error');
  
  return {
    success: {
      created: created.map(r => r.location),
      updated: updated.map(r => r.location)
    },
    errors: errors.map(r => ({
      _id: r._id,
      error: r.error
    }))
  };
};
```

### Crear locations desde un mapa de asientos completo

```javascript
const createLocationsFromSeatMap = async (seatMapId, zones) => {
  const allLocations = [];
  
  for (const zone of zones) {
    for (const section of zone.sections) {
      if (section.isWithCapacity) {
        // Sección por aforo: crear locations sin row/seat
        for (let i = 0; i < section.maxCapacity; i++) {
          allLocations.push({
            seatMap: seatMapId,
            zone: zone._id,
            section: section._id,
            isWithCapacity: true
          });
        }
      } else {
        // Sección numerada: crear locations con row/seat
        for (const row of section.rows) {
          for (const seat of row.seats) {
            allLocations.push({
              seatMap: seatMapId,
              zone: zone._id,
              section: section._id,
              row: row._id,
              seat: seat._id
            });
          }
        }
      }
    }
  }
  
  // Procesar en lotes para evitar peticiones muy grandes
  const batchSize = 100;
  const results = [];
  
  for (let i = 0; i < allLocations.length; i += batchSize) {
    const batch = allLocations.slice(i, i + batchSize);
    const response = await fetch('/api/v1/seat-map/create-or-update-location', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(batch)
    });
    
    const data = await response.json();
    results.push(data);
  }
  
  return results;
};
```

---

## Resumen de Validaciones

### Campos Requeridos
- ✅ `seatMap` es requerido
- ✅ `zone` es requerido
- ✅ `section` es requerido

### Campos Condicionales
- ✅ Para secciones numeradas (no por aforo): `row` y `seat` son requeridos
- ✅ Para secciones por aforo: `row` y `seat` se asignan automáticamente con valores por defecto

### Otros
- ✅ Debe enviar al menos una location
- ✅ Si se envía `_id`, la location debe existir (si no existe, se reporta como error pero no detiene el proceso)
- ✅ El formato puede ser array directo, objeto con propiedad `locations`, o objeto único
- ✅ `locationCode` es opcional (se genera automáticamente si no se envía)
- ✅ `isWithCapacity` es opcional (se detecta automáticamente si no se envía)
- ✅ `plan`, `table`, `chair`, `disabled` son opcionales

