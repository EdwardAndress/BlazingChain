# BLAZING CHAIN

A single-file, DoDonPachi-inspired vertical bullet-hell shooter. The entire game — engine, sprites, audio, and one hand-scripted stage with a boss — lives in [index.html](index.html) with no build step and no dependencies.

## Play

Open `index.html` in any modern browser, or play the hosted build via GitHub Pages.

The canvas renders at a fixed 240×320 and scales up to fit the window (integer scaling when possible, with a CRT scanline overlay).

## Controls

### Keyboard
| Key | Action |
| --- | --- |
| Arrows / WASD | Move |
| Z (tap) | Shot |
| Z (hold) | Laser — focuses fire and slows the ship for precise dodging |
| X | Bomb |
| P | Pause |
| M | Mute |
| Z / Enter | Start / restart (on title & game-over screens) |

### Touch
- **Drag** anywhere to move the ship (autofires while dragging).
- **Second finger held** activates the laser.
- **BOMB** button, bottom-right.

## Mechanics

- **Tiny hitbox.** The player's core collision radius is deliberately small (DDP style) — trust the dodge.
- **Chain gauge.** Keep chaining kills to build the gauge; it drains over time and drains faster while the laser bites an enemy. Chain length feeds your score multiplier.
- **Laser vs. shot.** Tapping fires a spread; holding pours a sustained laser at the cost of mobility.
- **Bombs** clear the screen and buy you a moment of safety.
- **Extends** are awarded at 1,000,000 and 3,000,000 points.
- Beat the boss to reach **STAGE CLEAR**.

Enemies include bees, rain-drones, helicopters, tanks, spinners, a mid-boss, and the stage boss.

## Tuning

Game feel is centralised in the `TUNE` object near the top of the script (move speeds, fire cooldown, laser DPS, chain-drain rates, hitbox radius, bullet cap). Adjust those knobs to rebalance without touching the rest of the engine.

## Structure

Everything is in one file. The script is organised top-to-bottom roughly as:

1. Config & helpers
2. Game state
3. Input (keyboard + pointer/touch)
4. Stage script (`at(sec, fn)` schedules spawns on a timeline)
5. Update loop (player, enemies, bullets, collisions, chain)
6. Background (a scrolling installation-tarmac motif)
7. Drawing (sprites, HUD, screens)
8. Audio (synthesised via the Web Audio API)

## Deploy

Served as a static site from GitHub Pages — `index.html` at the repo root is the entry point.
