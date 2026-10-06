# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. Sin dependencias, sin `package.json`, sin build, sin tests ni linter. El README (en español) documenta controles y mecánicas; la UI y los comentarios están en español.

## Ejecutar

Abrir `index.html` directamente o servir estático: `python3 -m http.server 8000` y visitar `http://localhost:8000`.

## Arquitectura

Tres archivos: `index.html` (DOM + canvas), `style.css`, y `game.js` (toda la lógica, ~300 líneas, un único script global con `'use strict'`).

- **Estado global** en `let` a nivel de módulo (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`…); `init()` lo reinicia todo y también sirve para el botón de reinicio.
- **Tablero**: matriz `ROWS × COLS` con `0` o índice de color 1–7. El mismo índice sirve en `PIECES` y `COLORS` (índice 0 = `null`), así que añadir una pieza exige tocar ambos arrays y `randomPiece` (que usa `Math.random() * 7`, sin bolsa de 7).
- **Flujo**: `loop(ts)` (rAF) acumula `dropAccum` y baja la pieza al superar `dropInterval`; `lockPiece()` = `merge` → `clearLines` → `spawn`. `spawn()` llama a `endGame()` si la nueva pieza colisiona. Pausa y game over cancelan el rAF (`cancelAnimationFrame(animId)`); reanudar relanza `loop` manualmente. Ojo: si el game over se dispara desde dentro de `loop` (vía `lockPiece` → `spawn` → `endGame`), el `requestAnimationFrame(loop)` del final de `loop` vuelve a armar el bucle, que sigue corriendo con `gameOver = true` (solo el `keydown` lo respeta); `init()` lo corta con su propio `cancelAnimationFrame`.
- **Rotación**: `rotateCW` + wall kicks simples `[0,-1,1,-2,2]` en `tryRotate` (no SRS).
- **Puntuación/nivel**: `LINE_SCORES[n] * level`; nivel = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`. Soft drop +1/celda, hard drop +2/celda.
- **Dimensiones acopladas**: si cambias `COLS`/`ROWS`/`BLOCK` en `game.js`, hay que actualizar `width`/`height` del `<canvas id="board">` en `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`). El canvas `next-canvas` (120×120) usa bloques de 30 px en una rejilla 4×4.

## Particularidades

- `index.html` incluye un `<script>` inline antes de `game.js` que parchea `KeyboardEvent.prototype.code` para que las flechas devuelvan siempre `ArrowUp/Down/Left/Right` (fix de teclas de dirección en layouts/entornos donde `code` falla). `game.js` depende de `e.code`; no lo quites ni lo muevas después de `game.js`.
- El `keydown` handler llama a `updateHUD()` tras cada tecla, y `softDrop` también; ten en cuenta que `hardDrop` no actualiza el HUD por sí mismo.
