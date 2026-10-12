# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + Canvas 2D. Sin `package.json`, build, linter ni tests. UI y README en español.

## Ejecutar

- Abrir `index.html` directamente, o servir estático: `python3 -m http.server 8000` → `http://localhost:8000`.
- No hay comando de test ni de lint. Verificar cambios jugando en el navegador.

## Arquitectura

Tres archivos: `index.html` (DOM + dos canvas), `style.css`, `game.js` (toda la lógica, script clásico con `'use strict'`, sin módulos).

- **Estado global mutable** en `game.js` (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`…). `init()` lo reinicia; el botón Reiniciar llama `init()` directamente.
- **Codificación del tablero**: `board[row][col]` es `0` (vacío) o `1–7`, que indexa a la vez `COLORS` y `PIECES`. Las formas de `PIECES` usan su propio índice como valor de celda, así que añadir/reordenar piezas exige tocar ambos arrays y `randomPiece()` (`Math.random() * 7`).
- **Ciclo de pieza**: `loop` (rAF) o input → `lockPiece()` → `merge()` + `clearLines()` + `spawn()`. `spawn()` llama `endGame()` si la pieza nueva colisiona.
- **Rotación**: `rotateCW` devuelve una matriz nueva; `tryRotate` prueba kicks horizontales `[0, -1, 1, -2, 2]` y descarta el giro si todos colisionan.
- **Puntuación/velocidad** en `clearLines`: `LINE_SCORES[n] * level`; nivel = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`. Soft drop +1/fila, hard drop +2/celda.
- **Dibujo**: `draw()` repinta todo cada frame (grid, tablero, ghost con alpha 0.2, pieza actual). `drawNext()` solo se llama desde `spawn()`; el canvas de siguiente pieza es 4×4 bloques de 30px.

## Acoplamientos a respetar

- `COLS`/`ROWS`/`BLOCK` en `game.js` deben coincidir con `width`/`height` de `<canvas id="board">` en `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`, hoy 300×600). No se derivan automáticamente.
- `game.js` obtiene elementos por id (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`); renombrar ids en el HTML rompe el script al cargar.
- El overlay se reutiliza para PAUSA y GAME OVER (cambia `overlay-title`/`overlay-score`, clase `hidden`).
