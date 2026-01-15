# Documentación: Creación de Impuestos (Taxes)

## Descripción

Esta documentación explica cómo crear diferentes tipos de impuestos en el sistema. Los impuestos se utilizan para calcular cargos adicionales sobre las reservas, como IVA, IEP, comisiones de pasarelas de pago, service charges, entre otros.

## Paso 1: Obtener Configuraciones

Antes de crear cualquier impuesto, es necesario consumir el endpoint de configuraciones para obtener los tipos de impuestos disponibles.

### Endpoint

```
GET /api/v1/taxes/configs
```

### Respuesta

El endpoint devuelve un objeto con tres propiedades:

```json
{
  "taxesType": {
    "name": "taxesType",
    "type": "taxesType",
    "elements": [
      "system",
      "b9Fee",
      "paymentGateway",
      "ticketOffice"
    ]
  },
  "ticketType": {
    "name": "ticketType",
    "type": "ticketType",
    "elements": [
      "iva",
      "iep",
      "igtf"
    ]
  },
  "paymentGateway": {
    "name": "paymentGateway",
    "type": "paymentGateway",
    "elements": [
      "STRIP",
      "BINANCE",
      "TRANSFER",
      // ... otros métodos de pago
    ]
  }
}
```

### Uso

Con las configuraciones obtenidas, se deben listar los `taxesType` disponibles para que el usuario pueda seleccionar el tipo de impuesto que desea crear.

## Tipos de Impuestos

### 1. Impuestos Tipo `system`

Los impuestos de tipo `system` sirven para crear impuestos simples que se guardan dentro del campo `fee` del precio de la reserva.

**Uso común:**
- IVA (Impuesto al Valor Agregado)
- IEP (Impuesto eventos publicos)

**Configuración:**
- `type`: `"system"`
- `ticketType`: Debe ser `"iva"` o `"iep"` según el tipo de impuesto que se desea crear
- `percentage`: Porcentaje del impuesto
- `name`: Nombre descriptivo del impuesto
- `active`: `true` para activar el impuesto

**Ejemplo:**

```json
{
  "taxes": [
    {
      "name": "IVA",
      "type": "system",
      "ticketType": "iva",
      "percentage": 16,
      "active": true
    },
    {
      "name": "IEP",
      "type": "system",
      "ticketType": "iep",
      "percentage": 8,
      "active": true
    }
  ]
}
```

**Nota:** 
- Los impuestos de tipo `system` se guardan en el campo `fee` del desglose de precio de la reserva.
- En el caso del **IVA** (`ticketType: "iva"`), el valor se guarda específicamente en `price.taxIVA` (en VES) y `price.taxIVAUSD` (en USD).
- En el caso del **IEP** (`ticketType: "iep"`), el valor se guarda específicamente en `price.taxIEP` (en VES) y `price.taxIEPUSD` (en USD).
- Ambos valores también se suman al campo `fee` general del precio.
---

### 2. Impuestos Tipo `b9Fee`

Los impuestos de tipo `b9Fee` se utilizan para crear un service charge (cargo por servicio). Este tipo de impuesto tiene su propio campo específico dentro del desglose de precio.

**Uso común:**
- Service Charge
- Cargo por servicio Back9

**Configuración:**
- `type`: `"b9Fee"`
- `percentage` o `amount`: Valor del cargo (porcentaje o monto fijo)
- `name`: Nombre descriptivo del cargo
- `active`: `true` para activar el cargo

**Ejemplo:**

```json
{
  "taxes": [
    {
      "name": "Service Charge 5%",
      "type": "b9Fee",
      "percentage": 5,
      "active": true
    }
  ]
}
```

**Nota:** Este impuesto se guarda en el campo `price.b9Fee` del desglose de precio de la reserva. Es el tipo más recomendable para crear un service charge.

---

### 3. Impuestos Tipo `paymentGateway`

Los impuestos de tipo `paymentGateway` se utilizan para crear comisiones de pasarelas de pago. Estos impuestos se aplican principalmente en la web.

**Uso común:**
- Comisiones de Stripe
- Comisiones de pago movil
- Comisiones de transferencias bancarias
- Otras pasarelas de pago

**Configuración:**
- `type`: `"paymentGateway"`
- `method`: Debe ser uno de los valores disponibles en `config.paymentGateway.elements` (por ejemplo: `"STRIP"`, `"BAPM"`, `"TRANSFER"`)
- `percentage`: Valor de la comisión (porcentaje o monto fijo)
- `name`: Nombre descriptivo de la comisión
- `active`: `true` para activar la comisión

**Ejemplo:**

```json
{
  "taxes": [
    {
      "name": "Comisión Stripe 3%",
      "type": "paymentGateway",
      "method": "STRIP",
      "percentage": 3,
      "active": true,
      "ticketType": null
    },
    {
      "name": "Comisión pago movil 2.5%",
      "type": "paymentGateway",
      "method": "BAPM",
      "percentage": 2.5,
      "active": true,
      "ticketType": null
    }
  ]
}
```

**Nota:** 
- Estos impuestos se guardan en el campo `price.feePaymentGateWay` del desglose de precio.
- Se utilizan principalmente para la web.
- Para renderizar el listado de pasarelas disponibles, se debe usar `config.paymentGateway.elements`.

---

### 4. Impuestos Tipo `ticketOffice`

Los impuestos de tipo `ticketOffice` sirven para crear impuestos que se cobran específicamente en la taquilla.

**Uso común:**
- IVA en taquilla
- IEP en taquilla
- Otros impuestos aplicables solo en taquilla

**Configuración:**
- `type`: `"ticketOffice"`
- `ticketType`: Opcional, puede ser `"iva"` o `"iep"` si se desea aplicar un impuesto específico
- `percentage`Valor del impuesto (porcentaje)
- `name`: Nombre descriptivo del impuesto
- `active`: `true` para activar el impuesto

**Ejemplo:**

```json
{
  "taxes": [
    {
      "name": "IVA Taquilla 16%",
      "type": "ticketOffice",
      "ticketType": "iva",
      "percentage": 16,
      "active": true
    }
  ]
}
```

**Nota:** Estos impuestos se aplican únicamente cuando las reservas se realizan desde la taquilla (ticket office).

---

### 5. Impuesto IGTF (Impuesto a las Grandes Transacciones Financieras)

El IGTF es un impuesto especial que se cobra sobre transacciones financieras. Para crear un impuesto IGTF, se debe:

1. Seleccionar el tipo `paymentGateway`
2. Agregar el `ticketType` con valor `"igtf"`
3. Especificar la pasarela de pago en el campo `method`

**Configuración:**
- `type`: `"paymentGateway"`
- `ticketType`: `"igtf"`
- `method`: Debe ser uno de los valores disponibles en `config.paymentGateway.elements` (la pasarela de pago sobre la cual se cobrará el IGTF)
- `percentage` o `amount`: Valor del IGTF (porcentaje o monto fijo)
- `name`: Nombre descriptivo del impuesto
- `active`: `true` para activar el impuesto

**Ejemplo:**

```json
{
  "taxes": [
    {
      "name": "IGTF Stripe 3%",
      "type": "paymentGateway",
      "ticketType": "igtf",
      "method": "STRIP",
      "percentage": 3,
      "active": true
    },
    {
      "name": "IGTF Transferencia 2%",
      "type": "paymentGateway",
      "ticketType": "igtf",
      "method": "TRANSFER",
      "percentage": 2,
      "active": true
    }
  ]
}
```

**Nota:** 
- Estos impuestos se guardan en el campo `igtfFee` del desglose de precio.
- Se cobran tanto en web como en taquilla.
- Se aplica sobre la pasarela de pago especificada en el campo `method`.

---

## Endpoint para Crear/Actualizar Impuestos

### Endpoint

```
POST /api/v1/taxes
```

### Autenticación

Requiere autenticación. El usuario debe pertenecer al grupo de Event Planner.

### Parámetros de Entrada

#### Body (JSON)

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `taxes` | Array\<Object\> | Sí | Array de objetos de impuestos a crear o actualizar |

#### Estructura del Objeto Tax

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `_id` | String (ObjectId) | No | ID del impuesto. Si se proporciona, se actualiza el impuesto existente |
| `name` | String | Sí | Nombre descriptivo del impuesto |
| `type` | String | Sí | Tipo de impuesto: `"system"`, `"b9Fee"`, `"paymentGateway"`, `"ticketOffice"` |
| `ticketType` | String | Condicional | Tipo de ticket: `"iva"`, `"iep"`, `"igtf"`. Requerido para `system` con IVA/IEP, y para IGTF |
| `method` | String | Condicional | Método de pago (de `config.paymentGateway.elements`). Requerido para `paymentGateway` |
| `percentage` | Number | Condicional | Porcentaje del impuesto (si se usa porcentaje) |
| `amount` | Number | Condicional | Monto fijo del impuesto (si se usa monto fijo) |
| `active` | Boolean | No | Estado del impuesto (default: `true`) |

**Nota:** Se debe proporcionar `percentage` o `amount`, pero no ambos.

### Ejemplo de Request Completo

```json
{
  "taxes": [
    {
      "name": "IVA 16%",
      "type": "system",
      "ticketType": "iva",
      "percentage": 16,
      "active": true
    },
    {
      "name": "Service Charge 5%",
      "type": "b9Fee",
      "percentage": 5,
      "active": true
    },
    {
      "name": "Comisión Stripe 3%",
      "type": "paymentGateway",
      "method": "STRIP",
      "percentage": 3,
      "active": true,
      "ticketType": null
    },
    {
      "name": "IGTF Stripe 3%",
      "type": "paymentGateway",
      "ticketType": "igtf",
      "method": "STRIP",
      "percentage": 3,
      "active": true
    },
    {
      "name": "IVA Taquilla 16%",
      "type": "ticketOffice",
      "ticketType": "iva",
      "percentage": 16,
      "active": true
    }
  ]
}
```

### Ejemplo de Actualización

Para actualizar un impuesto existente, incluir el `_id`:

```json
{
  "taxes": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "IVA 18%",
      "type": "system",
      "ticketType": "iva",
      "percentage": 18,
      "active": true
    }
  ]
}
```

### Respuesta

```json
{
  "message": "Okey"
}
```

---

## Resumen de Campos en el Desglose de Precio

Los diferentes tipos de impuestos se guardan en diferentes campos del objeto `price` de la reserva:

| Tipo de Impuesto | Campo en `price` | Descripción |
|------------------|------------------|-------------|
| `system` (IVA) | `taxIVA`, `taxIVAUSD`, `fee` | IVA se guarda en `taxIVA` (VES) y `taxIVAUSD` (USD), también se suma a `fee` |
| `system` (IEP) | `taxIEP`, `taxIEPUSD`, `fee` | IEP se guarda en `taxIEP` (VES) y `taxIEPUSD` (USD), también se suma a `fee` |
| `b9Fee` | `b9Fee` | Campo específico para service charge |
| `paymentGateway` (sin IGTF) | `feePaymentGateWay` | Campo específico para comisiones de pasarelas |
| `paymentGateway` (IGTF) | `igtfFee` | Campo específico para IGTF |
| `ticketOffice` | `fee` | Se suma al campo `fee` general (solo en taquilla) |

---

## Flujo Recomendado

1. **Obtener configuraciones**: Consumir `GET /api/v1/taxes/configs` para obtener los tipos disponibles
2. **Listar tipos de impuestos**: Mostrar `config.taxesType.elements` al usuario
3. **Seleccionar tipo**: El usuario selecciona el tipo de impuesto que desea crear
4. **Configurar según tipo**:
   - Si es `system`: Solicitar `ticketType` (iva/iep)
   - Si es `b9Fee`: No requiere campos adicionales
   - Si es `paymentGateway`: Mostrar `config.paymentGateway.elements` para seleccionar `method`
   - Si es `ticketOffice`: Opcionalmente solicitar `ticketType`
   - Si es IGTF: Seleccionar `paymentGateway` + `ticketType: "igtf"` + `method`
5. **Solicitar valores**: Pedir `percentage` o `amount` y `name`
6. **Crear impuesto**: Enviar `POST /api/v1/taxes` con la configuración

---

## Notas Importantes

- Los impuestos se aplican automáticamente al calcular el precio de las reservas
- El campo `active` permite activar/desactivar impuestos sin eliminarlos
- Los impuestos de tipo `paymentGateway` sin `ticketType` se aplican solo en web
- Los impuestos IGTF (`paymentGateway` con `ticketType: "igtf"`) se aplican tanto en web como en taquilla
- Los impuestos de tipo `ticketOffice` se aplican únicamente en taquilla
- Para actualizar un impuesto, incluir el `_id` en el objeto del array `taxes`

