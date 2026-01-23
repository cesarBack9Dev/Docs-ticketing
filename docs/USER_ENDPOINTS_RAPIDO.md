# Endpoints rápidos (Usuario)

## Abonos (season tickets) de un usuario

- **Método**: `GET`
- **Ruta**: `/api/v1/user/tickets/:eventPlanner`
- **Query params**:
  - `page`: número de página (ej. `1`)
  - `limit`: tamaño de página (ej. `10`)
  - `ticketType`: filtra tipo de ticket (para abonos usar `seasonTicket`)

Ejemplo:

```http
GET /api/v1/user/tickets/B9DEMO?page=1&limit=10&ticketType=seasonTicket
```

## Actualizar contact info de un ticket (usuario)

- **Método**: `PUT`
- **Ruta**: `/api/v1/tickets/update-user-ticket-info`
- **Descripción**: Permite al usuario actualizar **solo** `contactInfo` del ticket.

Ejemplo body (actualizar documento):

```json
{
  "ticketCode": "ABC123",
  "contactInfo": {
    "document": "24827248"
  }
}
```

### Foto de perfil del abono (season ticket avatar)

Para agregar/actualizar la foto de perfil del **season ticket**, usar:

```json
{
  "ticketCode": "ABC123",
  "contactInfo": {
    "aditionalInfo": {
      "seasonTicketAvatar": "Photo"
    }
  }
}
```

## Cerrar cuenta

- **Método**: `POST`
- **Ruta**: `/api/v1/users/closeaccount`

## Editar perfil

- **Método**: `POST`
- **Ruta**: `/api/v1/users/updateuser`

### Campos que se pueden editar

El endpoint actualiza los campos que envíes en el body, **excepto**:

- `email` (se ignora)
- `userRoll` (se ignora)
- `isActive` (se ignora)

Campos típicos de perfil (según modelo) que puedes editar:

- `firstName`
- `lastName`
- `phone`
- `document`
- `address`
- `station`
- `aditionalInfo` (objeto libre)
- `password` (si lo envías, se guarda **hasheado**)

Ejemplo:

```json
{
  "firstName": "Carlos",
  "lastName": "Pérez",
  "phone": "04141234567",
  "document": "24827248",
  "address": "Caracas",
  "station": "Altamira",
  "aditionalInfo": {
    "avatar": "base64-or-url"
  }
}
```
