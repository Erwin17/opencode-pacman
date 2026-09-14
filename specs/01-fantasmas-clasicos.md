# SPEC 01 — Cuatro fantasmas con personalidades clásicas

> **Estado:** Approved
> **Depende de:** —
> **Fecha:** 2026-09-08
> **Objetivo:** Dotar al juego de 4 fantasmas con personalidades de persecución clásicas (Blinky, Pinky, Inky y Clyde), de las cuales Blinky persigue agresivamente a Pac-Man.

## Por qué existe esta spec

Reemplaza el sistema actual de 2 fantasmas (`hunter` + `random`) por el patrón de IA del arcade original, que es el más legible y verificable para lograr "comportamientos distintos".

## Alcance

**In:**

- 4 fantasmas en `GHOST_STARTS`, cada uno con `kind` propio (`blinky`, `pinky`, `inky`, `clyde`).
- `blinky`: apunta a la celda actual de Pac-Man (persecución directa y agresiva).
- `pinky`: apunta 4 celdas delante del sentido de Pac-Man (emboscada).
- `inky`: apunta al doble del vector "Pac-Man + 2 celdas delante" menos la posición de Blinky.
- `clyde`: persigue a Pac-Man si está a más de 8 celdas; si está a 8 o menos, se retira a su esquina.
- Color fijo por `kind`: rojo, rosa, cian, naranja (deja de depender del índice del array).
- Los 4 fantasmas están activos desde el primer frame.
- Se eliminan los kinds `hunter` y `random`.

**Fuera de alcance (futuras specs):**

- Ciclo dispersión/persecución (scatter) por esquinas.
- Estado vulnerable/frightened (no existen pills de poder todavía).
- Puntos por comer fantasma.
- Velocidad de movimiento distinta por fantasma.
- Nombres/rótulos de los fantasmas en el HUD.
- Liberación escalonada desde la pen.

## Modelo de datos

`GHOST_STARTS` pasa de 2 a 4 entradas (`src/js/maze.js`). Cada fantasma gana una celda `home` (esquina base); hoy solo Clyde la usa, se deja lista para la futura spec de scatter.

```js
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'blinky', home: { x: 25, y: 2 } },
  { x: 14, y: 14, kind: 'pinky', home: { x: 2, y: 2 } },
  { x: 13, y: 15, kind: 'inky', home: { x: 25, y: 29 } },
  { x: 14, y: 15, kind: 'clyde', home: { x: 2, y: 29 } },
];
```

En `src/js/game.js`, `createGame` propaga `home` al objeto fantasma y `decideGhost` se generaliza:

```js
function targetFor( game, g ) { /* devuelve { tx, ty } según g.kind */ }
const PINKY_AHEAD = 4;
const INKY_AHEAD = 2;
const CLYDE_CHASE_RANGE = 8;
```

- Inky busca a Blinky por `game.ghosts.find( gh => gh.kind === 'blinky' )`; si no lo encuentra, usa la posición de Pac-Man (fallback).
- El target de Pinky/Inky se recorta al rango del grid antes de medir distancias.
- El esqueleto de `decideGhost` no cambia: filtrar opciones válidas (excluyendo el giro de 180°, con callejón como excepción) y elegir la que minimiza distancia Manhattan al target.

## Plan de implementación

1. `src/js/maze.js`: reemplazar `GHOST_STARTS` (2 entradas) por las 4 con kind clásico y `home`.
2. `src/js/game.js`: propagar `home` en `createGame` al construir los fantasmas.
3. `src/js/game.js`: añadir `targetFor( game, g )` con los 4 targets, recorte de bordes y fallback de Inky; refactorizar `decideGhost` para que elija dirección por distancia al target y eliminar las ramas `hunter`/`random`. Verificación: no hay errores de consola y los fantasmas se mueven.
4. `src/js/render.js`: usar mapa `kind → color` (blinky rojo, pinky rosa, inky cian, clyde naranja) en lugar de color por índice.
5. Verificación manual según criterios de aceptación.

Cada paso deja el juego funcional y es commiteable por separado.

## Criterios de aceptación

- [ ] Aparecen exactamente 4 fantasmas en una partida.
- [ ] Cada fantasma se dibuja con su color clásico (rojo, rosa, cian, naranja).
- [ ] Blinky reduce su distancia a Pac-Man en tramos abiertos (persecución directa).
- [ ] Pinky se dirige a la celda 4 celdas delante de Pac-Man según su dirección.
- [ ] Inky usa la posición de Blinky combinada con el vector a Pac-Man (itinerario distinto al de Blinky).
- [ ] Clyde persigue a Pac-Man cuando está a más de 8 celdas y se dirige a su esquina (2,29) cuando está a 8 o menos.
- [ ] Los 4 funcionan al mismo tiempo y sus trayectorias difieren entre sí al jugar.
- [ ] No hay errores en consola al cargar ni durante la partida.
- [ ] Comer todos los dots sigue terminando en victoria.

## Decisiones

- **Sí:** personalidades clásicas (blinky/pinky/inky/clyde). Es el patrón del arcade original y cumple "uno persigue agresivamente".
- **No:** variantes genéricas (chaser/ambusher/…). Menos fiel y sin referencia conocida para verificarlas.
- **Sí:** objetivo permanente, sin ciclo scatter/chase. Más simple; el ciclo temporal va en otra spec.
- **No:** ciclo dispersión/persecución con temporizadores. Complejidad no pedida.
- **Sí:** los 4 libres desde el inicio. Evita temporizadores y solapamiento en la pen (solo hay 4 celdas en filas 14/15).
- **No:** liberación escalonada / pen clásica. Requiere estados y retardos; futura spec.
- **Sí:** color por kind en vez de por índice. Cada personalidad es identificable y el orden del array deja de ser semántico.
- **Sí:** `home` en los 4 fantasmas aunque solo Clyde lo use hoy. Prepara la spec de scatter sin romper el modelo.
- **No:** conservar `hunter`/`random`. Quedan reemplazados por el conjunto clásico.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Inky queda sin Blinky (orden del array roto) | Búsqueda por `kind` con fallback a la posición de Pac-Man |
| Target de Pinky/Inky cae fuera del tablero en bordes | Recorte del target al rango del grid antes de decidir |
| Los 4 salen agolpados por la misma puerta | Posiciones iniciales escalonadas (filas 14/15) y verificación visual |

## Lo que **no** está en esta spec

- Ciclo scatter/persecución, estado frightened, puntos por fantasma, velocidades por fantasma, HUD por fantasma y liberación escalonada. Cada uno, si llega, va en su propia spec.