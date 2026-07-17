# BLAZING CHAIN

A DoDonPachi-inspired vertical bullet hell. The entire game is one file: `index.html`. Deployed via GitHub Pages.

## Hard constraints

- Single file, zero dependencies. No external assets, fonts, or libraries. No build step.
- Hi-score persists via localStorage key `blazingChain.hi`, always wrapped in try/catch with an in-memory fallback so the file still works where storage is blocked (claude.ai artifact preview). Nothing else may touch storage, and never without the guard.
- No em dashes anywhere, including comments.
- Pixel aesthetic: 240x320 internal canvas, integer-scaled, `imageSmoothingEnabled=false`, CSS scanline overlay.
- Fixed 60fps timestep loop. Do not switch to variable delta.

## File layout (in order)

1. CSS shell and scanlines
2. TUNING section: `TUNE` (feel), `DIFF` (speed multipliers, revenge speed, `DIFF.bossHp`), `ECON` (credits, lives, bombs, extends, bonuses), `JUICE` (hit-stop and pulse frames), `TARMAC_S1`/`TARMAC_S2` palettes, then the CHEAT block (URL params ?stage= ?loop= ?boss= ?pow= ?inv= ?credits= ?debug=; debug keys I/B/L/G). Any cheat sets `cheatUsed`: `saveHi` refuses to persist and a DEBUG tag shows on the HUD. All global tunables belong here; per-enemy hp and pattern timings deliberately stay with their enemies
3. Input: keyboard (`keys` by e.code) and pointer (drag=move+autofire, second finger=laser, BOMB touch button)
4. WebAudio synth: `blip(f0,f1,dur,type,vol,at)`, `noiseHit(dur,vol,dark,at)`, `sfxBossDown` fanfare, laser hum
5. Pixel sprites via `sprite(rows)` string maps (equal row widths required); `sprMid` built in `makeMid()`
6. Bullet patterns: `eb`, `ring`, `fan`, `aimAng`. `eb` applies a 1.15x speed multiplier in stage 2.
7. Enemy spawns and `updateEnemies` (kinds: bee, rain, heli, tank, spin, orb, mid)
8. Bosses: `spawnBoss(kind)`; kind 1 QUEEN VESPA, kind 2 KING LUCANUS (stag beetle). `DIFF.bossHp` per kind, per-boss hitbox `hw`/`hh`, patterns branch in `updateBoss`, draw in `drawBoss`/`drawBoss2`.
9. Stage scripts: `buildScript(st)` for stage 1, `buildStage2()` for stage 2; `at(sec,fn)` and `wave(t0,n,gap,fn)` off `stageFrame`
10. Player, shot/laser/bomb, `collide`, chain logic
11. Powerup items, background (`TARMAC` with `TARMAC_S1`/`TARMAC_S2` presets), drawing, HUD, loop

## Key mechanics (do not break)

- Chain (DDP GPC style): kill sets `gauge=100`, drains `TUNE.chainDrain`/frame, only `TUNE.laserSustain`/frame while lasering a target. Kill value = base x min(chain+1, 300).
- Weapons: tap Z = spread shot, hold Z past 14 frames = laser + focus movement. Laser damage scales with power.
- Power 0..4 via P items; option pods at 4; death costs one level and resets bombs to 3.
- Drops: tanks alternate P/B, spinners drop P, midboss drops P+P+B. Surplus converts to score.
- Orbs (stage 2) burst into a 10-bullet ring on death: kill them early or at a distance.
- Stage flow: beating boss 1 plays `sfxBossDown`, awards STAGE 1 CLEAR bonus, calls `startStage2()` (script reset, palette shift, bullets 15 percent faster). Beating boss 2 on one credit starts loop 2; otherwise ALL CLEAR. Score, chain stats, power, bombs, lives persist across stages.
- Credits and continues: `credits` (starts 3, C key or COIN button inserts, infinite allowed). Death opens a 9 to 0 CONTINUE countdown (`S.CONT`). Continuing costs a credit, resets score, and stamps the ones digit with `continuesUsed` (capped at 9 for the marker). INVARIANT: every score award must be a multiple of 10 or the marker breaks.
- Continuing sets `loopLocked=true`: loop 2 is only reachable by a one credit clear.
- Loop 2: bullets a further 1.2x faster (multiplies with the stage 2 1.15x), revenge bullets (dead enemies fire 1 or 2 aimed shots unless a bomb is active), and KING LUCANUS gains a 4th TRUE FORM phase (five arm spiral storm). Clearing it sets `trueClear` and shows TRUE ALL CLEAR.
- Player hitbox is tiny (`TUNE.hitR`), shown as a dot while lasering.
- Ships: `SHIPS` array in TUNING (TYPE-A default; TYPE-B faster, wider shot, weaker laser). `shipType` selects; arrows on the title switch once TYPE-B is unlocked.
- Boss timers: every phase lasts `DIFF.bossPhaseSec`; expiry escalates to the next phase (dodging reveals all patterns), final-phase expiry makes the boss withdraw (no bonus, no kill, stage advances normally). Timer shows under the boss bar.
- Pacifist ("dot eater") runs: `pacifist` starts true, broken by any shot, any laser engagement, any bomb, or continuing. A pacifist, no-continue loop 1 clear (boss 2 resolved while `!loopLocked`) unlocks TYPE-B, persisted under `blazingChain.typeB` with the same storage guard; cheat runs (`cheatUsed`) can never earn it. A small DOT EATER tag shows while a run is still pure.

## Visual readability rules (the design system)

- Brightness hierarchy: enemy bullets brightest (pink/cyan with white cores), player and enemies mid, scenery darkest.
- Stage 1 scenery: embedded hull tileset. `ATLAS_B64` is an 18-cell 48px atlas baked from an external tileset sheet (not in the repo); cyan glows were pre-toned at bake time (pixels with b>120 and b>1.5r scaled by 0.75/0.6/0.55) so terrain cyan cannot mimic type-1 bullets. `drawBgHull` lays it out world-anchored: panel edge rails, three centre lanes whose plate family shifts every 4 rows, a strut bulkhead band every 6 rows, a 2x2 blast door every 9 rows. Regenerating the atlas requires the original sheet plus that bake recipe.
- Stage 2 scenery: violet tarmac tile grid via `TARMAC_S2` (`drawBgTarmac`). `drawBg` dispatches on `stage` and falls back to tarmac until the atlas image decodes.
- Every enemy bullet draws a 1px dark rim (#0d0e18) behind its colour and white core, which keeps small bullets legible on the brighter hull terrain.
- Ground units (tank, mid) get drop shadows; flyers never do.
- Items are yellow/green diamonds with dark cores and letters so they cannot read as bullets.
- Juice conventions: brief hit-stop via `freezeT` (3 frames on big kills, 6 on boss phase changes, 10 on boss death), enemy bullets bloom white for their first 2 frames, ships fly in from the bottom of the screen on spawn and respawn, the chain counter pulses on increment, bosses announce their name on arrival.

## Verification

- Syntax: `sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/g.js && node --check /tmp/g.js`
- Sprite maps: every row in a `sprite([...])` call must have identical length.
- Playtest: chain gauge feel, bullet readability at max density, the stage 1 to 2 transition, and the continue countdown plus loop 2 entry.

## Companion files

- `boss-sound-test.html`: standalone audition page for the boss-defeat sound (variant A is live in the game).

## Backlog candidates

- Hidden bee medals revealed by laser sweep, escalating collect bonus (DonPachi signature).
