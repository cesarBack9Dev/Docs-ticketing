# Cambios de Reserva

Documento que describe las funcionalidades relacionadas con modificaciones sobre reservas existentes. Se contemplan **tres funcionalidades distintas**:

1. **Exchanges** — Cambio de reserva a otro evento
2. **Upgrades** — Mejora de asientos en el mismo evento
3. **Modificación de datos** — Cambio de usuario titular y cambio de nombre en tickets

---

# 1. Exchanges (Cambio a Otro Evento)

## 1.1 Objetivo

Permitir cambiar una reserva de un evento A a un evento B dentro del mismo **eventPlanner**. Se trabaja sobre la **misma reserva**: al confirmar, se actualizan `event`, `locations` y `price`; no se crea una reserva nueva. Si el evento destino tiene precio mayor, el usuario paga la diferencia; si es menor o igual, no hay reembolso (o crédito, según política).

## 1.2 Configuración en EventPlanner

- `enableExchanges`: Boolean para habilitar/deshabilitar exchanges
- `exchangePolicy`:
  - `allowRefundOnDowngrade`: Si precio destino < origen, permitir reembolso o crédito
  - `maxDaysBeforeEvent`: Límite de días antes del evento
  - `sameSeatMapRequired`: Requerir mismo seat map para mantener asientos

## 1.3 Flujo de precios

| Escenario | Acción |
|-----------|--------|
| Precio destino > precio origen | Usuario paga la diferencia (Order, flujo de pago) |
| Precio destino ≤ precio origen | Exchange inmediato, sin pago adicional |
| Precio destino < precio origen | Según `allowRefundOnDowngrade`: crédito o sin reembolso |

## 1.4 Eventos con mapa de asientos (seat map fijo)

**Opción A: Mantener mismos asientos**
- Validar mismo seat map en ambos eventos
- Validar disponibilidad de localidades en evento destino
- Si falla → ofrecer elegir nuevos asientos

**Opción B: Elegir nuevos asientos**
- Usuario selecciona nuevas localidades en el mapa del evento destino
- Validar disponibilidad

**Parámetro:** `keepSameSeats`: Boolean

## 1.5 Eventos por aforo (capacity)

- **Mantener localidades**: Validar que no estén tomadas en destino
- **Si ocupadas**: Asignar nuevas localidades aleatoriamente en la misma sección/plan
- **Plan inválido en destino**: Usuario debe seleccionar nuevo plan (tipo de boleto)

## 1.6 Resumen por tipo de mapa

| Tipo | Mantener localidades | Si ocupadas |
|------|---------------------|-------------|
| **Asientos fijos** | Usar mismas locations | Usuario debe elegir otros asientos |
| **Aforo** | Intentar mismas locations | Asignar aleatoriamente en misma sección/plan |

## 1.7 Flujo al confirmar el exchange

Se trabaja sobre la **misma reserva**; no se crea una nueva.

Un exchange **confirmado** es aquel en el que ya se aplicaron todos los cambios: disponibilidad reservada en el evento destino, localidades asignadas y precio actualizado. En ese momento:

1. Las localidades quedan reservadas en el evento destino
2. La reserva se actualiza con:
   - `event`: evento destino (reemplaza el evento origen)
   - `locations`: nuevas localidades en el evento destino
   - `price`: nuevo precio
   - `taxes`: si aplica
3. Se invalidan o eliminan los tickets del evento origen
4. Se generan nuevos tickets para el evento destino (asociados a la misma reserva)

La reserva mantiene su `reservationCode`, `user`, original, etc. Solo cambian `event`, `locations` y `price`.

## 1.8 Consideraciones

- Solo eventos simples (no season tickets ni multi-events)
- Solo reservas con `reservationStatus: "tickets_issued"`
- Origen y destino deben pertenecer al mismo eventPlanner
- Modelo Exchange: `reservation`, `sourceEvent`, `targetEvent`, `priceDifference`, `order` (si aplica) — registra el historial del cambio sobre la misma reserva

## 1.9 Endpoints sugeridos

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/exchanges/preview` | Preview de precio |
| POST | `/exchanges/request` | Crear exchange y Order si hay diferencia |
| GET | `/exchanges/:exchangeId` | Estado del exchange |

---

# 2. Upgrades (Mejora de Asientos en el Mismo Evento)

## 2.1 Objetivo

Permitir que el usuario mejore sus asientos dentro del **mismo evento** (ej: de General a VIP). El usuario paga la diferencia de precio.

## 2.2 Diferencias con Exchanges

| | Exchanges | Upgrades |
|---|-----------|----------|
| Evento | Cambia (A → B) | Mismo |
| Reserva | Misma reserva (actualizar `event`, `locations`, `price`) | Misma reserva (actualizar `locations`, `price`) |
| Tickets | Invalidar antiguos, generar nuevos para evento destino | Actualizar localidades |

## 2.3 Flujo

1. Usuario solicita upgrade indicando nuevas localidades (mejor plan)
2. Validar disponibilidad de las nuevas localidades
3. Calcular diferencia de precio: `priceNew - priceCurrent`
4. Si diferencia > 0: crear Order para pago
5. Al aprobar pago: actualizar `locations` y `price` en la reserva existente, regenerar/actualizar tickets

## 2.4 Consideraciones

- Mismo evento → no hay validación de seat map entre eventos
- Reutilizar lógica de diferencia de precio y pago (compartida con exchanges)
- Actualizar tickets existentes en lugar de cancelar/crear

## 2.5 Endpoints sugeridos

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/upgrades/preview` | Preview: `priceDifference`, `priceNew`, `priceCurrent` |
| POST | `/upgrades/request` | Crear upgrade y Order si hay diferencia |
| GET | `/upgrades/:upgradeId` | Estado del upgrade |

---

# 3. Modificación de Datos de la Reserva

Funcionalidad que agrupa dos tipos de cambios que **no involucran evento, asientos ni precio**:

- Cambio de usuario titular (asignación)
- Cambio de nombre en tickets

## 3.1 Cambio de Usuario Titular (Asignación)

### Objetivo

Asignar o cambiar el **usuario titular** de una reserva: la cuenta de la plataforma que posee la reserva y puede verla, gestionarla y descargar los tickets. Típicamente desde **backoffice** (admin/empleado).

### Casos de uso

- Transferir reserva a otro usuario
- Corregir usuario asignado por error
- Reasignar reserva comprada por un tercero

### ¿Qué cambia y qué no?

| Elemento | ¿Cambia? | Notas |
|----------|----------|-------|
| **Reserva** | Sí | Se actualiza el campo `user` (nuevo titular) |
| **Visibilidad** | Sí | La reserva pasa a aparecer en el perfil del nuevo usuario |
| **Acceso a tickets** | Sí | El nuevo titular puede descargar y gestionar los tickets |
| **Nombres en tickets** | No | Los tickets muestran los nombres de los asistentes (`additionalUsers` → `contactInfo`), no el titular. Esos nombres no cambian con esta operación |

En resumen: se cambia **quién posee** la reserva, no **qué nombres aparecen** en los tickets. Para cambiar los nombres mostrados en los tickets, usar la funcionalidad **3.2 Cambio de nombre en tickets**.

### Consideraciones

- Solo backoffice (no desde perfil de usuario)
- Actualizar `user` en la reserva
- Actualizar `user` en los tickets asociados (para mantener consistencia de propiedad)
- Auditoría: registrar quién hizo el cambio y cuándo

### Endpoint sugerido

| Método | Ruta | Descripción |
|--------|------|-------------|
| PATCH | `/reservations/:reservationCode/assign-user` | Asignar usuario a reserva (backoffice) |

---

## 3.2 Cambio de Nombre en Tickets

### Objetivo

Actualizar los nombres o datos que aparecen en los tickets. Afecta la información mostrada en cada ticket (asociada a `additionalUsers` en la reserva).

### Casos de uso

- Usuario corrige typo en nombre desde su perfil
- Actualizar datos de acompañantes
- Backoffice corrige información a solicitud del cliente

### Dónde se puede hacer

- **Perfil de usuario**: El titular puede editar sus reservas
- **Backoffice**: Admin/empleado puede editar cualquier reserva

### Consideraciones

- Actualizar `aditionalsUsers` en la reserva (fuente de datos para los tickets)
- Propagación a tickets: si los tickets ya fueron emitidos, decidir si se regeneran o se mantiene histórico
- Mantener correspondencia `locations[i]` ↔ `aditionalsUsers[i]` (según estructura actual)

### Endpoint sugerido

| Método | Ruta | Descripción |
|--------|------|-------------|
| PATCH | `/reservations/:reservationCode/ticket-names` | Actualizar nombres en tickets (perfil o backoffice) |

---

# Resumen de las 3 Funcionalidades

| Funcionalidad | Qué cambia | Dónde | Pago |
|---------------|------------|-------|------|
| **Exchanges** | Evento (A → B) | Perfil / Backoffice | Diferencia si precio mayor |
| **Upgrades** | Asientos (mejor plan, mismo evento) | Perfil / Backoffice | Diferencia si precio mayor |
| **Modificación de datos** | Usuario titular y/o nombres en tickets | Perfil (nombres) / Backoffice (ambos) | No |
