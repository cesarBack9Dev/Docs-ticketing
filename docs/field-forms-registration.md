# Documentación: Campos de Formulario de Inscripción (Field Forms)

Esta documentación describe cómo crear y gestionar campos de formulario dinámicos para eventos de inscripción, y cómo asociarlos con eventos.

## Tabla de Contenidos

1. [Crear Campos de Formulario](#crear-campos-de-formulario)
2. [Obtener Campos de Formulario](#obtener-campos-de-formulario)
3. [Asociar Campos de Formulario a un Evento](#asociar-campos-de-formulario-a-un-evento)
4. [Obtener Evento con Field Forms](#obtener-evento-con-field-forms)
5. [Obtener Eventos con Field Forms](#obtener-eventos-con-field-forms)
6. [Tipos de Campos Disponibles](#tipos-de-campos-disponibles)
7. [Ejemplos Completos](#ejemplos-completos)

---

## Crear Campos de Formulario

### Endpoint

```
POST /api/v1/field-form
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Descripción

Crea o actualiza campos de formulario dinámicos. Puedes crear múltiples campos en una sola petición y también editar campos existentes.

### Estructura del Body

```json
{
  "fieldForms": [
    {
      "_id": "507f1f77bcf86cd799439011",  // Opcional: solo si es edición
      "label": "Nombre Completo",          // Requerido
      "name": "fullName",                  // Requerido
      "key": "fullName",                   // Opcional: se usa 'name' si no se envía
      "type": "text",                      // Requerido
      "required": true,                    // Opcional: default true
      "placeholder": "Ingrese su nombre",  // Opcional
      "defaultValue": "",                  // Opcional
      "options": [                         // Requerido solo para select, radio
        {
          "label": "Opción 1",
          "value": "option1"
        }
      ],
      "validation": {                      // Opcional
        "minLength": 3,
        "maxLength": 50,
        "pattern": "^[a-zA-Z\\s]+$",
        "min": 0,
        "max": 100
      }
    }
  ]
}
```

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `fieldForms` | Array | Array de campos de formulario (requerido) |
| `fieldForms[].label` | String | Etiqueta visible del campo (requerido) |
| `fieldForms[].name` | String | Nombre único del campo (requerido) |
| `fieldForms[].type` | String | Tipo de campo (requerido) |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `fieldForms[]._id` | String | ID del campo (solo para edición) |
| `fieldForms[].key` | String | Clave única (default: usa `name`) |
| `fieldForms[].required` | Boolean | Si el campo es requerido (default: `true`) |
| `fieldForms[].placeholder` | String | Texto de ayuda del campo |
| `fieldForms[].defaultValue` | Mixed | Valor por defecto |
| `fieldForms[].options` | Array | Opciones para select/radio (requerido para estos tipos) |
| `fieldForms[].validation` | Object | Reglas de validación |

### Tipos de Campos Disponibles

- `"text"` - Campo de texto
- `"number"` - Campo numérico
- `"email"` - Campo de email
- `"phone"` - Campo de teléfono
- `"date"` - Campo de fecha
- `"select"` - Select dropdown (requiere `options`)
- `"textarea"` - Área de texto
- `"checkbox"` - Checkbox (puede ser simple o con opciones)
- `"radio"` - Radio button (requiere `options`)
- `"file"` - Campo de archivo
- `"password"` - Campo de contraseña

### Ejemplos de Request

#### 1. Crear un campo de texto simple

```json
{
  "fieldForms": [
    {
      "label": "Nombre",
      "name": "firstName",
      "type": "text",
      "required": true,
      "placeholder": "Ingrese su nombre"
    }
  ]
}
```

#### 2. Crear un campo select con opciones

```json
{
  "fieldForms": [
    {
      "label": "País",
      "name": "country",
      "type": "select",
      "required": true,
      "options": [
        {
          "label": "Venezuela",
          "value": "VE"
        },
        {
          "label": "Colombia",
          "value": "CO"
        },
        {
          "label": "Estados Unidos",
          "value": "US"
        }
      ]
    }
  ]
}
```

#### 3. Crear un checkbox simple (términos y condiciones)

```json
{
  "fieldForms": [
    {
      "label": "Acepto los términos y condiciones",
      "name": "acceptTerms",
      "type": "checkbox",
      "required": true
    }
  ]
}
```

#### 4. Crear múltiples campos a la vez

```json
{
  "fieldForms": [
    {
      "label": "Nombre",
      "name": "firstName",
      "type": "text",
      "required": true
    },
    {
      "label": "Email",
      "name": "email",
      "type": "email",
      "required": true,
      "validation": {
        "pattern": "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"
      }
    },
    {
      "label": "Teléfono",
      "name": "phone",
      "type": "phone",
      "required": false
    },
    {
      "label": "Fecha de Nacimiento",
      "name": "birthDate",
      "type": "date",
      "required": true
    }
  ]
}
```

#### 5. Editar un campo existente

```json
{
  "fieldForms": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "label": "Nombre Actualizado",
      "name": "firstName",
      "type": "text",
      "required": true,
      "placeholder": "Nuevo placeholder"
    }
  ]
}
```

### Respuesta Exitosa

```json
{
  "message": "Procesados 3 campo(s) de formulario: 2 creado(s), 1 actualizado(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "fieldForm": {
        "_id": "507f1f77bcf86cd799439011",
        "eventPlanner": "507f1f77bcf86cd799439010",
        "label": "Nombre",
        "name": "firstName",
        "key": "firstName",
        "type": "text",
        "required": true,
        "placeholder": "Ingrese su nombre",
        "options": [],
        "validation": {},
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    },
    {
      "operation": "created",
      "fieldForm": {
        "_id": "507f1f77bcf86cd799439012",
        "label": "Email",
        "name": "email",
        "type": "email",
        "required": true
      }
    },
    {
      "operation": "updated",
      "fieldForm": {
        "_id": "507f1f77bcf86cd799439013",
        "label": "Teléfono Actualizado",
        "name": "phone",
        "type": "phone",
        "required": false
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

### Errores Comunes

- **400 Bad Request**: Si falta algún campo requerido
- **400 Bad Request**: Si un campo de tipo `select` o `radio` no tiene opciones
- **400 Bad Request**: Si `fieldForms` no es un array

---

## Obtener Campos de Formulario

### Endpoint

```
GET /api/v1/field-form
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Descripción

Obtiene todos los campos de formulario del eventPlanner autenticado. El `eventPlanner` se obtiene automáticamente del token.

### Respuesta Exitosa

```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "eventPlanner": "507f1f77bcf86cd799439010",
    "label": "Nombre",
    "name": "firstName",
    "key": "firstName",
    "type": "text",
    "required": true,
    "placeholder": "Ingrese su nombre",
    "defaultValue": null,
    "options": [],
    "validation": {},
    "createdAt": "2024-01-15T10:30:00.000Z"
  },
  {
    "_id": "507f1f77bcf86cd799439012",
    "eventPlanner": "507f1f77bcf86cd799439010",
    "label": "País",
    "name": "country",
    "key": "country",
    "type": "select",
    "required": true,
    "options": [
      {
        "label": "Venezuela",
        "value": "VE"
      },
      {
        "label": "Colombia",
        "value": "CO"
      }
    ],
    "validation": {},
    "createdAt": "2024-01-15T10:31:00.000Z"
  }
]
```

---

## Asociar Campos de Formulario a un Evento

### Endpoint

```
POST /api/v1/events
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Descripción

Para crear un evento con formulario de inscripción, debes:

1. **Crear los campos de formulario** primero usando el endpoint de field-forms
2. **Crear o editar el evento** incluyendo:
   - `fieldForms`: Array de IDs de los campos de formulario
   - `hasRegistrationForm`: `true` para activar el formulario de inscripción

### Estructura del Body

```json
{
  "events": [
    {
      "title": "Evento con Formulario de Inscripción",
      "description": "Descripción del evento",
      "start": {
        "date": "2024-12-15T06:30:00",
        "timezone": "America/Caracas"
      },
      "end": {
        "date": "2024-12-15T11:30:00",
        "timezone": "America/Caracas"
      },
      "fieldForms": [
        "507f1f77bcf86cd799439011",
        "507f1f77bcf86cd799439012",
        "507f1f77bcf86cd799439013"
      ],
      "hasRegistrationForm": true
    }
  ]
}
```

### Campos Importantes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `fieldForms` | Array[String] | Array de IDs de los campos de formulario |
| `hasRegistrationForm` | Boolean | Debe ser `true` para activar el formulario de inscripción |

### Ejemplo Completo: Flujo de Trabajo

#### Paso 1: Crear los campos de formulario

```json
POST /api/v1/field-form
{
  "fieldForms": [
    {
      "label": "Nombre Completo",
      "name": "fullName",
      "type": "text",
      "required": true
    },
    {
      "label": "Email",
      "name": "email",
      "type": "email",
      "required": true
    },
    {
      "label": "Teléfono",
      "name": "phone",
      "type": "phone",
      "required": false
    },
    {
      "label": "Acepto los términos",
      "name": "acceptTerms",
      "type": "checkbox",
      "required": true
    }
  ]
}
```

**Respuesta:**
```json
{
  "message": "Procesados 4 campo(s) de formulario: 4 creado(s), 0 actualizado(s), 0 error(es)",
  "results": [
    {
      "operation": "created",
      "fieldForm": {
        "_id": "507f1f77bcf86cd799439011",
        "label": "Nombre Completo",
        "name": "fullName",
        ...
      }
    },
    ...
  ],
  "summary": {
    "total": 4,
    "created": 4,
    "updated": 0,
    "errors": 0
  }
}
```

#### Paso 2: Crear el evento con los fieldForms

```json
POST /api/v1/events
{
  "events": [
    {
      "title": "Taller de Programación",
      "description": "Taller intensivo de programación",
      "start": {
        "date": "2024-12-15T09:00:00",
        "timezone": "America/Caracas"
      },
      "end": {
        "date": "2024-12-15T17:00:00",
        "timezone": "America/Caracas"
      },
      "fieldForms": [
        "507f1f77bcf86cd799439011",
        "507f1f77bcf86cd799439012",
        "507f1f77bcf86cd799439013",
        "507f1f77bcf86cd799439014"
      ],
      "hasRegistrationForm": true
    }
  ]
}
```

#### Paso 3: Editar un evento existente para agregar fieldForms

```json
POST /api/v1/events
{
  "events": [
    {
      "_id": "507f1f77bcf86cd799439020",
      "fieldForms": [
        "507f1f77bcf86cd799439011",
        "507f1f77bcf86cd799439012"
      ],
      "hasRegistrationForm": true
    }
  ]
}
```

### Notas Importantes

- **`hasRegistrationForm` debe ser `true`**: Sin esto, el evento no se reconocerá como evento de inscripción
- **Los IDs de `fieldForms` deben existir**: Asegúrate de crear los campos primero
- **Los `fieldForms` deben pertenecer al mismo `eventPlanner`**: No puedes usar campos de otro eventPlanner

---

## Obtener Evento con Field Forms

### Endpoint

```
GET /api/v1/events/:eventId
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Descripción

Obtiene un evento específico con sus campos de formulario poblados. Los `fieldForms` se incluyen automáticamente en la respuesta.

### Respuesta Exitosa

```json
{
  "_id": "507f1f77bcf86cd799439020",
  "title": "Taller de Programación",
  "description": "Taller intensivo de programación",
  "hasRegistrationForm": true,
  "fieldForms": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "eventPlanner": "507f1f77bcf86cd799439010",
      "label": "Nombre Completo",
      "name": "fullName",
      "key": "fullName",
      "type": "text",
      "required": true,
      "placeholder": "Ingrese su nombre completo",
      "options": [],
      "validation": {},
      "createdAt": "2024-01-15T10:30:00.000Z"
    },
    {
      "_id": "507f1f77bcf86cd799439012",
      "label": "Email",
      "name": "email",
      "type": "email",
      "required": true,
      "validation": {
        "pattern": "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"
      }
    },
    {
      "_id": "507f1f77bcf86cd799439013",
      "label": "Teléfono",
      "name": "phone",
      "type": "phone",
      "required": false
    },
    {
      "_id": "507f1f77bcf86cd799439014",
      "label": "Acepto los términos",
      "name": "acceptTerms",
      "type": "checkbox",
      "required": true
    }
  ],
  "start": {
    "date": "2024-12-15T09:00:00",
    "timezone": "America/Caracas"
  },
  "end": {
    "date": "2024-12-15T17:00:00",
    "timezone": "America/Caracas"
  },
  ...
}
```

---

## Obtener Eventos con Field Forms

### Endpoint

```
GET /api/v1/events
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Query Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `page` | Number | Número de página (default: 1) |
| `limit` | Number | Cantidad de resultados por página (default: 10) |
| `active` | Boolean | Filtrar solo eventos activos |
| `eventPlanner` | String | Código del eventPlanner |
| `hasRegistrationForm` | Boolean | Filtrar solo eventos con formulario de inscripción |

### Descripción

Obtiene una lista paginada de eventos. Los eventos que tienen `fieldForms` los incluirán automáticamente en la respuesta.

### Ejemplo de Request

```
GET /api/v1/events?page=1&limit=10&active=true&hasRegistrationForm=true
```

### Respuesta Exitosa

```json
{
  "docs": [
    {
      "_id": "507f1f77bcf86cd799439020",
      "title": "Taller de Programación",
      "hasRegistrationForm": true,
      "fieldForms": [
        {
          "_id": "507f1f77bcf86cd799439011",
          "label": "Nombre Completo",
          "name": "fullName",
          "type": "text",
          "required": true
        },
        {
          "_id": "507f1f77bcf86cd799439012",
          "label": "Email",
          "name": "email",
          "type": "email",
          "required": true
        }
      ],
      ...
    },
    {
      "_id": "507f1f77bcf86cd799439021",
      "title": "Conferencia de Tecnología",
      "hasRegistrationForm": false,
      "fieldForms": [],
      ...
    }
  ],
  "totalDocs": 25,
  "limit": 10,
  "page": 1,
  "totalPages": 3
}
```

---

## Tipos de Campos Disponibles

### Text

```json
{
  "label": "Nombre",
  "name": "firstName",
  "type": "text",
  "required": true,
  "placeholder": "Ingrese su nombre",
  "validation": {
    "minLength": 2,
    "maxLength": 50
  }
}
```

### Email

```json
{
  "label": "Email",
  "name": "email",
  "type": "email",
  "required": true,
  "validation": {
    "pattern": "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"
  }
}
```

### Number

```json
{
  "label": "Edad",
  "name": "age",
  "type": "number",
  "required": true,
  "validation": {
    "min": 18,
    "max": 100
  }
}
```

### Select

```json
{
  "label": "País",
  "name": "country",
  "type": "select",
  "required": true,
  "options": [
    {
      "label": "Venezuela",
      "value": "VE"
    },
    {
      "label": "Colombia",
      "value": "CO"
    }
  ]
}
```

### Radio

```json
{
  "label": "Género",
  "name": "gender",
  "type": "radio",
  "required": true,
  "options": [
    {
      "label": "Masculino",
      "value": "M"
    },
    {
      "label": "Femenino",
      "value": "F"
    }
  ]
}
```

### Checkbox Simple

```json
{
  "label": "Acepto los términos y condiciones",
  "name": "acceptTerms",
  "type": "checkbox",
  "required": true
}
```

### Checkbox con Opciones (Múltiple selección)

```json
{
  "label": "Intereses",
  "name": "interests",
  "type": "checkbox",
  "required": false,
  "options": [
    {
      "label": "Deportes",
      "value": "sports"
    },
    {
      "label": "Música",
      "value": "music"
    },
    {
      "label": "Tecnología",
      "value": "tech"
    }
  ]
}
```

### Date

```json
{
  "label": "Fecha de Nacimiento",
  "name": "birthDate",
  "type": "date",
  "required": true
}
```

### Textarea

```json
{
  "label": "Comentarios",
  "name": "comments",
  "type": "textarea",
  "required": false,
  "validation": {
    "maxLength": 500
  }
}
```

### File

```json
{
  "label": "Subir CV",
  "name": "cv",
  "type": "file",
  "required": false
}
```

---

## Ejemplos Completos

### Ejemplo 1: Evento de Inscripción Completo

#### 1. Crear campos de formulario

```json
POST /api/v1/field-form
{
  "fieldForms": [
    {
      "label": "Nombre Completo",
      "name": "fullName",
      "type": "text",
      "required": true,
      "placeholder": "Ingrese su nombre completo"
    },
    {
      "label": "Email",
      "name": "email",
      "type": "email",
      "required": true,
      "validation": {
        "pattern": "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"
      }
    },
    {
      "label": "Teléfono",
      "name": "phone",
      "type": "phone",
      "required": false
    },
    {
      "label": "País",
      "name": "country",
      "type": "select",
      "required": true,
      "options": [
        { "label": "Venezuela", "value": "VE" },
        { "label": "Colombia", "value": "CO" },
        { "label": "Estados Unidos", "value": "US" }
      ]
    },
    {
      "label": "Acepto los términos y condiciones",
      "name": "acceptTerms",
      "type": "checkbox",
      "required": true
    }
  ]
}
```

#### 2. Crear evento con formulario

```json
POST /api/v1/events
{
  "events": [
    {
      "title": "Taller de Desarrollo Web",
      "description": "Aprende desarrollo web desde cero",
      "start": {
        "date": "2024-12-20T09:00:00",
        "timezone": "America/Caracas"
      },
      "end": {
        "date": "2024-12-20T17:00:00",
        "timezone": "America/Caracas"
      },
      "fieldForms": [
        "507f1f77bcf86cd799439011",
        "507f1f77bcf86cd799439012",
        "507f1f77bcf86cd799439013",
        "507f1f77bcf86cd799439014",
        "507f1f77bcf86cd799439015"
      ],
      "hasRegistrationForm": true
    }
  ]
}
```

#### 3. Obtener evento con formulario

```
GET /api/v1/events/507f1f77bcf86cd799439020
```

La respuesta incluirá todos los `fieldForms` poblados con sus datos completos.

---

## Validaciones y Reglas

### Validaciones por Tipo

1. **`select` y `radio`**: Deben tener al menos una opción en el array `options`
2. **`checkbox`**: Puede ser simple (sin opciones) o múltiple (con opciones)
3. **Todos los campos**: Deben tener `label`, `name` y `type`

### Reglas de Negocio

1. **`hasRegistrationForm`**: Debe ser `true` para que el evento se reconozca como evento de inscripción
2. **`fieldForms`**: Los IDs deben existir y pertenecer al mismo `eventPlanner`
3. **Orden de los campos**: El orden en el array `fieldForms` del evento determina el orden de visualización

---

## Errores Comunes

### Error: Campo requerido falta

```json
{
  "error": {
    "message": "El campo 'type' es requerido"
  }
}
```

**Solución**: Asegúrate de incluir todos los campos requeridos (`label`, `name`, `type`).

### Error: Select/Radio sin opciones

```json
{
  "error": {
    "message": "El campo de tipo 'select' requiere al menos una opción"
  }
}
```

**Solución**: Agrega al menos una opción en el array `options` para campos de tipo `select` o `radio`.

### Error: FieldForms no encontrados

Si intentas asociar `fieldForms` que no existen o pertenecen a otro `eventPlanner`, el evento se creará pero los `fieldForms` no se asociarán correctamente.

**Solución**: Verifica que los IDs de `fieldForms` existan y pertenezcan al mismo `eventPlanner`.

---

## Notas Finales

- Los campos de formulario son **específicos por eventPlanner**: Cada eventPlanner tiene sus propios campos
- El campo `hasRegistrationForm` es **obligatorio** para activar el formulario de inscripción
- Los `fieldForms` se **poblan automáticamente** en `get-event` y `get-events`
- Puedes **editar campos existentes** enviando el `_id` en el array `fieldForms`
- Los campos se pueden **reutilizar** en múltiples eventos del mismo eventPlanner

