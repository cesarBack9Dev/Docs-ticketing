## Hotfix: mitigación de alto CPU en Mongo (`seats`) y caída por `UnhandledPromiseRejection`

### Contexto / Síntomas
- En MongoDB (Realtime) se observó que la colección **`seats`** estaba consumiendo **~100% CPU**.
- El API se estaba cayendo con:
  - `UnhandledPromiseRejection ... reason "#<Object>" (ERR_UNHANDLED_REJECTION)`

### Cómo se detectó que el problema era `buildSeatMapV3`
- **Se partió del síntoma en Atlas**: “`seats` al 100% CPU”.
- Se buscó en el código dónde se toca la colección `seats`:
  - Búsqueda de `from: "seats"` y usos de `seatModel`.
- Se encontró un “hot path” claro:
  - `src/models/location-model.js` → `Location.buildSeatMapV3()`
  - Dentro del pipeline existe `$lookup` hacia **`seats`** (y también `rows`, `sections`, etc.), lo cual se vuelve muy costoso con mapas grandes y alta concurrencia.
- Se identificaron endpoints “candidatos” que disparan ese flujo (seat maps v3/v4), y en particular:
  - `GET /api/v1/event/seat-map-v3/:event` usa `src/uses-cases/seat-maps/build-seat-map-v3.js`
  - Ese caso de uso llamaba directamente a `locationModel.buildSeatMapV3(...)`

### Objetivo del hotfix
- **Bajar el costo** de construcción del seatmap v3 (principal sospechoso del CPU).
- **Evitar que el proceso caiga** por rechazos no manejados y mejorar el logging para identificar el endpoint culpable.

---

## Cambios aplicados

### 1) Estabilización del proceso y trazabilidad del endpoint (Node/Express)
Archivo: `src/index.js`
- Se agregaron handlers globales:
  - `process.on("unhandledRejection", ...)`
  - `process.on("uncaughtException", ...)`
- Se agregó **contexto de request** para poder identificar **método + URL** cuando ocurra un error:
  - `AsyncLocalStorage` + `x-request-id`
- Se envolvieron rutas/middlewares para que cualquier Promise rechazada termine en el handler de Express (en vez de “unhandled”).

Archivo: `src/express-callback/index.js`
- Se mejoró el log de error para imprimir `METHOD URL` cuando falla un controller.

Resultado:
- El server deja de “morir a ciegas” y queda log claro del endpoint que dispara el error.

---

### 2) Optimización en el caso de uso del seat map v3 (CPU en aplicación)
Archivo: `src/uses-cases/seat-maps/build-seat-map-v3.js`
- Se redujo costo en `sectionsBuild`:
  - Se precomputaron mapas clave→valor para configs (ej. `plansGroupBySectionId`).
  - Se eliminó el patrón O(secciones * tickets) que contaba tickets por sección con `Object.values(...).filter(...)`.
    - Se reemplazó por:
      - `ticketsByLocationId[locationId]`
      - `ticketsCountBySectionId[sectionId]`
  - Se corrigió un `reduce` que asignaba el mismo `acc[location]` dos veces.

Resultado:
- Menos CPU en Node al construir la respuesta final.

---

### 3) Nueva función optimizada en el modelo (reemplazo del aggregate pesado)
Archivo: `src/models/location-model.js`
- Se creó un nuevo método:
  - **`Location.buildSeatMapV3Event()`**
- Intención:
  - Evitar el aggregate grande con `$lookup` a `seats`.
  - Reemplazarlo por:
    - 1 query a `locations` (solo ids necesarios)
    - batch queries por `$in` a `seats`, `rows`, `sections`, `zonenames`, `plans`
    - combinación en memoria con `Map`s (O(n))

Archivo: `src/uses-cases/seat-maps/build-seat-map-v3.js`
- Se cambió la llamada:
  - de `locationModel.buildSeatMapV3(...)`
  - a **`locationModel.buildSeatMapV3Event(...)`**

Bugs corregidos durante el hotfix:
- Variable `seatMap` duplicada (parámetro vs `Map`) → renombrada a `seatById`.
- `zone` venía `null` porque se estaba resolviendo desde `Section` en vez de `Location.zone`.
  - Se corrigió para que la zona se tome desde `loc.zone`.

Resultado:
- Se reduce la carga sobre Mongo (especialmente lo que hacía “caliente” a `seats`).
- Se conserva el “shape” de la respuesta por secciones + locations.

---

### 4) Ajustes “index-friendly” y mitigación en aggregate existente
Archivo: `src/models/location-model.js` (sobre `buildSeatMapV3` existente)
- `disabled: { $ne: true }` se cambió por un criterio más indexable:
  - `disabled: { $in: [null, false] }`
- Se añadió un `$project` temprano para cortar payload.
- Se agregó `.allowDiskUse(true)`.

Índices sugeridos/agregados en el schema (críticos en emergencia):
- `{ seatMap: 1, disabled: 1 }`
- `{ seatMap: 1, section: 1, disabled: 1 }`

Nota:
- En producción, si `autoIndex` está desactivado, estos índices deben crearse manualmente en Mongo.

---

## Qué endpoints quedaron como “alto riesgo” (seat maps)
Principalmente endpoints que construyen mapas grandes:
- `GET /api/v1/event/seat-map-v3/:event` ✅ (migrado a `buildSeatMapV3Event`)
- `GET /api/v1/season-tickets/seat-map-v3/:seasonTicket` (candidato a aplicar el mismo cambio)
- `GET /api/v1/seat-map/build-seats/:seatmap` (usa build V3 internamente)
- `GET /api/v1/event/seat-map-v4/:event` / `GET /api/v1/seat-map-v4/:seatMap` (usan otro camino, pero también son candidatos si hay presión)

---

## Validación recomendada (post-hotfix)
- En Atlas:
  - revisar “Top Operations / Slow Queries” para confirmar caída del CPU en `seats`.
- En Mongo:
  - `db.currentOp({ active: true, ns: /seats/ })` (ver si sigue habiendo operaciones pesadas)
  - `db.system.profile.find({ ns: /seats/ }).sort({ ts: -1 }).limit(20)` (si profiler activo)
- En API:
  - monitorear logs `[unhandledRejection][request] ...` / `[express:error] ...` para detectar endpoints restantes.

---

## Siguientes mejoras (si persiste carga)
- Aplicar `buildSeatMapV3Event` también a:
  - `src/uses-cases/seat-maps/build-seat-map-season-ticket-v3.js`
- Cache por `seatMapId` (TTL corto) para evitar recomputar en ráfagas.
- Reducir payload opcional (p.ej. no traer `svgInfo` si no es necesario).

