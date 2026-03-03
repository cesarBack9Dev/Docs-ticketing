# accessibilityType

Campo del modelo **Location** que indica el tipo de asiento accesible para personas con discapacidad o necesidades especiales.

## Ubicación

- **Modelo:** `Location`
- **Tipo:** String (enum)
- **Opcional:** Sí

## Valores permitidos

| Valor | Descripción |
|-------|-------------|
| `wheelchair` | Asiento para silla de ruedas |
| `companion` | Asiento para acompañante (junto a asiento accesible) |
| `transfer` | Asiento con espacio para transferencia desde silla de ruedas |
| `ambulatory` | Para personas con dificultad para caminar |
| `hearing` | Cerca de interpretación en lengua de señas |
| `vision` | Ubicación preferida para baja visión |
| `sensory` | Zona de menor estimulación sensorial (ej. autismo) |
| `obstructedView` | Visibilidad reducida |

## Uso

### Crear/actualizar location

```json
{
  "seatMap": "...",
  "zone": "...",
  "section": "...",
  "row": "...",
  "seat": "...",
  "accessibilityType": "wheelchair"
}
```

### Respuesta en seat map

Cada location en la respuesta de `seat-map-v3`, `buildSeatMapSkeleton` y `buildSeatMapV3` incluye:

```json
{
  "label": "Fila 1 Asiento 5",
  "location": "...",
  "accessibilityType": "wheelchair"
}
```

Si la location no es accesible, `accessibilityType` será `null`.

## Endpoints afectados

- `POST/PUT` create-or-update-location
- `GET` seat-map-v3
- `GET` seat-map-v5 (skeleton)
- `buildSeatMapV3`
- `buildSeatMapV3Event`
- `buildSeatMapLocationsSkeleton`
