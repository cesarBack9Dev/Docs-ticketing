# Informe técnico: Cola SeatMap V3, Cache, Hold Locations y doble verificación de las localidades

Fecha: 2026-01-29  
Proyecto: `ticketing-api`

## Alcance
Este documento resume el trabajo realizado para:
- Controlar la **concurrencia** al construir el SeatMap V3 (cuando el mapa es grande y puede tardar ~12s).
- Fortalecer el flujo de **Holds** (bloqueo temporal de asientos) y la **sincronización en vivo** con SSE.
- Preparar mejoras de consistencia para **reservas** (`addReservationV2`) y selección en **secciones de aforo**.
- Reducir carga de base de datos mediante **cache del esqueleto del mapa**.

## Puntos trabajados (resumen)
- **`cacheService` (Redis wrapper)**: ampliación de comandos para soportar colas, TTL, locks y operaciones atómicas.
- **Hold Locations**:
  - Bloqueo de asientos con **clave única en cache** (evita doble toma concurrente).
  - Notificación en vivo con **SSE** para bloquear asientos en el frontend en tiempo real.
- **Middleware de cola para SeatMap V3 (`getSeatMapEventV3`)**:
  - Cola por `eventId` y variante (`web` / `ticketOffice`).
  - Respuesta 202 “queued” con metadata de cola + **placeholder** del mapa para compatibilidad del frontend.
  - Estabilidad bajo alta concurrencia (1000 usuarios) con expiración y lógica atómica en Redis.
- **Observabilidad**: logs opcionales para tamaño de cola, TTLs y escenarios “fail-open”.
- **Cache del esqueleto del mapa**: guardar y reutilizar estructura base del seatmap para evitar recalcular en DB en cada request.
- **Mejoras planificadas en `addReservationV2`**:
  - Validación contra hold locations de otros usuarios.
  - Creación temprana de una reserva preliminar; si se detecta ocupación, se cancela y se notifica al usuario.
- **Secciones de aforo**:
  - Request para obtener/confirmar la localidad específica durante selección.
  - Evitar localidades duplicadas entre reservas.
  - Evitar “tickets en blanco” antes de reservar y posteriormente emitir el ticket.

## 1) `cacheService` (Redis)
### Objetivo
Habilitar primitives para:
- Colas (ZSET + ranking).
- TTL y limpieza automática.
- Locks (evitar dobles ejecuciones).
- Operaciones atómicas bajo alta concurrencia.

### Comandos/funciones incorporadas
- **TTL/Counters**: `ttl`, `expire`, `incr`
- **ZSET**: `zadd`, `zrank`, `zrem`, `zcard`, `zrange`, `zremrangebyscore`
- **Atomicidad**: `eval` (Lua) para decisiones/movimientos en un solo paso.

### Consideraciones
- Se mantiene comportamiento **best-effort / fail-open** cuando Redis no está listo, para no tumbar el servicio.

## 2) Hold Locations (bloqueo y SSE)
### 2.1 Clave única en cache (evitar doble hold locations)
Cuando un asiento entra en hold locations:
- Se crea una **clave única** en cache (por `eventId + locationId`) usando un patrón tipo **`setNX` con TTL**.
- Esto asegura que **2 usuarios no puedan tomar el mismo asiento al mismo tiempo**.

### 2.2 Notificación en vivo (SSE)
Con el canal SSE de hold locations:
- Se notifica a otros usuarios que estén viendo el mapa del mismo evento cuando un asiento:
  - entra en hold
  - sale de hold
  - expira
- Con esa notificación, el **frontend puede bloquear/desbloquear en vivo** los asientos, evitando refrescos completos del mapa y reduciendo inconsistencias visuales.

## 3) Middleware de cola para SeatMap V3
### Objetivo
Evitar saturación de CPU/DB cuando muchos usuarios solicitan el SeatMap V3 simultáneamente (mapas grandes con tiempos ~12s).

**Acotación**: este control de concurrencia reduce picos de carga (CPU/DB) y ayuda a **evitar colapsos del servidor** cuando hay alta demanda.

### Modelo de cola
- La cola se segmenta por:
  - `eventId`
  - `variant` (WEB vs TicketOffice)
- El middleware responde:
  - **200** cuando el usuario es admitido y continúa al caso de uso.
  - **202** cuando debe esperar (queued).

### Parámetros operativos
- **`MAX_ACTIVE`**: 5 (default).
- **`Retry-After`**: 12 segundos (default), alineado al costo típico del mapa grande.

### Compatibilidad con frontend (respuesta 202)
Cuando el usuario cae en cola (202), se devuelve:
- Datos de cola (token/posición/cupo/reintento).
- Un **placeholder** del mapa para evitar ruptura del contrato en el frontend:

```json
{
  "sections": [],
  "_id": "",
  "viewBox": "0 0 3500 3500",
  "imgUrlMap": ""
}
```

### Estabilidad bajo alta concurrencia (1000 usuarios)
Se atendieron escenarios donde la cola “no avanza” por:
- Clientes que abandonan sin liberar estado.
- Tokens stale que quedan “bloqueando” el head.
- Requests que no disparan el cleanup esperado.

Para mitigar:
- Uso de expiración y limpieza.
- Lógica crítica ejecutada de forma **atómica en Redis** (Lua) para evitar condiciones de carrera.

## 4) Cache del esqueleto del SeatMap
### Objetivo
Evitar que cada request reconstruya por completo el mapa consultando/agregando datos desde DB.

**Acotación**: al combinar cache del esqueleto con la cola, se disminuye el trabajo repetido y se controla la simultaneidad, lo que contribuye a **evitar colapsos del servidor** en picos de tráfico.

### Enfoque
- Guardar en cache el **esqueleto del mapa** (estructura base: secciones/zonas/layout y metadatos como `viewBox`/`imgUrlMap`).
- Reutilizarlo para requests posteriores, y solo complementar con datos dinámicos cuando aplique.

### Beneficios esperados
- Menos queries/agregaciones repetidas en DB.
- Menor presión sobre Mongo/DB en picos de tráfico.
- Mejor latencia percibida en eventos de alta demanda.

## 5) Mejoras planificadas en `addReservationV2`
### 5.1 Validación contra hold locations
Antes de confirmar una reserva, se validará que las localidades/asientos solicitados:
- **no estén en hold por otro usuario** que está seleccionando actualmente.

### 5.2 Reserva preliminar (creación temprana)
Para reducir condiciones de carrera:
- Se creará la reserva **apenas pase la primera validación de localidades**, avanzando hasta la validación final de “ocupación real”.
- Con esa reserva preliminar se **toman/bloquean** esos asientos durante el proceso.
- Si en la validación final se detecta que los asientos **ya están ocupados**, la reserva se:
  - **cancela**
  - y se responde al usuario con el mensaje de que **los asientos están ocupados**.

## 6) Secciones de aforo durante selección (evitar duplicados)
Para secciones de aforo (capacidad):
- Durante la selección en el mapa se hará un request para **obtener/confirmar la localidad específica**.
- Esto evita:
  - **localidades duplicadas** entre reservas concurrentes
  - **tickets en blanco**, asegurando que cada selección se vincule a una localidad válida/confirmada **antes de reservar y posteriormente emitir el ticket**.

