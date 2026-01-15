# Pay Remaining Amount - Flujo Completo con Endpoints

## Descripción

Este documento describe el flujo completo para procesar el pago del monto restante (`remainingAmount`) de una orden, incluyendo los endpoints HTTP, estructura de peticiones, respuestas y el flujo de validación posterior.

---

## Endpoints

### 1. Pagar Monto Restante

**Endpoint**: `POST /api/v1/orders/pay-remaining-amount`

**Ubicación del código**:
- Endpoint: `src/index.js` (línea 1251)
- Controlador: `src/controllers/orders-controller/index.js`
- Caso de uso: `src/uses-cases/orders/pay-remaining-amount.js`

### 2. Validar Orden (para órdenes pendientes)

**Endpoint**: `POST /api/v1/orders/validation`

**Ubicación del código**:
- Endpoint: `src/index.js` (línea 1250)
- Controlador: `src/controllers/orders-controller/index.js`
- Caso de uso: `src/uses-cases/orders/validation-order.js`

---

## Flujo Completo

### Paso 1: Pagar Monto Restante

#### Request

```http
POST /api/v1/orders/pay-remaining-amount
Content-Type: application/json

{
  "orderId": "507f1f77bcf86cd799439011",
  "payment": {
    "ref": "REF123456",
    "date": "2025-01-15",
    "bank": "0172",
    "phone": "584243252326",
    "type": "BNCPM"
  }
}
```

#### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `orderId` | String | ID de la orden que tiene `remainingAmount` |
| `payment.ref` | String | Referencia del pago |
| `payment.date` | String | Fecha del pago (formato: YYYY-MM-DD) |
| `payment.bank` | String | Código del banco |
| `payment.phone` | String | Teléfono del pagador |
| `payment.type` | String | Tipo de pago: `"BNCPM"` o `"BAPM"` |

#### Validaciones

1. La orden debe existir
2. La orden debe tener un `remainingAmount` > 0
3. Todos los campos de `payment` son requeridos

#### Respuestas

##### Caso A: Pago Encontrado (Status 200)

```json
{
  "order": {
    "_id": "507f1f77bcf86cd799439012",
    "amount": 5000.00,
    "status": "approved",
    "payment": {
      "ref": "REF123456",
      "date": "2025-01-15",
      "bank": "0172",
      "phone": "584243252326",
      "type": "BNCPM"
    },
    "orderCode": "ABC123XYZ",
    "date": "2025-01-15T10:30:00.000Z",
    "eventPlanner": "507f1f77bcf86cd799439013",
    "user": "507f1f77bcf86cd799439014",
    "currency": "VES"
  },
  "message": "Pago Aprobado",
  "status": "approved",
  "validationToken": null
}
```

**Acción del Usuario**: ✅ Ir a thanks page (pago exitoso)

##### Caso B: Pago No Encontrado (Status 200)

```json
{
  "order": {
    "_id": "507f1f77bcf86cd799439012",
    "amount": 5000.00,
    "status": "pending",
    "payment": {
      "ref": "REF123456",
      "date": "2025-01-15",
      "bank": "0172",
      "phone": "584243252326",
      "type": "BNCPM"
    },
    "validationToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "validationAttempts": 1,
    "orderCode": "ABC123XYZ",
    "date": "2025-01-15T10:30:00.000Z",
    "eventPlanner": "507f1f77bcf86cd799439013",
    "user": "507f1f77bcf86cd799439014",
    "currency": "VES"
  },
  "message": "Pago en proceso",
  "status": "pending",
  "validationToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Acción del Usuario**: ⚠️ Continuar al **Paso 2** para validar la orden

##### Caso C: Error (Status 400)

```json
{
  "error": {
    "message": "La orden no tiene monto pendiente de pago"
  }
}
```

**Otros errores posibles**:
- `"Orden no encontrada"`
- `"Campos requeridos faltantes: ref, date, bank, phone"`
- `"Servicio de pago no disponible para tipo: {type}"`

---

### Paso 2: Validar Orden Pendiente

**⚠️ IMPORTANTE**: Este paso solo se ejecuta si en el Paso 1 la respuesta tiene `status: "pending"`.

#### Request

```http
POST /api/v1/orders/validation
Content-Type: application/json

{
  "order": "507f1f77bcf86cd799439012",
  "validationToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "ref": "REF123456",
  "date": "2025-01-15",
  "bank": "0172",
  "phone": "584243252326"
}
```

#### Campos Requeridos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `order` | String | ID de la orden pendiente (retornada en Paso 1) |
| `validationToken` | String | Token de validación (retornado en Paso 1) |
| `ref` | String | (Opcional en intentos 1-2, requerido en 3-4) Referencia del pago |
| `date` | String | (Opcional en intentos 1-2, requerido en 3-4) Fecha del pago |
| `bank` | String | (Opcional en intentos 1-2, requerido en 3-4) Código del banco |
| `phone` | String | (Opcional en intentos 1-2, requerido en 3-4) Teléfono del pagador |

**Nota**: En los intentos 1 y 2, se usan los datos de la orden. En intentos 3 y 4, se pueden proporcionar nuevos datos de pago.

#### Validaciones

1. La orden debe existir
2. La orden debe estar en estado `"pending"`
3. El `validationToken` debe coincidir
4. Máximo 4 intentos de validación
5. La orden no debe tener `remainingAmount` (si lo tiene, retorna código 606)

#### Respuestas

##### Caso A: Pago Validado Exitosamente (Status 200)

```json
{
  "error": false,
  "payment": {
    "amount": 5000.00,
    "ref": "REF123456",
    "date": "2025-01-15",
    "bank": "0172",
    "phone": "584243252326"
  },
  "paymentCode": 200,
  "paymentMessage": "Pago encontrado",
  "errorField": null,
  "remainingAmount": null
}
```

**Acción del Usuario**: ✅ Ir a thanks page (pago validado)

##### Caso B: Pago No Validado (Status 200)

```json
{
  "error": true,
  "message": "Pago no validado",
  "validationAttempts": 2,
  "paymentCode": 601,
  "paymentMessage": "Uno o más datos de su pago están errados, por favor intentar de nuevo",
  "errorField": "phone | bank",
  "remainingAmount": null
}
```

**Acción del Usuario**: 
- Si `paymentCode` es **500, 601, 602, 604**: ✅ Reintentar el pago (llamar nuevamente al endpoint de validación)
- Si `paymentCode` es **603 o 606**: ❌ Ir al flujo de multipago

**Códigos de pago posibles**:
- `200`: Pago encontrado ✅
- `500`: Error al procesar el pago (reintentar)
- `601`: Datos incorrectos (reintentar)
- `602`: Referencia no encontrada (reintentar)
- `603`: Monto no coincide (ir a multipago)
- `604`: Fecha incorrecta (reintentar)
- `606`: Monto pendiente (ir a multipago)

Ver [payment-codes.md](./payment-codes.md) para más detalles sobre los códigos.

##### Caso C: Error (Status 400)

```json
{
  "error": {
    "message": "Token de validacion invalido"
  }
}
```

**Otros errores posibles**:
- `"Orden no encontrada"`
- `"Orden no esta pendiente de validacion"`
- `"Maximo de intentos de validacion alcanzado"`
- `"Tipo de pago no soportado para validacion"`

---

## Diagrama de Flujo

```
┌─────────────────────────────────────────────────────────────┐
│  Usuario tiene orden con remainingAmount                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  POST /api/v1/orders/pay-remaining-amount                   │
│  { orderId, payment: { ref, date, bank, phone, type } }     │
└──────────────────────┬──────────────────────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
┌──────────────────┐        ┌──────────────────┐
│ Pago Encontrado  │        │ Pago No Encontrado│
│ status: approved │        │ status: pending   │
└────────┬─────────┘        └────────┬─────────┘
         │                           │
         │                           │
         ▼                           ▼
┌──────────────────┐        ┌──────────────────────────────┐
│ Ir a Thanks Page │        │ POST /api/v1/orders/validation│
│   (Flujo Final)  │        │ { order, validationToken }   │
└──────────────────┘        └────────┬─────────────────────┘
                                      │
                        ┌─────────────┴─────────────┐
                        │                           │
                        ▼                           ▼
              ┌──────────────────┐      ┌──────────────────┐
              │ Pago Validado     │      │ Pago No Validado  │
              │ paymentCode: 200  │      │ paymentCode: 5xx  │
              └────────┬──────────┘      └────────┬──────────┘
                       │                          │
                       │                          │
                       ▼                          ▼
              ┌──────────────────┐      ┌──────────────────────┐
              │ Ir a Thanks Page │      │ Reintentar o Multipago│
              │   (Flujo Final)  │      │ según paymentCode     │
              └──────────────────┘      └──────────────────────┘
```

---

## Ejemplo Completo de Integración

### JavaScript/TypeScript

```javascript
// Paso 1: Pagar monto restante
async function payRemainingAmount(orderId, paymentInfo) {
  const response = await fetch('/api/v1/orders/pay-remaining-amount', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      orderId: orderId,
      payment: {
        ref: paymentInfo.ref,
        date: paymentInfo.date,
        bank: paymentInfo.bank,
        phone: paymentInfo.phone,
        type: paymentInfo.type // "BNCPM" o "BAPM"
      }
    })
  })

  const result = await response.json()

  if (response.status === 400) {
    throw new Error(result.error.message)
  }

  // Si el pago está pendiente, validar
  if (result.status === 'pending') {
    return await validateOrder(result.order._id, result.validationToken, paymentInfo)
  }

  // Pago aprobado
  return result
}

// Paso 2: Validar orden pendiente
async function validateOrder(orderId, validationToken, paymentInfo = null) {
  const body = {
    order: orderId,
    validationToken: validationToken
  }

  // En intentos 3 y 4, se pueden enviar nuevos datos de pago
  if (paymentInfo) {
    body.ref = paymentInfo.ref
    body.date = paymentInfo.date
    body.bank = paymentInfo.bank
    body.phone = paymentInfo.phone
  }

  const response = await fetch('/api/v1/orders/validation', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(body)
  })

  const result = await response.json()

  if (response.status === 400) {
    throw new Error(result.error.message)
  }

  // Si el pago se validó exitosamente
  if (result.paymentCode === 200) {
    return result
  }

  // Si debe ir a multipago
  if (result.paymentCode === 603 || result.paymentCode === 606) {
    // Redirigir al flujo de multipago
    window.location.href = `/multipago?orderId=${orderId}&remainingAmount=${result.remainingAmount}`
    return result
  }

  // Si puede reintentar (500, 601, 602, 604)
  if ([500, 601, 602, 604].includes(result.paymentCode)) {
    // Mostrar mensaje y permitir reintentar
    console.log('Pago no validado, puede reintentar:', result.paymentMessage)
    return result
  }

  return result
}

// Uso
try {
  const result = await payRemainingAmount('order123', {
    ref: 'REF123456',
    date: '2025-01-15',
    bank: '0172',
    phone: '584243252326',
    type: 'BNCPM'
  })

  if (result.status === 'approved' || result.paymentCode === 200) {
    // Redirigir a thanks page
    window.location.href = '/thanks'
  }
} catch (error) {
  console.error('Error:', error.message)
}
```

---

## Comportamientos Especiales

### Retry Automático (Código 601)

Tanto en `pay-remaining-amount` como en `validation-order`, cuando el código de respuesta es **601**, el sistema realiza automáticamente un retry después de 15 segundos antes de retornar la respuesta.

### Límite de Intentos de Validación

- Máximo **4 intentos** de validación por orden
- En intentos 1 y 2: se usan los datos de pago de la orden
- En intentos 3 y 4: se pueden proporcionar nuevos datos de pago

### Códigos que Redirigen a Multipago

Los siguientes códigos indican que el usuario debe ser redirigido al flujo de multipago:

- **603**: Monto no coincide
- **606**: Monto pendiente de completar

### Códigos que Permiten Reintentar

Los siguientes códigos permiten al usuario reintentar el pago:

- **500**: Error al procesar el pago
- **601**: Datos de pago incorrectos
- **602**: Referencia no encontrada
- **604**: Fecha incorrecta

---

## Notas Importantes

1. **Nueva Orden**: Siempre se crea una nueva orden para el monto restante, no se modifica la orden original
2. **Pagos Adicionales**: Si el pago se encuentra, se registra como `aditionalsPayments` en la entidad correspondiente (reservation/installment/purchaseOrder)
3. **Emisión de Entidades**: Solo se emiten las entidades de la orden original cuando el pago restante se encuentra y aprueba
4. **Token de Validación**: Solo se genera si el pago no se encuentra inmediatamente en el Paso 1
5. **Estado de la Orden**: La nueva orden puede quedar en `"approved"` (pago encontrado) o `"pending"` (pago no encontrado)

---

## Referencias

- [Payment Codes Documentation](./payment-codes.md) - Documentación completa de códigos de pago
- `src/uses-cases/orders/pay-remaining-amount.js` - Implementación del caso de uso
- `src/uses-cases/orders/validation-order.js` - Implementación de validación de órdenes
- `src/controllers/orders-controller/index.js` - Controladores
- `src/index.js` - Definición de endpoints
