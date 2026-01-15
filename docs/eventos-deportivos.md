# Documentación: Creación de Eventos Deportivos y Categorías

Esta documentación describe cómo crear eventos deportivos tipo "running" (carreras) y sus categorías deportivas asociadas.

## Tabla de Contenidos

1. [Creación de Eventos Deportivos](#creación-de-eventos-deportivos)
2. [Creación de Categorías Deportivas](#creación-de-categorías-deportivas)
3. [Modos de Asignación de Dorsales](#modos-de-asignación-de-dorsales)
4. [Ejemplos Completos](#ejemplos-completos)

---

## Creación de Eventos Deportivos

### Endpoint

```
POST /api/v1/events
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Estructura del Body

El body debe contener un array de eventos en la propiedad `events`:

```json
{
  "events": [
    {
      // ... datos del evento
    }
  ]
}
```

### Campos del Evento

#### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `title` | String | Título del evento |
| `description` | String | Descripción del evento |
| `start.date` | String | Fecha y hora de inicio en formato `YYYY-MM-DDTHH:mm:ss` |
| `end.date` | String | Fecha y hora de finalización en formato `YYYY-MM-DDTHH:mm:ss` |

#### Campos Opcionales para Eventos Deportivos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `start.timezone` | String | Zona horaria (default: "America/Caracas") |
| `end.timezone` | String | Zona horaria (default: "America/Caracas") |
| `logo` | String | URL del logo del evento |
| `banner` | String | URL del banner del evento |
| `featuredBanner` | String | URL del banner destacado |
| `plansGroup` | String (ObjectId) | ID del grupo de planes de precios |
| `address` | Object | Información de ubicación del evento |
| `address.latitude` | String | Latitud de la ubicación |
| `address.longitude` | String | Longitud de la ubicación |
| `address.name` | String | Nombre de la ubicación |
| `address.mapUrl` | String | URL del mapa (opcional) |
| `eventType` | String | Tipo de evento (ej: "running") |
| `sportCategories` | Array[String] | Array de IDs de categorías deportivas |
| `dorsalAssignedType` | String | Tipo de asignación: "AUTO" o "MANUAL" |
| `dorsalAssignmentMode` | String | Modo de asignación: "BY_AGE", "BY_CATEGORY", o "BY_AGE_AND_CATEGORY" |

### Ejemplo de Request

```json
{
  "events": [
    {
      "title": "Carrera en El Morro Lechería 10K",
      "description": "Vive la experiencia de correr junto al mar en El Morro, Lechería. Un recorrido plano y panorámico ideal para corredores de todas las categorías: Elite, SubMaster y Master.",
      "start": {
        "date": "2025-12-15T06:30:00",
        "timezone": "America/Caracas"
      },
      "end": {
        "date": "2025-12-15T11:30:00",
        "timezone": "America/Caracas"
      },
      "logo": "https://b9ticketing.s3.amazonaws.com/logos/carrera-morro-lecheria.png",
      "plansGroup": "6915fa1f9baf9454d4ffd3ed",
      "address": {
        "latitude": "10.186675",
        "longitude": "-64.628732",
        "name": "Complejo Turístico El Morro, Lechería, Venezuela",
        "mapUrl": "https://maps.google.com/?q=10.186675,-64.628732"
      },
      "eventType": "running",
      "sportCategories": [
        "6915f76594324863d6b9ffb9",
        "6915f76594324863d6b9ffba",
        "6915f76594324863d6b9ffbb",
        "6915f76594324863d6b9ffbc",
        "6915f76594324863d6b9ffbd",
        "6915f76594324863d6b9ffbe",
        "6915f76a94324863d6b9ffc7",
        "6915f76a94324863d6b9ffc8",
        "6915f76a94324863d6b9ffc9",
        "6915f76a94324863d6b9ffca",
        "6915f76a94324863d6b9ffcb",
        "6915f76a94324863d6b9ffcc"
      ],
      "dorsalAssignedType": "AUTO",
      "dorsalAssignmentMode": "BY_CATEGORY"
    }
  ]
}
```

### Respuesta Exitosa

```json
{
  "message": "Okey"
}
```

### Errores Comunes

- **400 Bad Request**: Si falta algún campo requerido o el formato de fecha es incorrecto
- **401 Unauthorized**: Si el token de autenticación es inválido o falta

---

## Creación de Categorías Deportivas

### Endpoint

```
POST /api/v1/sport-categories
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Estructura del Body

El body debe contener un array de categorías en la propiedad `categories`:

```json
{
  "categories": [
    {
      // ... datos de la categoría
    }
  ]
}
```

### Campos de la Categoría

#### Campos Opcionales (pero recomendados)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | String | Nombre de la categoría (ej: "Elite Masculino", "Master Femenino") |
| `classificationType` | String | Tipo de clasificación: "AGE", "BIRTH_YEAR", o "GENDER" (default: "AGE") |
| `minAge` | Number | Edad mínima para la categoría (requerido si classificationType es "AGE") |
| `maxAge` | Number | Edad máxima para la categoría (requerido si classificationType es "AGE") |
| `minBirthYear` | Number | Año de nacimiento mínimo (requerido si classificationType es "BIRTH_YEAR") |
| `maxBirthYear` | Number | Año de nacimiento máximo (requerido si classificationType es "BIRTH_YEAR") |
| `gender` | String | Género: "MALE", "FEMALE", o "ALL" (default: "ALL") |
| `startBib` | Number | Número de dorsal inicial (opcional) |
| `endBib` | Number | Número de dorsal final (opcional) |
| `lastAssignBib` | Number | Último dorsal asignado (default: 0) |
| `_id` | String | ID de la categoría (solo si se está actualizando) |

### Tipos de Clasificación

#### 1. Por Edad (`classificationType: "AGE"`)

Se usa cuando la categoría se determina por la edad del participante.

```json
{
  "name": "Elite Masculino",
  "classificationType": "AGE",
  "minAge": 18,
  "maxAge": 29,
  "gender": "MALE"
}
```

#### 2. Por Año de Nacimiento (`classificationType: "BIRTH_YEAR"`)

Se usa cuando la categoría se determina por el año de nacimiento.

```json
{
  "name": "Master 40-49",
  "classificationType": "BIRTH_YEAR",
  "minBirthYear": 1975,
  "maxBirthYear": 1984,
  "gender": "ALL"
}
```

#### 3. Por Género (`classificationType: "GENDER"`)

Se usa cuando la categoría se determina solo por género (sin restricción de edad).

```json
{
  "name": "Categoría Libre",
  "classificationType": "GENDER",
  "gender": "MALE"
}
```

### Ejemplo de Request

```json
{
  "categories": [
    {
      "name": "Elite Masculino",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "MALE",
      "startBib": 1,
      "endBib": 100
    },
    {
      "name": "Elite Femenino",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "FEMALE",
      "startBib": 101,
      "endBib": 200
    },
    {
      "name": "SubMaster Masculino",
      "classificationType": "AGE",
      "minAge": 30,
      "maxAge": 39,
      "gender": "MALE",
      "startBib": 201,
      "endBib": 300
    },
    {
      "name": "SubMaster Femenino",
      "classificationType": "AGE",
      "minAge": 30,
      "maxAge": 39,
      "gender": "FEMALE",
      "startBib": 301,
      "endBib": 400
    },
    {
      "name": "Master 40-49 Masculino",
      "classificationType": "AGE",
      "minAge": 40,
      "maxAge": 49,
      "gender": "MALE",
      "startBib": 401,
      "endBib": 500
    },
    {
      "name": "Master 40-49 Femenino",
      "classificationType": "AGE",
      "minAge": 40,
      "maxAge": 49,
      "gender": "FEMALE",
      "startBib": 501,
      "endBib": 600
    },
    {
      "name": "Master 50-59 Masculino",
      "classificationType": "AGE",
      "minAge": 50,
      "maxAge": 59,
      "gender": "MALE",
      "startBib": 601,
      "endBib": 700
    },
    {
      "name": "Master 50-59 Femenino",
      "classificationType": "AGE",
      "minAge": 50,
      "maxAge": 59,
      "gender": "FEMALE",
      "startBib": 701,
      "endBib": 800
    },
    {
      "name": "Master 60+ Masculino",
      "classificationType": "AGE",
      "minAge": 60,
      "maxAge": 999,
      "gender": "MALE",
      "startBib": 801,
      "endBib": 900
    },
    {
      "name": "Master 60+ Femenino",
      "classificationType": "AGE",
      "minAge": 60,
      "maxAge": 999,
      "gender": "FEMALE",
      "startBib": 901,
      "endBib": 1000
    },
    {
      "name": "Juvenil Masculino",
      "classificationType": "AGE",
      "minAge": 16,
      "maxAge": 17,
      "gender": "MALE",
      "startBib": 1001,
      "endBib": 1100
    },
    {
      "name": "Juvenil Femenino",
      "classificationType": "AGE",
      "minAge": 16,
      "maxAge": 17,
      "gender": "FEMALE",
      "startBib": 1101,
      "endBib": 1200
    }
  ]
}
```

### Respuesta Exitosa

La respuesta incluye información detallada sobre cada categoría procesada:

```json
{
  "message": "Procesadas 3 categoría(s): 2 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffb9",
        "name": "Elite Masculino",
        "classificationType": "AGE",
        "minAge": 18,
        "maxAge": 29,
        "gender": "MALE",
        "startBib": 1,
        "endBib": 100,
        "lastAssignBib": 0,
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    },
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffba",
        "name": "Elite Femenino",
        "classificationType": "AGE",
        "minAge": 18,
        "maxAge": 29,
        "gender": "FEMALE",
        "startBib": 101,
        "endBib": 200,
        "lastAssignBib": 0,
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    },
    {
      "operation": "updated",
      "category": {
        "_id": "6915f76594324863d6b9ffbb",
        "name": "SubMaster Masculino Actualizado",
        "classificationType": "AGE",
        "minAge": 30,
        "maxAge": 39,
        "gender": "MALE",
        "startBib": 201,
        "endBib": 300,
        "lastAssignBib": 0,
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
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

#### Campos de la Respuesta

- **`message`**: Mensaje descriptivo con el resumen del procesamiento
- **`results`**: Array con el resultado de cada categoría procesada, en el mismo orden que fueron enviadas
  - **`operation`**: Tipo de operación realizada: `"created"`, `"updated"`, o `"error"`
  - **`category`**: Objeto completo de la categoría creada o actualizada (solo presente si la operación fue exitosa)
  - **`error`**: Mensaje de error (solo presente si `operation` es `"error"`)
  - **`_id`**: ID de la categoría (presente en todos los casos)
- **`summary`**: Resumen numérico del procesamiento
  - **`total`**: Total de categorías procesadas
  - **`created`**: Cantidad de categorías creadas
  - **`updated`**: Cantidad de categorías actualizadas
  - **`errors`**: Cantidad de errores encontrados

### Actualizar Categorías Existentes

Para actualizar una categoría existente, incluye el `_id` en el objeto:

```json
{
  "categories": [
    {
      "_id": "6915f76594324863d6b9ffb9",
      "name": "Elite Masculino Actualizado",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "MALE"
    }
  ]
}
```

**Respuesta:**

```json
{
  "message": "Procesadas 1 categoría(s): 0 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "updated",
      "category": {
        "_id": "6915f76594324863d6b9ffb9",
        "name": "Elite Masculino Actualizado",
        "classificationType": "AGE",
        "minAge": 18,
        "maxAge": 29,
        "gender": "MALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    }
  ],
  "summary": {
    "total": 1,
    "created": 0,
    "updated": 1,
    "errors": 0
  }
}
```

### Crear y Actualizar en la Misma Petición

Puedes mezclar creaciones y actualizaciones en una sola petición:

```json
{
  "categories": [
    {
      "name": "Nueva Categoría",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "MALE"
    },
    {
      "_id": "6915f76594324863d6b9ffb9",
      "name": "Categoría Actualizada",
      "classificationType": "AGE",
      "minAge": 30,
      "maxAge": 39,
      "gender": "FEMALE"
    }
  ]
}
```

**Respuesta:**

```json
{
  "message": "Procesadas 2 categoría(s): 1 creada(s), 1 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffc0",
        "name": "Nueva Categoría",
        "classificationType": "AGE",
        "minAge": 18,
        "maxAge": 29,
        "gender": "MALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    },
    {
      "operation": "updated",
      "category": {
        "_id": "6915f76594324863d6b9ffb9",
        "name": "Categoría Actualizada",
        "classificationType": "AGE",
        "minAge": 30,
        "maxAge": 39,
        "gender": "FEMALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "updated": 1,
    "errors": 0
  }
}
```

---

## Modos de Asignación de Dorsales

El campo `dorsalAssignmentMode` del evento determina cómo se asignan los dorsales a los participantes:

### BY_AGE

Asigna dorsales basándose en la edad del participante y su género. Busca la categoría que coincida con:
- `classificationType === "AGE"`
- `minAge <= edad del participante <= maxAge`
- `gender === género del participante`

**Ejemplo:**
- Participante: 25 años, Masculino
- Se busca categoría con `minAge: 18, maxAge: 29, gender: "MALE"`

### BY_CATEGORY

Asigna dorsales basándose solo en la categoría y género. Busca la categoría que coincida con:
- `classificationType === "CATEGORY"` (Nota: actualmente el código busca "CATEGORY" pero el modelo solo permite "AGE", "BIRTH_YEAR", "GENDER")
- `gender === género del participante`

**Ejemplo:**
- Participante: Masculino
- Se busca categoría con `gender: "MALE"` (sin restricción de edad)

### BY_AGE_AND_CATEGORY

Asigna dorsales basándose en edad y categoría. Busca la categoría que coincida con:
- `classificationType === "AGE"`
- `minAge <= edad del participante <= maxAge`
- `gender === género del participante`

**Ejemplo:**
- Participante: 35 años, Femenino
- Se busca categoría con `minAge: 30, maxAge: 39, gender: "FEMALE"`

---

## Ejemplos Completos

### Flujo Completo: Crear Categorías y Evento

#### Paso 1: Crear las Categorías Deportivas

```bash
POST /api/v1/sport-categories
Authorization: Bearer {token}
Content-Type: application/json

{
  "categories": [
    {
      "name": "Elite Masculino",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "MALE"
    },
    {
      "name": "Elite Femenino",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "FEMALE"
    },
    {
      "name": "SubMaster Masculino",
      "classificationType": "AGE",
      "minAge": 30,
      "maxAge": 39,
      "gender": "MALE"
    },
    {
      "name": "SubMaster Femenino",
      "classificationType": "AGE",
      "minAge": 30,
      "maxAge": 39,
      "gender": "FEMALE"
    }
  ]
}
```

**Respuesta:**
```json
{
  "message": "Procesadas 4 categoría(s): 4 creada(s), 0 actualizada(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffb9",
        "name": "Elite Masculino",
        "classificationType": "AGE",
        "minAge": 18,
        "maxAge": 29,
        "gender": "MALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    },
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffba",
        "name": "Elite Femenino",
        "classificationType": "AGE",
        "minAge": 18,
        "maxAge": 29,
        "gender": "FEMALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    },
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffbb",
        "name": "SubMaster Masculino",
        "classificationType": "AGE",
        "minAge": 30,
        "maxAge": 39,
        "gender": "MALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    },
    {
      "operation": "created",
      "category": {
        "_id": "6915f76594324863d6b9ffbc",
        "name": "SubMaster Femenino",
        "classificationType": "AGE",
        "minAge": 30,
        "maxAge": 39,
        "gender": "FEMALE",
        "eventPlanner": "6915fa1f9baf9454d4ffd3ed"
      }
    }
  ],
  "summary": {
    "total": 4,
    "created": 4,
    "updated": 0,
    "errors": 0
  }
}
```

**Nota:** Los IDs de las categorías creadas están disponibles directamente en la respuesta (`results[].category._id`), por lo que no es necesario hacer una petición adicional para obtenerlos.

#### Paso 2: Obtener los IDs de las Categorías Creadas (Opcional)

```bash
GET /api/v1/sport-categories
Authorization: Bearer {token}
```

**Respuesta:**
```json
{
  "categories": [
    {
      "_id": "6915f76594324863d6b9ffb9",
      "name": "Elite Masculino",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "MALE"
    },
    {
      "_id": "6915f76594324863d6b9ffba",
      "name": "Elite Femenino",
      "classificationType": "AGE",
      "minAge": 18,
      "maxAge": 29,
      "gender": "FEMALE"
    },
    // ... más categorías
  ]
}
```

#### Paso 3: Crear el Evento con las Categorías

```bash
POST /api/v1/events
Authorization: Bearer {token}
Content-Type: application/json

{
  "events": [
    {
      "title": "Carrera en El Morro Lechería 10K",
      "description": "Vive la experiencia de correr junto al mar en El Morro, Lechería.",
      "start": {
        "date": "2025-12-15T06:30:00",
        "timezone": "America/Caracas"
      },
      "end": {
        "date": "2025-12-15T11:30:00",
        "timezone": "America/Caracas"
      },
      "logo": "https://b9ticketing.s3.amazonaws.com/logos/carrera-morro-lecheria.png",
      "plansGroup": "6915fa1f9baf9454d4ffd3ed",
      "address": {
        "latitude": "10.186675",
        "longitude": "-64.628732",
        "name": "Complejo Turístico El Morro, Lechería, Venezuela",
        "mapUrl": "https://maps.google.com/?q=10.186675,-64.628732"
      },
      "eventType": "running",
      "sportCategories": [
        "6915f76594324863d6b9ffb9",
        "6915f76594324863d6b9ffba",
        "6915f76594324863d6b9ffbb",
        "6915f76594324863d6b9ffbc"
      ],
      "dorsalAssignedType": "AUTO",
      "dorsalAssignmentMode": "BY_CATEGORY"
    }
  ]
}
```

**Respuesta:**
```json
{
  "message": "Okey"
}
```

---

## Notas Importantes

1. **Formato de Fechas**: Las fechas deben estar en formato ISO 8601: `YYYY-MM-DDTHH:mm:ss`
   - Ejemplo: `"2025-12-15T06:30:00"`

2. **IDs de Categorías**: Los IDs de las categorías deportivas deben existir antes de asociarlas al evento.

3. **Asignación Automática de Dorsales**: Cuando `dorsalAssignedType` es `"AUTO"`, el sistema asignará automáticamente los dorsales según el `dorsalAssignmentMode` configurado.

4. **Rangos de Dorsales**: Los campos `startBib` y `endBib` en las categorías son opcionales pero recomendados para controlar los rangos de dorsales por categoría.

5. **Event Planner**: El `eventPlanner` se asigna automáticamente desde el token de autenticación, no es necesario incluirlo en el request.

---

## Errores Comunes y Soluciones

### Error: "Titulo del evento es requerido"
**Solución**: Asegúrate de incluir el campo `title` en el evento.

### Error: "el formato de la fecha inicio debe ser YYYY-MM-DDTHH:mm:ss"
**Solución**: Verifica que las fechas estén en el formato correcto. Ejemplo: `"2025-12-15T06:30:00"`

### Error: "No se encontró la categoría"
**Solución**: Verifica que:
- Las categorías estén creadas antes del evento
- Los IDs de las categorías en `sportCategories` sean correctos
- El `dorsalAssignmentMode` coincida con el `classificationType` de las categorías

### Error: "Las categorias deben ser array"
**Solución**: Asegúrate de que el campo `categories` sea un array, incluso si solo hay una categoría.

---

## Referencias

- Endpoint de eventos: `POST /api/v1/events`
- Endpoint de categorías: `POST /api/v1/sport-categories`
- Endpoint para obtener categorías: `GET /api/v1/sport-categories`

