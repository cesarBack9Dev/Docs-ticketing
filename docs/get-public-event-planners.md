# Documentación: Get Public Event Planners

## Descripción

Este endpoint retorna una lista **pública** de Event Planners (organizadores) visibles (excluye los que tengan `hidden: true`), con **paginación** y **ordenamiento**.

## Endpoint

```
GET /api/v1/eventplanners/public
```

## Autenticación

No requiere autenticación. Este endpoint es público.

## Parámetros de entrada

### Query params

| Parámetro | Tipo | Requerido | Default | Descripción |
|----------|------|-----------|---------|-------------|
| `page` | Number | No | `1` | Número de página (paginación) |
| `limit` | Number | No | `10` | Tamaño de página |
| `query` | String | No | - | Búsqueda por nombre del event planner (`name`) usando regex case-insensitive |
| `sortField` | String | No | `createdAt` | Campo por el cual ordenar. Permitidos: `name`, `code`, `domain`, `createdAt` |
| `sortType` | String | No | `desc` | Dirección de orden. Valores: `asc` o `desc` |

## Comportamiento

- **Filtro de visibilidad**: solo retorna event planners con `hidden: { $ne: true }` y `private: { $ne: true }`.
- **Selección de campos**: solo retorna `name`, `logo`, `code`, `domain` (no retorna el documento completo).
- **Búsqueda por nombre**:
  - Si viene `query`, filtra por `name` con `$regex` y `$options: "i"` (case-insensitive).
  - El valor del `query` se **escapa** para tratarlo como texto literal (evita problemas por caracteres especiales de regex).
- **Ordenamiento seguro**:
  - Si `sortField` no es permitido, se usa `createdAt`.
  - Si `sortType` no es `asc`, se usa `desc`.

## Respuesta exitosa

### Status Code: 200

El formato lo entrega `mongoose-paginate-v2`, típicamente:

```json
{
  "docs": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "Gladiadores",
      "logo": "https://example.com/logo.png",
      "code": "GLD001",
      "domain": "gladiadores.b9ticketing.com"
    }
  ],
  "totalDocs": 1,
  "limit": 10,
  "totalPages": 1,
  "page": 1,
  "pagingCounter": 1,
  "hasPrevPage": false,
  "hasNextPage": false,
  "prevPage": null,
  "nextPage": null
}
```

> Nota: los campos exactos adicionales pueden variar según versión/config de `mongoose-paginate-v2`, pero `docs` incluye únicamente `name logo code domain`.

## Errores

### 400 - Bad Request

```json
{
  "error": {}
}
```

> El controller actualmente responde `400` ante cualquier excepción no controlada.

## Ejemplos

### Ejemplo 1: Defaults

```bash
curl -X GET "http://localhost:3000/api/v1/eventplanners/public"
```

### Ejemplo 2: Paginación + orden por nombre asc

```bash
curl -X GET "http://localhost:3000/api/v1/eventplanners/public?page=2&limit=25&sortField=name&sortType=asc"
```

### Ejemplo 3: Buscar por nombre (regex)

```bash
curl -X GET "http://localhost:3000/api/v1/eventplanners/public?query=glad"
```

## Ubicación del código

- **Caso de uso**: `src/uses-cases/event-planner/get-public-event-planners.js`
- **Controlador**: `src/controllers/event-planner-controller/get-public-event-planners.js`
- **Ruta**: `src/index.js` (`/api/v1/eventplanners/public`)

