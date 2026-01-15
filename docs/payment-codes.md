# Payment Codes - Documentación

Este documento describe todos los códigos de respuesta (`paymentCode`) que pueden retornar los servicios de validación de pagos.

## Códigos de Éxito

### 200 - Pago Encontrado
- **Mensaje**: "Pago encontrado"
- **Reintentar**: ❌ No
- **Acción del Usuario**: Ir a thanks page (pago exitoso)
- **Descripción**: El pago fue validado correctamente y la orden está verificada.

---

## Códigos de Error de Validación

### 500 - Error al Procesar el Pago
- **Mensaje**: "Error al procesar el pago"
- **errorField**: `"bank_system"`
- **Reintentar**: ✅ Sí
- **Acción del Usuario**: Reintentar el pago
- **Descripción**: Error en el sistema bancario al procesar la solicitud.

### 601 - Datos de Pago Incorrectos
- **Mensaje**: "Uno o más datos de su pago están errados, por favor intentar de nuevo"
- **errorField**: 
  - `"phone | bank"` (BNCPM)
  - `"phone | bank | date"` (BAPM)
- **Reintentar**: ✅ Sí
- **Acción del Usuario**: Reintentar el pago
- **Descripción**: No se encontraron pagos con los datos proporcionados (teléfono, banco o fecha).

### 602 - Referencia No Encontrada
- **Mensaje**: "No encontramos su referencia, por favor intente de nuevo"
- **errorField**: `"ref"`
- **Reintentar**: ✅ Sí
- **Acción del Usuario**: Reintentar el pago
- **Descripción**: La referencia de pago proporcionada no coincide con ningún pago encontrado.

### 603 - Monto No Coincide
- **Mensaje**: "El monto del pago no coincide con el monto de la order"
- **errorField**: `"amount"`
- **Campos Adicionales**:
  - `amount`: Monto del pago encontrado
  - `remainingAmount`: Monto restante (si el pago es menor)
  - `paymentAmount`: Monto del pago
- **Reintentar**: ❌ No
- **Acción del Usuario**: Ir al flujo de multipago
- **Descripción**: El monto del pago no coincide con el monto de la orden. El usuario debe completar el pago restante a través del flujo de multipago.

### 604 - Fecha Incorrecta (Solo BNCPM)
- **Mensaje**: "Uno o más datos de su pago están errados, por favor intentar de nuevo"
- **errorField**: `"date"`
- **Reintentar**: ✅ Sí
- **Acción del Usuario**: Reintentar el pago
- **Descripción**: No se encontraron pagos en la fecha especificada. Solo aplica para pagos BNCPM.

### 606 - Monto Pendiente de Completar
- **Mensaje**: "Debe completar el monto de la order: {remainingAmount}"
- **Campos Adicionales**:
  - `remainingAmount`: Monto pendiente de la orden
- **Reintentar**: ❌ No
- **Acción del Usuario**: Ir al flujo de multipago
- **Descripción**: La orden tiene un monto pendiente que debe ser completado. El usuario debe ir al flujo de multipago para completar el pago.

---

## Resumen de Flujos

| Código | Reintentar | Acción del Usuario |
|--------|-----------|-------------------|
| **200** | ❌ No | Ir a thanks page (pago exitoso) |
| **500** | ✅ Sí | Reintentar pago |
| **601** | ✅ Sí | Reintentar pago |
| **602** | ✅ Sí | Reintentar pago |
| **603** | ❌ No | Ir al flujo de multipago |
| **604** | ✅ Sí | Reintentar pago |
| **606** | ❌ No | Ir al flujo de multipago |

---

## Códigos que Redirigen a Multipago

Los siguientes códigos indican que el usuario debe ser redirigido al flujo de multipago:

- **603** - Monto no coincide
- **606** - Monto pendiente de completar

---

## Códigos que Permiten Reintentar

Los siguientes códigos permiten al usuario reintentar el pago:

- **500** - Error al procesar el pago
- **601** - Datos de pago incorrectos
- **602** - Referencia no encontrada
- **604** - Fecha incorrecta

---

## Archivos Relacionados

- `src/uses-cases/bnc-pm/payment-bnc-pm.js` - Implementación BNCPM
- `src/uses-cases/ba-pm/payment-banca-amiga-pm.js` - Implementación BAPM
- `src/uses-cases/orders/validation-order.js` - Validación de órdenes
- `src/uses-cases/orders/pay-remaining-amount.js` - Pago de monto restante

---

## Notas de Implementación

- El código **601** tiene un retry automático con delay de 15 segundos en `validation-order.js`
- Los códigos **603** y **606** requieren manejo especial para el flujo de multipago
- El código **200** debe actualizar el estado de la orden a "approved" y emitir las entidades correspondientes

