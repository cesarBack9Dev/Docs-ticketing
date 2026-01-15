# Documentación: Get Event Planner By Origin

## Descripción

Este caso de uso permite obtener un Event Planner (Organizador de Eventos) basándose en el dominio desde donde se realiza la petición. Busca el event planner que coincida con el dominio proporcionado en los campos `domain` o `domains` del event planner.

## Endpoint

```
GET /api/v1/eventplanner/byorigin
```

## Autenticación

No requiere autenticación. Este endpoint es público.

## Parámetros de Entrada

### Headers

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Origin` | String | Sí | Dominio desde donde se realiza la petición. Puede incluir protocolo (https://) o no |

**Nota**: El sistema busca el origin en el siguiente orden:
1. Header `Origin`
2. Campo `origin` del request (si está disponible)
3. Header `Access-Control-Allow-Origin` (fallback)

### Ejemplo de Request

```bash
GET /api/v1/eventplanner/byorigin
Headers:
  Origin: https://gladiadores.b9ticketing.com
```

O sin protocolo:

```bash
GET /api/v1/eventplanner/byorigin
Headers:
  Origin: gladiadores.b9ticketing.com
```

## Comportamiento

### Búsqueda del Event Planner

El sistema busca el event planner que coincida con el dominio proporcionado usando búsqueda regex case-insensitive en:

1. **Campo `domain`** (string): Dominio principal del event planner
2. **Array `domains`** (array de strings): Array de dominios asociados al event planner

### Características de la Búsqueda

- **Case-insensitive**: No distingue entre mayúsculas y minúsculas
- **Flexible con protocolo**: Funciona tanto con `https://gladiadores.b9ticketing.com` como con `gladiadores.b9ticketing.com`
- **Seguridad**: Escapa caracteres especiales de regex para prevenir inyecciones
- **Filtrado**: Excluye automáticamente event planners con `hidden: true`

### Campos Removidos

Por seguridad, los siguientes campos se eliminan de la respuesta:

- `employees`: Array de empleados
- `adminId`: ID del administrador principal

## Respuesta Exitosa

### Status Code: 200

```json
{
  "doc": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Gladiadores",
    "code": "GLD001",
    "logo": "https://example.com/logo.png",
    "domain": "gladiadores.b9ticketing.com",
    "domains": [
      "https://gladiadores.b9ticketing.com",
      "https://www.gladiadores.b9ticketing.com"
    ],
    "contactEmail": "contacto@gladiadores.com",
    "currency": "USD",
    "withDolarPrice": true,
    "digitalBilling": true,
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

## Errores

### 400 - Bad Request

#### Error: Origin requerido

```json
{
  "error": {
    "message": "El dominio (origin) es requerido",
    "code": 400
  }
}
```

**Causa**: No se proporcionó el header `Origin` en la petición.

### 404 - Not Found

#### Error: Event planner no encontrado

```json
{
  "error": {
    "message": "Event planner no encontrado para el dominio proporcionado",
    "code": 404
  }
}
```

**Causa**: No existe un event planner que coincida con el dominio proporcionado, o el event planner está oculto (`hidden: true`).

## Ejemplos de Uso

### Ejemplo 1: Con protocolo HTTPS

```bash
GET /api/v1/eventplanner/byorigin
Headers:
  Origin: https://gladiadores.b9ticketing.com
```

**Response:**
```json
{
  "doc": {
    "_id": "...",
    "name": "Gladiadores",
    "domain": "gladiadores.b9ticketing.com",
    "domains": ["https://gladiadores.b9ticketing.com"]
  }
}
```

### Ejemplo 2: Sin protocolo

```bash
GET /api/v1/eventplanner/byorigin
Headers:
  Origin: gladiadores.b9ticketing.com
```

**Response:** (Igual que el ejemplo anterior)

### Ejemplo 3: Con Postman

**Configuración en Postman:**

1. **Método**: `GET`
2. **URL**: `http://localhost:3000/api/v1/eventplanner/byorigin`
3. **Headers**: Agregar:
   - **Key**: `Origin`
   - **Value**: `https://gladiadores.b9ticketing.com`

**Nota**: El header `Origin` es estándar en las peticiones HTTP y se envía automáticamente por los navegadores cuando se hace una petición desde JavaScript.

## Notas Importantes

1. **Formato de dominios**: Los dominios pueden estar almacenados en la base de datos con o sin protocolo (`https://`). El sistema busca coincidencias en ambos formatos.

2. **Búsqueda flexible**: La búsqueda es case-insensitive, por lo que `Gladiadores.B9Ticketing.com` coincidirá con `gladiadores.b9ticketing.com`.

3. **Seguridad**: Los caracteres especiales de regex se escapan automáticamente para prevenir inyecciones de regex.

4. **Event planners ocultos**: Los event planners con `hidden: true` no se retornan, incluso si el dominio coincide.

5. **Campos sensibles**: Los campos `employees` y `adminId` se eliminan automáticamente de la respuesta por seguridad.

## Casos de Uso

Este endpoint es útil para:

- **Multi-tenant**: Identificar qué event planner corresponde a cada dominio
- **White-label**: Permitir que diferentes dominios muestren diferentes configuraciones
- **Frontend dinámico**: Obtener la configuración del event planner basándose en el dominio desde donde se accede

## Ubicación del Código

- **Caso de uso**: `src/uses-cases/event-planner/get-event-planner-by-origin.js`
- **Controlador**: `src/controllers/event-planner-controller/get-event-planner-by-origin.js`
- **Ruta**: `src/index.js` (línea ~1147)
- **Índice de servicios**: `src/uses-cases/event-planner/index.js`

