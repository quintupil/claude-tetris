# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es

Tetris en JavaScript vanilla con HTML5 Canvas. **Sin dependencias, sin build, sin tests.** No hay `package.json`, ni bundler, ni transpilador. Solo tres archivos: `index.html`, `style.css` y `game.js`.

## Ejecutar

Abrir `index.html` directamente en el navegador, o servir la carpeta con cualquier servidor estático (`python3 -m http.server 8000`, `npx serve .`). No hay comandos de build, lint ni test.

## Arquitectura

Toda la lógica vive en `game.js` (~300 líneas, `'use strict'`, sin módulos ni clases: funciones globales + estado en variables `let` a nivel de módulo). Puntos clave para no romper nada:

- **Estado global mutable**: `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`. `init()` los reinicia todos y es el punto de entrada (se llama al cargar y desde el botón *Reiniciar*).

- **Representación de piezas**: `PIECES` y `COLORS` están **indexados desde 1** (el índice `0` es `null`). Cada celda del tablero y de las piezas guarda ese índice de color 1–7; `0` = celda vacía. El tipo de pieza y su color son el mismo número.

- **Rotación**: `rotateCW` transpone + invierte filas. `tryRotate` aplica *wall kicks* probando desplazamientos `[0, -1, 1, -2, 2]` antes de descartar el giro.

- **Game loop** (`loop`): un único `requestAnimationFrame`. Acumula `dt` en `dropAccum` y baja la pieza cuando supera `dropInterval`. Pausa/reanudación se gestiona con `cancelAnimationFrame(animId)` + rearranque del loop, no con un flag dentro del loop.

- **Ciclo de vida de una pieza**: `spawn()` mueve `next → current` y genera una nueva `next`; si la nueva pieza ya colisiona al aparecer → `endGame()`. `lockPiece()` = `merge()` (fija la pieza en `board`) + `clearLines()` + `spawn()`.

- **Render**: `draw()` limpia y repinta todo cada frame (grid → tablero → *ghost* con `alpha 0.2` → pieza actual). `drawNext()` pinta la vista previa en su propio canvas. Ningún estado se lee del DOM; el DOM solo refleja el estado vía `updateHUD()`.

## Restricción importante al modificar dimensiones

`COLS`, `ROWS` y `BLOCK` (arriba de `game.js`) definen el tablero. Si los cambias, **debes** ajustar a mano `width`/`height` del `<canvas id="board">` en `index.html` para que sigan siendo `COLS*BLOCK` × `ROWS*BLOCK` (hoy 300×600). No se calculan automáticamente.

## Convenciones

- Todo el texto visible al jugador está en **español** (overlays, HUD, README).
- Los IDs del DOM que `game.js` busca por `getElementById` viven en `index.html`: `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. Renombrar uno exige tocar ambos archivos.
