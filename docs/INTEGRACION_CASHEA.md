# Guía de Integración de Cashea

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Flujo de Integración Completo](#flujo-de-integración-completo)
3. [Instalación y Configuración](#instalación-y-configuración)
4. [Endpoints de API Backend](#endpoints-de-api-backend)
5. [Integración del SDK](#integración-del-sdk)
6. [Ejemplos de Código](#ejemplos-de-código)
7. [Estructura de Datos](#estructura-de-datos)
8. [Mejores Prácticas](#mejores-prácticas)
9. [Troubleshooting](#troubleshooting)

---

## Introducción

### ¿Qué es Cashea?

Cashea es una solución de pago que permite a los comercios integrar un checkout seguro y fácil de usar. El SDK de Cashea genera dinámicamente un botón de pago que maneja todo el flujo de transacción.

### Requisitos Previos

- Node.js 18+ y npm
- Framework React/Next.js (o similar)
- Acceso a la API backend de tu sistema
- Credenciales de Cashea (publicKey)

### Información de Contacto

Para obtener tus credenciales de Cashea o soporte técnico, contacta al equipo:
- Email: soporte@cashea.com

---

## Flujo de Integración Completo

### Diagrama de Flujo

```
┌─────────────────────────────────┐
│  Página de Checkout Inicial     │
│  - Muestra items seleccionados  │
│  - Botón "Pagar con Cashea"     │
└──────────────┬──────────────────┘
               │
               │ Usuario hace clic
               ▼
┌─────────────────────────────────┐
│  Llamada a API: Reprice         │
│  POST /api/v1/reservations/     │
│      reprice/{code}/CASHEA      │
└──────────────┬──────────────────┘
               │
               │ Precios actualizados
               ▼
┌─────────────────────────────────┐
│  Navegación a Checkout Cashea   │
│  Opción A: Nueva pestaña         │
│  Opción B: Misma pestaña + boton Atrás │
└──────────────┬──────────────────┘
               │
               │
               ▼
┌─────────────────────────────────┐
│  Página de Checkout Cashea       │
│  - Obtiene casheaInfo de API    │
│  - Inicializa SDK               │
│  - SDK genera botón dinámicamente│
└──────────────┬──────────────────┘
               │
               │ Usuario completa pago en Cashea
               │ (SDK maneja el proceso)
               ▼
┌─────────────────────────────────┐
│  Redirección a Pago Inicial     │
│  (Si requiere pago inicial)     │
└──────────────┬──────────────────┘
               │
               │
               ▼
┌─────────────────────────────────┐
│  Página Checkout Inicial        │
│  /cashea-checkout-inicial/      │
│  {code}/{type}?idNumber={id}    │
│  - Obtiene info de orden        │
│  - Muestra montos a pagar       │
│  - Procesa pago inicial          │
└──────────────┬──────────────────┘
               │
               │ Usuario completa pago inicial
               ▼
┌─────────────────────────────────┐
│  Procesamiento del Pago inicial |
│  (Manejado por Back9)   │
└─────────────────────────────────┘
```

### Pasos Detallados

#### 1. Agregar Botón Cashea al Checkout Inicial

En tu página de selección de métodos de pago, debes agregar un botón que diga "Pagar con Cashea". Este botón debe estar visible junto a otros métodos de pago disponibles.

**Ejemplo visual:**
```
┌─────────────────────────────┐
│  💳 Pago Móvil              │
│  ⚡ Pagar con Cashea        │  ← Este botón
│  💰 Otro método             │
└─────────────────────────────┘
```

#### 2. Proceso de Reprice

Cuando el usuario hace clic en "Pagar con Cashea", **antes de navegar** a la página de checkout, debes hacer un llamado al endpoint de reprice con el método de pago `CASHEA` en la URL. Este endpoint actualiza los precios según el método de pago seleccionado.

**Importante:** Este paso es necesario para que los precios se ajusten correctamente según el método de pago elegido.

**Endpoints según el tipo:**

- **Para Reservation:** `POST /api/v1/reservations/reprice/{reservationCode}/CASHEA`
- **Para PurchaseOrder:** `POST /api/v1/purchaseOrder/reprice/{purchaseOrder}/CASHEA`

Debes usar el endpoint correspondiente según si estás trabajando con un código de reserva o un purchaseOrder.

#### 3. Navegación a Página de Checkout de Cashea

Tienes dos opciones para navegar a la página de checkout de Cashea:

**Opción A: Abrir en Nueva Pestaña**
- Usa `window.open(url, '_blank')`
- El usuario puede mantener abierta la página original
- Útil si quieres que el usuario pueda comparar métodos de pago

**Opción B: Navegación en la Misma Pestaña con Botón Atrás**
- Usa `router.push(url)` (Next.js) o similar
- Agrega un botón "Atrás" en la página de checkout de Cashea
- Permite al usuario volver fácilmente para seleccionar otro método de pago

#### 4. Obtención de Información desde API Backend

En la página de checkout de Cashea, debes llamar al endpoint que retorna la información necesaria para inicializar el SDK:

- `publicKey`: Clave pública de Cashea (obtenida del backend)
- `casheaBody`: Payload completo con la información de la orden

#### 5. Inicialización del SDK de Cashea

**IMPORTANTE:** El botón de Cashea **NO es un botón HTML estático**. Se genera dinámicamente usando el SDK con la información obtenida del backend.

El SDK requiere:
- Un contenedor HTML donde renderizar el botón
- La `publicKey` para autenticación
- El `casheaBody` con la información de la orden

#### 6. Renderizado del Botón de Pago

El SDK crea automáticamente el botón dentro del contenedor HTML especificado. El botón se renderiza con:
- Estilo visual de Cashea
- Funcionalidad completa de pago
- Validaciones y manejo de errores integrado

#### 7. Procesamiento del Pago en Cashea

Una vez que el usuario interactúa con el botón generado por el SDK, todo el flujo de pago es manejado automáticamente por Cashea. El SDK procesa el pago y, si el flujo requiere un pago inicial, redirige al usuario a la página de checkout inicial.

**Importante:** Después de que el usuario complete el pago en Cashea, si la orden requiere un pago inicial, el sistema redirigirá automáticamente a la página de checkout inicial con los parámetros necesarios.

#### 8. Crear Página de Checkout Inicial de Cashea

**Esta página se accede DESPUÉS de que el usuario completa el pago en Cashea**, cuando el sistema requiere un pago inicial adicional.

Debes crear una página especial para el checkout inicial de Cashea con la siguiente estructura de ruta:

**Ruta:** `cashea-checkout-inicial/{code}/{type}`

**Parámetros de Ruta:**
- `{code}` (string): Código de reserva (`reservationCode`) o código de orden de compra (`purchaseOrder`)
- `{type}` (string): Tipo de código, puede ser:
  - `"reservation"` - Si el código es un código de reserva
  - `"purchaseOrder"` - Si el código es un código de orden de compra

**Query Parameters:**
- `idNumber` (string, requerido): ID de identificación de Cashea (número de cédula o identificación del cliente). Este valor se obtiene del proceso de pago en Cashea.

**Ejemplo de URL:**
```
https://tudominio.com/cashea-checkout-inicial/ABC123/reservation?idNumber=V12345678
```

o

```
https://tudominio.com/cashea-checkout-inicial/PO-456/purchaseOrder?idNumber=V12345678
```

**Funcionalidad de esta página:**
- Obtiene el `idNumber` de los query parameters (proporcionado por Cashea después del pago)
- Obtiene información de la orden desde el endpoint `/api/v1/cashea/order` (usando el parámetro correcto según el tipo: `reservationCode` o `purchaseOrder`)
- Muestra montos de pago inicial (entrada USD, entrada VES, monto financiado)
- Permite al usuario seleccionar método de pago para el pago inicial
- Procesa el pago inicial usando el endpoint correspondiente según el método seleccionado

**Nota:** Esta página solo se muestra cuando el flujo de Cashea requiere un pago inicial adicional. El `idNumber` se obtiene automáticamente del proceso de pago en Cashea.

#### 9. Procesar Pago Inicial con BNC (Pago Móvil)

Una vez que el usuario está en la página de checkout inicial y ha seleccionado el método de pago, debes procesar el pago inicial.

**Métodos de pago disponibles para pago inicial:**
- **BNC (Pago Móvil)** - Disponible actualmente
- **Stripe** - Próximamente
- **BAPM** - Próximamente

**Proceso para BNC (Pago Móvil):**

1. El usuario completa el formulario con:
   - Teléfono
   - Banco (código del banco)
   - Referencia (número de referencia del pago)

2. Envía la información al endpoint `/api/v1/bnc/cashea-payment` con:
   - `reservationCode` o `purchaseOrder` (según el tipo)
   - `casheaId` (obtenido del query parameter `idNumber`)
   - Información del pago (teléfono, banco, referencia, fecha)
   - Información del cliente

3. El backend procesa el pago y retorna el resultado

**Importante:** 
- Si el tipo es `purchaseOrder`, debes enviar `purchaseOrder` en el body en lugar de `reservationCode`
- La fecha debe estar en formato `YYYY-MM-DD`
- El tipo de pago debe ser `"BNCPM"` para Pago Móvil

#### 10. Implementación del Botón "Atrás" (Opción B)

Si elegiste la Opción B de navegación, debes agregar un botón "Atrás" en la página de checkout de Cashea que permita al usuario volver a la página de selección de métodos de pago.

---

## Instalación y Configuración

### 1. Instalación del SDK

```bash
npm install cashea-web-checkout-sdk
```

### 2. Configuración de TypeScript

Si estás usando TypeScript, crea un archivo de definiciones de tipos. Crea `types/cashea-web-checkout-sdk.d.ts`:

```typescript
declare module 'cashea-web-checkout-sdk' {
  interface CheckoutSDKOptions {
    apiKey: string
  }

  interface Product {
    id: string
    name: string
    sku: string
    description: string
    imageUrl: string
    quantity: number
    price: number
    tax: number | null
    discount: number
  }

  interface Store {
    id: string
    name: string
    enabled: boolean
    type?: string
  }

  interface Order {
    store: Store
    products: Product[]
  }

  interface Payload {
    identificationNumber: string
    externalClientId: string
    deliveryMethod: 'IN_STORE' | 'DELIVERY'
    merchantName: string
    redirectUrl: string
    invoiceId: string
    deliveryPrice: number
    orders: Order[]
  }

  interface CreateCheckoutButtonOptions {
    payload: Payload
    container: HTMLElement
  }

  class CheckoutSDK {
    constructor(options: CheckoutSDKOptions)
    createCheckoutButton(options: CreateCheckoutButtonOptions): void
  }

  export default CheckoutSDK
}
```

### 3. Variables de Entorno

Aunque la `publicKey` se obtiene del backend, puedes configurar la URL base de tu API en variables de entorno:

```env
NEXT_PUBLIC_API_URL=https://api.b9ticketing.com
```

---

## Endpoints de API Backend

### 1. Reprice - Actualizar Precios

Actualiza los precios según el método de pago seleccionado. **Debes usar el endpoint correspondiente según el tipo de orden.**

#### 1.1. Reprice para Reservation

**Endpoint:** `POST /api/v1/reservations/reprice/{reservationCode}/CASHEA`

**URL Completa:** `https://api.b9ticketing.com/api/v1/reservations/reprice/{reservationCode}/CASHEA`

**Método:** `POST`

**Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Parámetros de Ruta:**
- `reservationCode` (string, requerido): Código de reserva de la orden (en la URL)
- `CASHEA` (string, requerido): Método de pago (en la URL, debe ser literalmente "CASHEA")

**Respuesta Exitosa (200):**
```json
{
  "success": true,
  "message": "Precios actualizados",
  "data": {
    "subtotal": 500.00,
    "tax": 80.00,
    "total": 580.00
  }
}
```

**Ejemplo de Request:**
```javascript
const reservationCode = 'ABC123'
const response = await fetch(
  `https://api.b9ticketing.com/api/v1/reservations/reprice/${reservationCode}/CASHEA`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    }
  }
)

const data = await response.json()
```

#### 1.2. Reprice para PurchaseOrder

**Endpoint:** `POST /api/v1/purchaseOrder/reprice/{purchaseOrder}/CASHEA`

**URL Completa:** `https://api.b9ticketing.com/api/v1/purchaseOrder/reprice/{purchaseOrder}/CASHEA`

**Método:** `POST`

**Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Parámetros de Ruta:**
- `purchaseOrder` (string, requerido): Código de orden de compra (en la URL)
- `CASHEA` (string, requerido): Método de pago (en la URL, debe ser literalmente "CASHEA")

**Respuesta Exitosa (200):**
```json
{
  "success": true,
  "message": "Precios actualizados",
  "data": {
    "subtotal": 500.00,
    "tax": 80.00,
    "total": 580.00
  }
}
```

**Ejemplo de Request:**
```javascript
const purchaseOrder = 'PO-456'
const response = await fetch(
  `https://api.b9ticketing.com/api/v1/purchaseOrder/reprice/${purchaseOrder}/CASHEA`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    }
  }
)

const data = await response.json()
```

---

### 2. Obtener Información de Cashea

Obtiene la información necesaria para inicializar el SDK de Cashea.

**Endpoint:** `GET /api/v1/cashea/info`

**URL Completa:** `https://api.b9ticketing.com/api/v1/cashea/info?reservationCode={code}`

**Método:** `GET`

**Query Parameters:**
- `reservationCode` (string, requerido): Código de reserva de la orden

**Respuesta Exitosa (200):**
```json
{
  "publicKey": "pk_live_xxxxxxxxxxxxx",
  "casheaBody": {
    "identificationNumber": "V12345678",
    "externalClientId": "client-123",
    "deliveryMethod": "IN_STORE",
    "merchantName": "Mi Tienda",
    "redirectUrl": "https://mitienda.com/checkout/success",
    "invoiceId": "INV-12345",
    "deliveryPrice": 0,
    "orders": [
      {
        "store": {
          "id": "store-1",
          "name": "Tienda Principal",
          "enabled": true,
          "type": "PHYSICAL"
        },
        "products": [
          {
            "id": "prod-1",
            "name": "Producto Ejemplo",
            "sku": "SKU-001",
            "description": "Descripción del producto",
            "imageUrl": "https://example.com/image.jpg",
            "quantity": 2,
            "price": 250.00,
            "tax": 40.00,
            "discount": 0
          }
        ]
      }
    ]
  }
}
```

**Ejemplo de Request:**
```javascript
const reservationCode = 'ABC123'
const response = await fetch(
  `https://api.b9ticketing.com/api/v1/cashea/info?reservationCode=${reservationCode}`
)

const casheaInfo = await response.json()
```

---

### 3. Obtener Información de Orden (Pago Inicial)

Obtiene información sobre montos de pago inicial y financiamiento.

**Endpoint:** `GET /api/v1/cashea/order`

**URL Completa:** 
- Para Reservation: `https://api.b9ticketing.com/api/v1/cashea/order?reservationCode={code}&casheaId={id}`
- Para PurchaseOrder: `https://api.b9ticketing.com/api/v1/cashea/order?purchaseOrder={code}&casheaId={id}`

**Método:** `GET`

**Query Parameters:**
- `reservationCode` (string, opcional si es reservation): Código de reserva de la orden
- `purchaseOrder` (string, opcional si es purchaseOrder): Código de orden de compra
- `casheaId` (string, requerido): ID de identificación de Cashea

**Nota:** Debes usar `reservationCode` o `purchaseOrder` según el tipo de orden, no ambos.

**Respuesta Exitosa (200):**
```json
{
  "downPaymentUSD": 100.00,
  "downPaymentVES": 3500000.00,
  "financedAmountUSD": 400.00
}
```

**Ejemplo de Request para Reservation:**
```javascript
const reservationCode = 'ABC123'
const casheaId = 'V12345678'
const response = await fetch(
  `https://api.b9ticketing.com/api/v1/cashea/order?reservationCode=${reservationCode}&casheaId=${casheaId}`
)

const orderInfo = await response.json()
```

**Ejemplo de Request para PurchaseOrder:**
```javascript
const purchaseOrder = 'PO-456'
const casheaId = 'V12345678'
const response = await fetch(
  `https://api.b9ticketing.com/api/v1/cashea/order?purchaseOrder=${purchaseOrder}&casheaId=${casheaId}`
)

const orderInfo = await response.json()
```

---

### 4. Procesar Pago con Pago Móvil

Procesa un pago usando Pago Móvil (usado en flujos de pago inicial).

**Endpoint:** `POST /api/v1/bnc/cashea-payment`

**URL Completa:** `https://api.b9ticketing.com/api/v1/bnc/cashea-payment`

**Método:** `POST`

**Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Body:**
```json
{
  "reservationCode": "ABC123",
  "casheaId": "V12345678",
  "payment": {
    "phone": "04141234567",
    "bank": "0172",
    "ref": "1234567890",
    "date": "2024-01-15",
    "type": "BNCPM"
  },
  "paymentInfo": {
    "firstName": "Juan",
    "lastName": "Pérez",
    "phone": "04141234567",
    "email": "juan@example.com"
  }
}
```

**O para PurchaseOrder:**
```json
{
  "purchaseOrder": "PO-456",
  "casheaId": "V12345678",
  "payment": {
    "phone": "04141234567",
    "bank": "0172",
    "ref": "1234567890",
    "date": "2024-01-15",
    "type": "BNCPM"
  },
  "paymentInfo": {
    "firstName": "Juan",
    "lastName": "Pérez",
    "phone": "04141234567",
    "email": "juan@example.com"
  }
}
```

**Parámetros:**
- `reservationCode` (string, opcional si es reservation): Código de reserva
- `purchaseOrder` (string, opcional si es purchaseOrder): Código de orden de compra
- `casheaId` (string, requerido): ID de identificación de Cashea

**Nota:** Debes enviar `reservationCode` o `purchaseOrder` según el tipo, no ambos.
- `payment` (object, requerido): Información del pago
  - `phone` (string): Número de teléfono
  - `bank` (string): Código del banco
  - `ref` (string): Número de referencia
  - `date` (string): Fecha del pago (YYYY-MM-DD)
  - `type` (string): Tipo de pago, debe ser `"BNCPM"`
- `paymentInfo` (object, requerido): Información del cliente

**Respuesta Exitosa (200):**
```json
{
  "success": true,
  "message": "Pago procesado exitosamente",
  "transactionId": "TXN-12345"
}
```

**Ejemplo de Request para Reservation:**
```javascript
const response = await fetch('https://api.b9ticketing.com/api/v1/bnc/cashea-payment', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    reservationCode: 'ABC123',
    casheaId: 'V12345678',
    payment: {
      phone: '04141234567',
      bank: '0172',
      ref: '1234567890',
      date: '2024-01-15',
      type: 'BNCPM'
    },
    paymentInfo: {
      firstName: 'Juan',
      lastName: 'Pérez',
      phone: '04141234567',
      email: 'juan@example.com'
    }
  })
})

const result = await response.json()
```

**Ejemplo de Request para PurchaseOrder:**
```javascript
const response = await fetch('https://api.b9ticketing.com/api/v1/bnc/cashea-payment', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    purchaseOrder: 'PO-456',
    casheaId: 'V12345678',
    payment: {
      phone: '04141234567',
      bank: '0172',
      ref: '1234567890',
      date: '2024-01-15',
      type: 'BNCPM'
    },
    paymentInfo: {
      firstName: 'Juan',
      lastName: 'Pérez',
      phone: '04141234567',
      email: 'juan@example.com'
    }
  })
})

const result = await response.json()
```

---

## Integración del SDK

### 1. Importar el SDK

```typescript
import CheckoutSDK from 'cashea-web-checkout-sdk'
```

### 2. Inicialización con apiKey

La `apiKey` (publicKey) se obtiene del endpoint `/api/v1/cashea/info`. **Nunca** hardcodees esta clave en el frontend.

```typescript
const sdk = new CheckoutSDK({
  apiKey: casheaInfo.publicKey // Obtenida del backend
})
```

### 3. Estructura del Payload Requerido

El `casheaBody` debe tener la siguiente estructura:

```typescript
interface CasheaBody {
  identificationNumber: string      // Número de identificación del cliente
  externalClientId: string          // ID externo del cliente
  deliveryMethod: 'IN_STORE' | 'DELIVERY'
  merchantName: string              // Nombre del comercio
  redirectUrl: string               // URL de redirección después del pago
  invoiceId: string                 // ID de la factura
  deliveryPrice: number             // Precio de entrega
  orders: Array<{
    store: {
      id: string
      name: string
      enabled: boolean
      type?: string
    }
    products: Array<{
      id: string
      name: string
      sku: string
      description: string
      imageUrl: string
      quantity: number
      price: number
      tax: number | null
      discount: number
    }>
  }>
}
```

### 4. Creación del Botón de Checkout

**IMPORTANTE:** El botón NO se crea manualmente. El SDK lo genera automáticamente dentro del contenedor especificado.

```typescript
// 1. Crea una referencia al contenedor
const containerRef = useRef<HTMLDivElement>(null)

// 2. Asegúrate de que el contenedor esté montado y tengas la información
if (casheaInfo && containerRef.current) {
  // 3. Inicializa el SDK
  const sdk = new CheckoutSDK({
    apiKey: casheaInfo.publicKey
  })
  
  // 4. Crea el botón (el SDK lo renderiza automáticamente)
  sdk.createCheckoutButton({
    payload: casheaInfo.casheaBody,
    container: containerRef.current
  })
}

// 5. En el JSX, renderiza el contenedor
return (
  <div>
    {/* Otros elementos */}
    <div ref={containerRef} style={{ minHeight: '50px' }}></div>
  </div>
)
```

### 5. Manejo de Errores

```typescript
try {
  const sdk = new CheckoutSDK({
    apiKey: casheaInfo.publicKey
  })
  
  sdk.createCheckoutButton({
    payload: casheaInfo.casheaBody,
    container: containerRef.current!
  })
} catch (error) {
  console.error('Error inicializando Cashea SDK:', error)
  setError('Error al inicializar el botón de pago')
}
```

---

## Ejemplos de Código

### Componente de Checkout Principal

Este componente muestra los items y el botón "Pagar con Cashea". Cuando el usuario hace clic, debe llamar al endpoint de reprice correspondiente (según el tipo) y luego navegar.

```typescript
'use client'

import { useState, Suspense } from 'react'
import { useSearchParams, useRouter } from 'next/navigation'

function CheckoutContent() {
  const searchParams = useSearchParams()
  const router = useRouter()
  const reservationCode = searchParams.get('reservationCode')
  const purchaseOrder = searchParams.get('purchaseOrder')
  const type = searchParams.get('type') // 'reservation' o 'purchaseOrder'
  
  const [isProcessing, setIsProcessing] = useState(false)

  const handleCasheaPayment = async () => {
    try {
      setIsProcessing(true)
      
      // Determinar el código y tipo
      const code = reservationCode || purchaseOrder
      const orderType = type || (reservationCode ? 'reservation' : 'purchaseOrder')
      
      if (!code) {
        throw new Error('No se encontró código de reserva o purchaseOrder')
      }
      
      // 1. Llamar al endpoint de reprice según el tipo
      let repriceUrl: string
      if (orderType === 'purchaseOrder') {
        repriceUrl = `https://api.b9ticketing.com/api/v1/purchaseOrder/reprice/${code}/CASHEA`
      } else {
        repriceUrl = `https://api.b9ticketing.com/api/v1/reservations/reprice/${code}/CASHEA`
      }
      
      const repriceResponse = await fetch(repriceUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        }
      })
      
      if (!repriceResponse.ok) {
        throw new Error('Error al actualizar precios')
      }
      
      // 2. Navegar a la página de checkout de Cashea
      const url = `/cashea-checkout?reservationCode=${code}`
      
      // Opción A: Nueva pestaña
      // window.open(url, '_blank')
      
      // Opción B: Misma pestaña (con botón atrás)
      router.push(url)
      
    } catch (error) {
      console.error('Error:', error)
      alert('Error al procesar. Por favor intenta nuevamente.')
    } finally {
      setIsProcessing(false)
    }
  }

  return (
    <div>
      {/* Mostrar items, resumen, etc. */}
      
      <div className="payment-buttons">
        <button onClick={handleCasheaPayment} disabled={isProcessing}>
          ⚡ Pagar con Cashea
        </button>
      </div>
    </div>
  )
}

export default function CheckoutPage() {
  return (
    <Suspense fallback={<div>Cargando...</div>}>
      <CheckoutContent />
    </Suspense>
  )
}
```

### Componente de Checkout de Cashea

Este componente obtiene la información de Cashea, inicializa el SDK y renderiza el botón.

```typescript
'use client'

import { useEffect, useRef, useState } from 'react'
import { useSearchParams, useRouter } from 'next/navigation'
import CheckoutSDK from 'cashea-web-checkout-sdk'

interface CasheaInfo {
  publicKey: string
  casheaBody: {
    identificationNumber: string
    externalClientId: string
    deliveryMethod: 'IN_STORE' | 'DELIVERY'
    merchantName: string
    redirectUrl: string
    invoiceId: string
    deliveryPrice: number
    orders: Array<{
      store: {
        id: string
        name: string
        enabled: boolean
        type?: string
      }
      products: Array<{
        id: string
        name: string
        sku: string
        description: string
        imageUrl: string
        quantity: number
        price: number
        tax: number | null
        discount: number
      }>
    }>
  }
}

export default function CasheaCheckout() {
  const containerRef = useRef<HTMLDivElement>(null)
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)
  const [casheaInfo, setCasheaInfo] = useState<CasheaInfo | null>(null)
  const hasInitialized = useRef(false)
  const router = useRouter()
  const reservationCode = useSearchParams().get('reservationCode') || ''

  // 1. Obtener información de Cashea desde la API
  useEffect(() => {
    const fetchCasheaInfo = async () => {
      try {
        setIsLoading(true)
        const response = await fetch(
          `https://api.b9ticketing.com/api/v1/cashea/info?reservationCode=${reservationCode}`
        )
        
        if (!response.ok) {
          throw new Error('Error al obtener información de Cashea')
        }
        
        const data = await response.json()
        setCasheaInfo(data)
        setError(null)
      } catch (err) {
        console.error('Error fetching Cashea info:', err)
        setError('No se pudo cargar la información de pago')
      } finally {
        setIsLoading(false)
      }
    }

    if (reservationCode) {
      fetchCasheaInfo()
    }
  }, [reservationCode])

  // 2. Inicializar SDK cuando tengamos la información
  useEffect(() => {
    // Asegurarse de que tenemos la información, el contenedor está montado y no se ha inicializado antes
    if (!casheaInfo || !containerRef.current || hasInitialized.current) return

    hasInitialized.current = true

    try {
      // Inicializar el SDK con la clave pública
      const sdk = new CheckoutSDK({
        apiKey: casheaInfo.publicKey,
      })
      
      // Crear el botón (el SDK lo renderiza automáticamente)
      sdk.createCheckoutButton({
        payload: casheaInfo.casheaBody as any,
        container: containerRef.current,
      })
    } catch (error) {
      console.error('Error inicializando Cashea SDK:', error)
      setError('Error al inicializar el botón de pago')
    }
  }, [casheaInfo])

  // Estados de carga y error
  if (isLoading) {
    return <div>Cargando información de pago...</div>
  }

  if (error || !casheaInfo) {
    return (
      <div>
        <h1>Error</h1>
        <p>{error || 'No se pudo cargar la información'}</p>
        {/* Botón Atrás (Opción B) */}
        <button onClick={() => router.back()}>
          ← Volver
        </button>
      </div>
    )
  }

  return (
    <div>
      <h1>Cashea Checkout</h1>
      
      {/* Mostrar información de la orden */}
      <div>
        <h3>{casheaInfo.casheaBody.merchantName}</h3>
        {/* Mostrar productos, totales, etc. */}
      </div>
      
      {/* Contenedor donde el SDK renderizará el botón */}
      <div ref={containerRef} style={{ minHeight: '50px' }}></div>
      
      {/* Botón Atrás (Opción B) */}
      <button onClick={() => router.back()}>
        ← Volver a métodos de pago
      </button>
    </div>
  )
}
```

### Componente de Checkout Inicial de Cashea

**IMPORTANTE:** Este componente se accede **DESPUÉS** de que el usuario completa el pago en Cashea. Cuando el flujo requiere un pago inicial adicional, Cashea redirige automáticamente a esta página con los parámetros necesarios.

Se accede a través de la ruta `cashea-checkout-inicial/{code}/{type}?idNumber={casheaId}`.

**Flujo:**
1. Usuario completa el pago en la página de checkout de Cashea (usando el SDK)
2. Si la orden requiere pago inicial, Cashea redirige automáticamente a esta página
3. El `idNumber` (casheaId) se obtiene automáticamente del proceso de pago en Cashea
4. Esta página muestra los montos de pago inicial y permite procesar el pago adicional

**Estructura de la ruta:**
- `{code}`: Código de reserva o purchaseOrder
- `{type}`: `"reservation"` o `"purchaseOrder"`
- Query param `idNumber`: ID de identificación de Cashea

```typescript
'use client'

import { useEffect, useState, Suspense } from 'react'
import { useParams, useSearchParams } from 'next/navigation'

interface OrderInfo {
  downPaymentUSD: number
  downPaymentVES: number
  financedAmountUSD: number
}

interface PaymentForm {
  phone: string
  bank: string
  ref: string
}

function CasheaCheckoutInicialContent() {
  const params = useParams()
  const searchParams = useSearchParams()
  
  // Obtener parámetros de la ruta
  const code = params.reservationCode as string // Puede ser reservationCode o purchaseOrder
  const type = params.type as string // 'reservation' o 'purchaseOrder'
  
  // Obtener idNumber de query params
  const casheaId = searchParams.get('idNumber')
  
  const [orderInfo, setOrderInfo] = useState<OrderInfo | null>(null)
  const [isLoadingOrder, setIsLoadingOrder] = useState(true)
  const [showPaymentForm, setShowPaymentForm] = useState(false)
  const [paymentForm, setPaymentForm] = useState<PaymentForm>({
    phone: '',
    bank: '0172',
    ref: ''
  })
  const [isProcessing, setIsProcessing] = useState(false)
  const [error, setError] = useState<string | null>(null)

  // Obtener información de la orden
  useEffect(() => {
    const fetchOrderInfo = async () => {
      if (!casheaId || !code) {
        setIsLoadingOrder(false)
        setError('Faltan parámetros requeridos (casheaId o código)')
        return
      }
      
      try {
        setIsLoadingOrder(true)
        setError(null)
        
        // Construir la URL según el tipo
        const orderUrl = type === 'purchaseOrder'
          ? `https://api.b9ticketing.com/api/v1/cashea/order?purchaseOrder=${code}&casheaId=${casheaId}`
          : `https://api.b9ticketing.com/api/v1/cashea/order?reservationCode=${code}&casheaId=${casheaId}`
        
        const response = await fetch(orderUrl)
        
        if (!response.ok) {
          throw new Error('Error al obtener información de la orden')
        }
        
        const data = await response.json()
        setOrderInfo({
          downPaymentUSD: data.downPaymentUSD,
          downPaymentVES: data.downPaymentVES,
          financedAmountUSD: data.financedAmountUSD
        })
      } catch (err: any) {
        console.error('Error fetching order info:', err)
        setError(err.message || 'No se pudo cargar la información de la orden')
      } finally {
        setIsLoadingOrder(false)
      }
    }

    fetchOrderInfo()
  }, [code, casheaId])

  // Procesar pago con Pago Móvil
  const handlePaymentSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    
    if (!paymentForm.phone || !paymentForm.bank || !paymentForm.ref) {
      setError('Por favor completa todos los campos')
      return
    }

    try {
      setIsProcessing(true)
      setError(null)
      
      // Fecha de hoy en formato YYYY-MM-DD
      const today = new Date().toISOString().split('T')[0]
      
      // Construir el body según el tipo
      const paymentBody = type === 'purchaseOrder'
        ? {
            purchaseOrder: code,
            casheaId: casheaId,
            payment: {
              phone: paymentForm.phone,
              bank: paymentForm.bank,
              ref: paymentForm.ref,
              date: today,
              type: 'BNCPM'
            },
            paymentInfo: {
              firstName: ' ',
              lastName: ' ',
              phone: ' ',
              email: ' '
            }
          }
        : {
            reservationCode: code,
            casheaId: casheaId,
            payment: {
              phone: paymentForm.phone,
              bank: paymentForm.bank,
              ref: paymentForm.ref,
              date: today,
              type: 'BNCPM'
            },
            paymentInfo: {
              firstName: ' ',
              lastName: ' ',
              phone: ' ',
              email: ' '
            }
          }
      
      const response = await fetch('https://api.b9ticketing.com/api/v1/bnc/cashea-payment', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(paymentBody)
      })

      if (!response.ok) {
        throw new Error('Error al procesar el pago')
      }

      // Pago exitoso
      alert('Pago procesado exitosamente')
    } catch (err) {
      console.error('Error processing payment:', err)
      setError('No se pudo procesar el pago. Por favor intenta nuevamente.')
    } finally {
      setIsProcessing(false)
    }
  }

  if (isLoadingOrder) {
    return <div>Cargando información...</div>
  }

  if (error && !orderInfo) {
    return (
      <div>
        <h1>Error</h1>
        <p>{error}</p>
        <p><strong>Código:</strong> {code || 'No encontrado'}</p>
        <p><strong>Tipo:</strong> {type || 'No encontrado'}</p>
        <p><strong>Cashea ID:</strong> {casheaId || 'No encontrado'}</p>
      </div>
    )
  }

  return (
    <div>
      <h1>Pago Inicial</h1>
      
      <div>
        <p><strong>Código:</strong> {code}</p>
        <p><strong>Tipo:</strong> {type}</p>
      </div>

      {/* Mostrar información de pagos */}
      {orderInfo && (
        <div>
          <h3>Montos a Pagar</h3>
          <p>Entrada USD: ${orderInfo.downPaymentUSD.toFixed(2)}</p>
          <p>Entrada VES: Bs. {orderInfo.downPaymentVES.toFixed(2)}</p>
          <p>Financiado USD: ${orderInfo.financedAmountUSD.toFixed(2)}</p>
        </div>
      )}

      {!showPaymentForm ? (
        <div>
          <p>Selecciona un método de pago</p>
          <p style={{ fontSize: '0.875rem', color: '#666', marginBottom: '1rem' }}>
            Métodos disponibles: BNC (Pago Móvil) - Stripe y BAPM próximamente
          </p>
          <button onClick={() => setShowPaymentForm(true)}>
            📱 Pago Móvil (BNC)
          </button>
      ) : (
        <form onSubmit={handlePaymentSubmit}>
          <h3>Formulario de Pago Móvil</h3>

          {error && (
            <div style={{ color: 'red' }}>{error}</div>
          )}

          <div>
            <label>Teléfono *</label>
            <input
              type="tel"
              value={paymentForm.phone}
              onChange={(e) => setPaymentForm({ ...paymentForm, phone: e.target.value })}
              placeholder="04141234567"
              required
            />
          </div>

          <div>
            <label>Banco *</label>
            <select
              value={paymentForm.bank}
              onChange={(e) => setPaymentForm({ ...paymentForm, bank: e.target.value })}
              required
            >
              <option value="0172">Banca Amiga</option>
              <option value="0171">Banco del Tesoro</option>
              {/* Más opciones de bancos */}
            </select>
          </div>

          <div>
            <label>Referencia *</label>
            <input
              type="text"
              value={paymentForm.ref}
              onChange={(e) => setPaymentForm({ ...paymentForm, ref: e.target.value })}
              placeholder="Número de referencia"
              required
            />
          </div>

          <div>
            <button type="button" onClick={() => setShowPaymentForm(false)}>
              Cancelar
            </button>
            <button type="submit" disabled={isProcessing}>
              {isProcessing ? 'Procesando...' : 'Confirmar Pago'}
            </button>
          </div>
        </form>
      )}
    </div>
  )
}

export default function CasheaCheckoutInicial() {
  return (
    <Suspense fallback={<div>Cargando...</div>}>
      <CasheaCheckoutInicialContent />
    </Suspense>
  )
}
```

**Estructura de archivos en Next.js:**
```
app/
  └── cashea-checkout-inicial/
      └── [reservationCode]/
          └── [type]/
              └── page.tsx
```

**Nota sobre la navegación:**

Normalmente, **no necesitas navegar manualmente a esta página**. Cashea redirige automáticamente después del pago cuando se requiere un pago inicial. Sin embargo, si necesitas construir la URL manualmente (por ejemplo, para testing o casos especiales):

```typescript
// Para reservation
const url = `/cashea-checkout-inicial/${reservationCode}/reservation?idNumber=${casheaId}`
router.push(url)

// Para purchaseOrder
const url = `/cashea-checkout-inicial/${purchaseOrder}/purchaseOrder?idNumber=${casheaId}`
router.push(url)
```

**Importante:** El `idNumber` (casheaId) se obtiene del proceso de pago en Cashea. Asegúrate de que tu backend configure correctamente la `redirectUrl` en el `casheaBody` para que Cashea redirija a esta página con los parámetros correctos.

### Manejo de Estados

```typescript
// Estados recomendados
const [isLoading, setIsLoading] = useState(true)
const [error, setError] = useState<string | null>(null)
const [casheaInfo, setCasheaInfo] = useState<CasheaInfo | null>(null)

// Prevenir inicialización múltiple
const hasInitialized = useRef(false)
```

### Navegación entre Páginas

**Opción A: Nueva Pestaña**
```typescript
const handleCasheaPayment = () => {
  const url = `/cashea-checkout?reservationCode=${reservationCode}`
  window.open(url, '_blank')
}
```

**Opción B: Misma Pestaña con Botón Atrás**
```typescript
import { useRouter } from 'next/navigation'

const router = useRouter()

// Navegar
const handleCasheaPayment = () => {
  const url = `/cashea-checkout?reservationCode=${reservationCode}`
  router.push(url)
}

// Botón atrás en la página de checkout
<button onClick={() => router.back()}>
  ← Volver
</button>
```

---

## Estructura de Datos

### Interfaces TypeScript Completas

```typescript
// Información completa de Cashea
interface CasheaInfo {
  publicKey: string
  casheaBody: CasheaBody
}

// Payload para el SDK
interface CasheaBody {
  identificationNumber: string
  externalClientId: string
  deliveryMethod: 'IN_STORE' | 'DELIVERY'
  merchantName: string
  redirectUrl: string
  invoiceId: string
  deliveryPrice: number
  orders: Order[]
}

// Orden
interface Order {
  store: Store
  products: Product[]
}

// Tienda
interface Store {
  id: string
  name: string
  enabled: boolean
  type?: string
}

// Producto
interface Product {
  id: string
  name: string
  sku: string
  description: string
  imageUrl: string
  quantity: number
  price: number
  tax: number | null
  discount: number
}

// Información de orden (para pago inicial)
interface OrderInfo {
  downPaymentUSD: number
  downPaymentVES: number
  financedAmountUSD: number
}
```

---

## Mejores Prácticas

### 1. Manejo de Errores

- Siempre maneja errores en las llamadas a la API
- Muestra mensajes de error claros al usuario
- Implementa retry logic para llamadas fallidas si es necesario

```typescript
try {
  const response = await fetch(url)
  if (!response.ok) {
    throw new Error(`Error ${response.status}`)
  }
  const data = await response.json()
} catch (error) {
  console.error('Error:', error)
  setError('No se pudo cargar la información')
}
```

### 2. Estados de Carga

- Muestra un indicador de carga mientras se obtiene la información
- Deshabilita botones durante el procesamiento
- Evita múltiples llamadas simultáneas

```typescript
const [isLoading, setIsLoading] = useState(true)

if (isLoading) {
  return <div>Cargando...</div>
}
```

### 3. Validación de Datos

- Valida que `reservationCode` esté presente antes de hacer llamadas
- Verifica que `casheaInfo` esté completo antes de inicializar el SDK
- Valida que el contenedor esté montado antes de crear el botón

```typescript
if (!reservationCode) {
  setError('Código de reserva no encontrado')
  return
}

if (!casheaInfo || !containerRef.current) {
  return
}
```

### 4. Seguridad

- **NUNCA** hardcodees la `publicKey` en el frontend
- Siempre obtén la `publicKey` del backend
- No expongas claves privadas
- Usa HTTPS para todas las comunicaciones

### 5. Cuándo Usar Nueva Pestaña vs Navegación en la Misma Página

**Nueva Pestaña (`window.open`):**
- ✅ Permite al usuario mantener abierta la página original
- ✅ Útil para comparar métodos de pago
- ❌ Puede ser bloqueada por popup blockers

**Misma Pestaña con Botón Atrás (`router.push`):**
- ✅ Mejor experiencia en móviles
- ✅ No bloqueado por popup blockers
- ✅ Permite navegación fácil con botón atrás
- ❌ El usuario pierde la página original (a menos que uses botón atrás)

### 6. Asegurar que el Contenedor Esté Montado

El SDK requiere que el contenedor HTML esté montado en el DOM antes de inicializarlo:

```typescript
useEffect(() => {
  // Verificar que el contenedor existe
  if (!containerRef.current) return
  
  // Verificar que tenemos la información
  if (!casheaInfo) return
  
  // Prevenir inicialización múltiple
  if (hasInitialized.current) return
  
  // Inicializar SDK
  hasInitialized.current = true
  const sdk = new CheckoutSDK({ apiKey: casheaInfo.publicKey })
  sdk.createCheckoutButton({
    payload: casheaInfo.casheaBody,
    container: containerRef.current
  })
}, [casheaInfo]) // Solo cuando casheaInfo cambie
```

---

## Troubleshooting

### El Botón No Aparece

**Problema:** El botón de Cashea no se renderiza en la página.

**Soluciones:**
1. Verifica que el contenedor esté montado:
   ```typescript
   console.log('Container:', containerRef.current) // No debe ser null
   ```

2. Verifica que `casheaInfo` esté disponible:
   ```typescript
   console.log('Cashea Info:', casheaInfo) // Debe tener publicKey y casheaBody
   ```

3. Verifica que no se haya inicializado múltiples veces:
   ```typescript
   // Usa un ref para prevenir inicialización múltiple
   const hasInitialized = useRef(false)
   ```

4. Revisa la consola del navegador para errores del SDK

### Error al Obtener Información de la API

**Problema:** La llamada a `/api/v1/cashea/info` falla.

**Soluciones:**
1. Verifica que `reservationCode` esté presente y sea válido
2. Verifica la URL de la API
3. Revisa los headers de la petición
4. Verifica que el backend esté respondiendo correctamente

### El SDK No Se Inicializa

**Problema:** Error al crear la instancia del SDK.

**Soluciones:**
1. Verifica que `publicKey` sea válida y no esté vacía
2. Asegúrate de que el SDK esté instalado: `npm list cashea-web-checkout-sdk`
3. Verifica que el `casheaBody` tenga la estructura correcta
4. Revisa la consola para mensajes de error específicos

### Errores de TypeScript

**Problema:** Errores de tipos al usar el SDK.

**Soluciones:**
1. Asegúrate de tener el archivo de definiciones de tipos (`types/cashea-web-checkout-sdk.d.ts`)
2. Verifica que TypeScript esté configurado para incluir el directorio `types`
3. Reinicia el servidor de TypeScript en tu IDE

### El Botón Se Renderiza Múltiples Veces

**Problema:** El botón aparece duplicado o se crea varias veces.

**Soluciones:**
1. Usa un `ref` para prevenir inicialización múltiple:
   ```typescript
   const hasInitialized = useRef(false)
   
   if (hasInitialized.current) return
   hasInitialized.current = true
   ```

2. Asegúrate de que el `useEffect` tenga las dependencias correctas

### Problemas de Navegación

**Problema:** La navegación no funciona correctamente.

**Soluciones:**
1. Si usas `window.open()`, verifica que no esté bloqueado por popup blockers
2. Si usas `router.push()`, asegúrate de importar `useRouter` correctamente
3. Verifica que la URL de destino sea correcta

---

## Recursos Adicionales

- [Documentación oficial de Cashea](https://docs.cashea.com)
- [Repositorio del SDK](https://github.com/cashea/web-checkout-sdk)
- Soporte: soporte@cashea.com

---

## Conclusión

Esta guía cubre todos los aspectos necesarios para integrar Cashea en tu aplicación. Recuerda:

1. ✅ Siempre llama al endpoint de reprice antes de navegar al checkout de Cashea
2. ✅ Obtén la información de Cashea desde tu backend para inicializar el SDK
3. ✅ El botón se genera automáticamente por el SDK en la página de checkout de Cashea
4. ✅ Después del pago en Cashea, si se requiere pago inicial, el sistema redirige automáticamente a la página de checkout inicial
5. ✅ La página de checkout inicial se accede después del pago en Cashea, no antes
6. ✅ Maneja errores y estados de carga apropiadamente
7. ✅ Elige la opción de navegación que mejor se adapte a tu caso de uso

Si tienes preguntas o necesitas ayuda, contacta al equipo de soporte de Cashea.

