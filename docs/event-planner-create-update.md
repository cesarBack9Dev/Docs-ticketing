# Documentación: Creación y Edición de Event Planner

## Descripción

Esta documentación describe los campos disponibles para la **creación** y **edición** de un Event Planner (Organizador de Eventos) en el sistema.

---

## Creación de Event Planner

### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | String | Nombre de la organización de eventos (requerido) |
| `code` | String | Código único del organizador de eventos (requerido, debe ser único) |

### Campos Opcionales

| Campo | Tipo | Valor por Defecto | Descripción |
|-------|------|-------------------|-------------|
| `logo` | String | - | URL o ruta del logo de la organización |
| `domain` | String | - | Dominio asociado al event planner |
| `contactEmail` | String | - | Email de contacto |
| `employees` | Array\<ObjectId\> | `[]` | Array de IDs de usuarios empleados |
| `aboveFee` | Boolean | `false` | **Por arriba**: Si es `true`, las comisiones (IVA, IEP, etc.) se suman al precio base. Si es `false`, las comisiones se calculan sobre el precio base (por abajo) |
| `aboveFeePaymentGateway` | Boolean | `false` | **Por arriba**: Si es `true`, la comisión del gateway de pago se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeTicketOffice` | Boolean | `false` | **Por arriba**: Si es `true`, la comisión de taquilla se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeIGTF` | Boolean | `false` | **Por arriba**: Si es `true`, la comisión IGTF se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeIGTFTicketOffice` | Boolean | `false` | **Por arriba**: Si es `true`, la comisión IGTF en taquilla se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeB9` | Boolean | `false` | **Por arriba**: Si es `true`, la comisión B9 se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeIVA` | Boolean | `false` | **Por arriba**: Si es `true`, la comisión IVA se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `isWithDiscounts` | Boolean | `false` | Indica si el event planner permite descuentos |
| `aditionalInfo` | Object | `{}` | Información adicional en formato objeto |
| `notIncludeFees` | Boolean | `false` | Indica si no se incluyen comisiones en el precio total del ticket. **Nota**: Cuando es `true`, los tickets se guardan sin las comisiones incluidas en el precio total. En el dashboard y reportes se muestra el precio del ticket sin comisiones. Las comisiones están guardadas en su campo correspondiente pero sin afectar el `totalAmount` del ticket |
| `priceExonerated` | Number | `0` | Precio exonerado |
| `noUpdateLocationsCode` | Boolean | `false` | Indica si no se actualizan los códigos de ubicaciones |
| `withDolarPrice` | Boolean | - | Indica si el event planner usa tasa de cambio independiente (diferente a la tasa general del sistema) |
| `stripeConnectedFee` | Boolean | - | Indica si Stripe está conectado para comisiones |
| `stripeConnectedField` | String | - | Campo de conexión de Stripe |
| `stripeName` | String | - | Nombre en Stripe |
| `defaultTicket` | Boolean | - | Indica si es el ticket por defecto |
| `notInclueSeasonTicketsInTicketsPerProductReport` | Boolean | - | Excluir abonos del reporte de tickets por producto |
| `useAmountAsFeeBase` | Boolean | - | Usar monto como base para comisiones |
| `digitalBilling` | Boolean | - | Habilitar facturación digital para el event planner |
| `enableResetPasswordLink` | Boolean | `false` | Habilitar enlace de restablecimiento de contraseña |
| `casheaAvailable` | Boolean | - | Activa Cashea con los datos del event planner |
| `currency` | String | - | Moneda por defecto (ej: "USD", "VES") |
| `domains` | Array\<String\> | `[]` | Array de dominios asociados |

### Notas sobre la Creación

1. **`adminId`**: Se asigna automáticamente durante la creación cuando se crea a través de `add-sub-user`. No debe enviarse manualmente.

2. **`code`**: Debe ser único en el sistema. El sistema valida que no exista otro event planner con el mismo código.

3. **`hidden`**: Se puede establecer durante la creación, pero no se puede modificar después.

### Notas sobre las Comisiones (aboveFee*)

Los campos `aboveFee*` determinan **cómo se calculan las comisiones**:

- **`true` (Por arriba)**: Las comisiones se **suman al precio base**. 
  - Ejemplo: Precio base $100, comisión 10% → Precio final = $100 + $10 = **$110**
  
- **`false` (Por abajo)**: Las comisiones se **calculan sobre el precio base** (incluidas en el precio).
  - Ejemplo: Precio base $100, comisión 10% → Precio final = **$100** (la comisión está incluida en los $100)

**Importante**: Cada tipo de comisión (IVA, IGTF, B9, gateway de pago, taquilla) puede configurarse independientemente.

### Ejemplo de Creación

```javascript
// A través del caso de uso add-sub-user
const eventPlannerData = {
  name: "Mi Organización de Eventos",
  code: "ORG001",
  logo: "https://example.com/logo.png",
  domain: "mieventos.com",
  contactEmail: "contacto@mieventos.com",
  currency: "USD",
  withDolarPrice: true,
  digitalBilling: true,
  domains: ["www.mieventos.com", "app.mieventos.com"]
}

// El adminId se asigna automáticamente al crear el usuario asociado
```

---

## Edición de Event Planner

### Campos que NO se pueden modificar

Los siguientes campos están **protegidos** y **no pueden ser modificados** después de la creación:

| Campo | Razón |
|-------|-------|
| `adminId` | ID del administrador principal, asignado en la creación |
| `code` | Código único del event planner, no modificable |
| `hidden` | Estado de visibilidad, protegido |
| `isWithDiscounts` | Configuración de descuentos, protegida |
| `aditionalInfo` | Información adicional, protegida |
| `noUpdateLocationsCode` | Configuración de códigos de ubicación, protegida |

### Campos que SÍ se pueden modificar

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | String | Nombre de la organización |
| `logo` | String | URL o ruta del logo |
| `domain` | String | Dominio asociado |
| `contactEmail` | String | Email de contacto |
| `employees` | Array\<ObjectId\> | Array de IDs de empleados |
| `aboveFee` | Boolean | **Por arriba**: Si es `true`, las comisiones (IVA, IEP, etc.) se suman al precio base. Si es `false`, se calculan sobre el precio base (por abajo) |
| `aboveFeePaymentGateway` | Boolean | **Por arriba**: Si es `true`, la comisión del gateway de pago se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeTicketOffice` | Boolean | **Por arriba**: Si es `true`, la comisión de taquilla se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeIGTF` | Boolean | **Por arriba**: Si es `true`, la comisión IGTF se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeIGTFTicketOffice` | Boolean | **Por arriba**: Si es `true`, la comisión IGTF en taquilla se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeB9` | Boolean | **Por arriba**: Si es `true`, la comisión B9 se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `aboveFeeIVA` | Boolean | **Por arriba**: Si es `true`, la comisión IVA se suma al precio. Si es `false`, se calcula sobre el precio base (por abajo) |
| `notIncludeFees` | Boolean | Indica si no se incluyen comisiones en el precio total del ticket. **Nota**: Cuando es `true`, los tickets se guardan sin las comisiones incluidas en el precio total. En el dashboard y reportes se muestra el precio del ticket sin comisiones. Las comisiones están guardadas en su campo correspondiente pero sin afectar el `totalAmount` del ticket |
| `priceExonerated` | Number | Precio exonerado |
| `withDolarPrice` | Boolean | Indica si el event planner usa tasa de cambio independiente (diferente a la tasa general del sistema) |
| `stripeConnectedFee` | Boolean | Stripe conectado para comisiones |
| `stripeConnectedField` | String | Campo de conexión Stripe |
| `stripeName` | String | Nombre en Stripe |
| `defaultTicket` | Boolean | Ticket por defecto |
| `notInclueSeasonTicketsInTicketsPerProductReport` | Boolean | Excluir abonos del reporte |
| `useAmountAsFeeBase` | Boolean | Usar monto como base para comisiones |
| `digitalBilling` | Boolean | Habilitar facturación digital para el event planner |
| `enableResetPasswordLink` | Boolean | Enlace de restablecimiento de contraseña |
| `casheaAvailable` | Boolean | Activa Cashea con los datos del event planner |
| `currency` | String | Moneda por defecto |
| `domains` | Array\<String\> | Array de dominios |

### Ejemplo de Edición

```javascript
import { eventPlannerServices } from "../../uses-cases"

const resultado = await eventPlannerServices.updateEventPlanner({
  user: currentUser, // Usuario que realiza la actualización
  eventPlanner: {
    _id: "507f1f77bcf86cd799439011" // ID del event planner a actualizar
  },
  // Campos a actualizar (los campos protegidos serán ignorados automáticamente)
  name: "Nuevo Nombre de la Organización",
  contactEmail: "nuevo@email.com",
  logo: "https://example.com/nuevo-logo.png",
  currency: "VES",
  digitalBilling: true,
  domains: ["nuevo-dominio.com"]
})
```

### Respuesta de Edición

```json
{
  "message": "Organizador de eventos actualizado exitosamente.",
  "doc": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Nuevo Nombre de la Organización",
    "code": "ORG001",
    "contactEmail": "nuevo@email.com",
    // ... resto de campos actualizados
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

## Validaciones

### Creación

1. **`name`**: Debe estar presente y no puede estar vacío
2. **`code`**: Debe estar presente, no puede estar vacío y debe ser único en el sistema
3. **`adminId`**: Se asigna automáticamente, no debe enviarse

### Edición

1. **`_id`**: Debe estar presente y el event planner debe existir
2. **Campos protegidos**: Se ignoran automáticamente si se intentan modificar
3. **Al menos un campo válido**: Debe haber al menos un campo modificable para actualizar

---

## Logs de Auditoría

### Edición

Cuando se actualiza un event planner, se crea automáticamente un registro en `reservationLogModel`:

```javascript
{
  admin: ObjectId("..."), // Usuario que realizó la actualización
  type: "event_planner_updated",
  eventPlanner: ObjectId("..."), // ID del event planner actualizado
  description: "Eventplanner actualizado: {...}" // JSON con los campos modificados
}
```

---

## Errores Comunes

### Creación

- **"Nombre de la organizacion de eventos es requerido"**: Falta el campo `name`
- **"Codigo del organizador evento es requerido"**: Falta el campo `code`
- **"El codigo se encuentra registrado"**: El código ya existe en el sistema

### Edición

- **"Eventplanner no encontrado"**: El ID proporcionado no existe (404)
- **"No hay campos válidos para actualizar"**: Solo se intentaron modificar campos protegidos (400)
- **"Error al actualizar el organizador de eventos"**: Error interno del servidor (500)

---

## Resumen de Campos

### Comparación: Creación vs Edición

| Campo | Creación | Edición | Notas |
|-------|----------|---------|-------|
| `name` | ✅ Requerido | ✅ Modificable | - |
| `code` | ✅ Requerido | ❌ Protegido | Debe ser único |
| `adminId` | ⚠️ Auto-asignado | ❌ Protegido | No enviar manualmente |
| `logo` | ✅ Opcional | ✅ Modificable | - |
| `domain` | ✅ Opcional | ✅ Modificable | - |
| `contactEmail` | ✅ Opcional | ✅ Modificable | - |
| `employees` | ✅ Opcional | ✅ Modificable | Array de ObjectIds |
| `hidden` | ✅ Opcional | ❌ Protegido | - |
| `isWithDiscounts` | ✅ Opcional | ❌ Protegido | - |
| `aditionalInfo` | ✅ Opcional | ❌ Protegido | - |
| `noUpdateLocationsCode` | ✅ Opcional | ❌ Protegido | - |
| `currency` | ✅ Opcional | ✅ Modificable | - |
| `domains` | ✅ Opcional | ✅ Modificable | Array de strings |
| Todos los campos `aboveFee*` | ✅ Opcional | ✅ Modificable | - |
| Campos Stripe | ✅ Opcional | ✅ Modificable | - |
| Otros campos booleanos | ✅ Opcional | ✅ Modificable | - |

---

## Ubicación del Código

- **Modelo**: `src/models/event-planner-model.js`
- **Entidad de validación**: `src/entities/event-planner.js`
- **Caso de uso de edición**: `src/uses-cases/event-planner/update-event-planner.js`
- **Caso de uso de creación**: `src/uses-cases/users/add-sub-user.js` (cuando `userRoll === "event_planner"`)
- **Índice de servicios**: `src/uses-cases/event-planner/index.js`

