# AGENTS.md

## What this is

**The Gazette** — a procedurally generated Wild West territory sim that runs entirely in one HTML file. A Wild West re-imagining of [Ethan Mollick's ANNALS](https://github.com/emollick/annals-kingdom). The whole app — worldgen, Three.js rendering, simulation, UI — lives in `index.html` (~7,500 lines, one IIFE). There is deliberately no build step, no framework, no backend, no localStorage. The seed in the URL hash (`#s=...`) fully determines the world.

| File | Role |
|---|---|
| `index.html` | The entire app. The only file that ships. |
| `SPEC.md` | The design document. Read it before changing systems; update it when behavior changes. |
| `server.js` | Static dev server on **:8545**. |
| `PLAN.md`, `example-impl/` | The original ANNALS prompt and implementation — **reference only, never modify**. |

## Run & verify

```sh
node server.js                  # http://localhost:8545/#s=1234
```

Syntax-check the inline script without a browser:

```sh
node -e "const s=require('fs').readFileSync('index.html','utf8').match(/<script>\n([\s\S]*)<\/script>/)[1]; require('fs').writeFileSync('/tmp/g.js',s)" && node --check /tmp/g.js
```

Drive it headlessly (e.g. via agent-browser) with the **`window.GAZETTE`** debug API:

- `GAZETTE.ready`, `stats()` (fps, draw calls, buildings, chron count), `seed(s)`, `cam()`
- `simDays(n)` — runs n whole sim-days synchronously (no rendering needed); `step(n)` — n manual frames
- `fate(name, x, z)` / `act(name)` — trigger any disaster or political act (`'fire'`, `'twister'`, `'gold'`, `'gang'`, `'contest'`, `'assassinate'`, `'rail'`, …)
- `goto(x,z,alt,snap)`, `gotoSettlement(idx|name)`, `tod(0..1)`, `speed(0..5)`, `watch(bool)`
- Inspectors: `rail()`, `gang()`, `outfits()`, `officials()`, `strikes()`, `world` (raw `W`)

Set `window.__fastForge = true` before reforging to skip loading-screen delays. Boot logs a one-line world census — same seed must produce a byte-identical line (determinism check).

**Sanity bars:** `simDays(3600)` → population within ±3× and no NaN; `simDays(72000)` (200 years) → pop/treasury/outfits bounded, still producing story; draw calls < 250 at any altitude (typically 45–100). The full acceptance criteria ("five-minute test") are in SPEC.md §18–19.

## Architecture (order inside index.html)

CSS → HTML skeleton → one `<script>` IIFE: **Part 0** foundations (rng, perlin, names, palettes, patched material, GeoBuilder) → **worldgen** (`GEN_STEPS`: terrain → hydrology → biomes → settlements → roads → town plans → buildings → flora → outfits) → **rendering** (sky/water shaders, terrain, roads, building archetypes, settlement meshes, flora instancing) → **simulation** (gazette bus, economy/agents, politics/wars, fates, railroad) → camera → sky/seasons frame → HUD/UI → forge & main loop → debug API.

Global world state is `W`, rebuilt per seed in `forgeWorld()`. Discrete logic runs in `tickDay(day)` in a fixed system order; `simAdvance` is a fixed-step integer-day accumulator.

## Invariants — break these and things rot quietly

1. **Determinism.** Four seeded RNG streams: `gen` (worldgen), `det` (mesh cosmetics), `amb` (ambience), `hist` (everything in the simulation). Sim logic must draw only from `W.rng.hist` via `pickH/chance/rr/ri`. `Math.random` is allowed only in pure render/director code. History must never perturb the map.
2. **Sim vs render split.** State changes happen only in whole-day `tickDay`; the render loop interpolates from fractional `W.clock.day`. Sim-side proximity checks must compute positions with `routePos(...)` — never read `mesh.position` (stale during headless `simDays`).
3. **The shared material.** Every `InstancedMesh` using `MAT.world` **must** call `setColorAt` on all instances, and every merged geometry needs `aGlow/aAut/aBurn` attributes (`gbToGeometry` provides them). Violations crash seed-dependently.
4. **One event bus.** All narrative goes through `chron(cls, text, x, z, pri)`. Class (`law/frontier/trade/fate/myth/rail/flavor/year/era`) drives filter chips and colors; `pri ≥ 4` with coords makes it a director beat. Write entry text in the frontier-newspaper voice (headline, em-dash, dry wit) — match the `T` templates.
5. **Mesh rebuilds are queued.** After mutating `s.buildings`/`s.props`/`s.ruins`, call `queueRebuild(s)` — never rebuild synchronously in a tick.
6. **Every positive feedback loop ships a predator** (treasury → gang, herds → die-up, boomtowns → bust…). If you add a growth mechanic, add its check. Verify with a 200-year run.
7. **Budget.** < 250 draw calls, 60 fps target, graceful degradation ladder in `frame()`. Prefer merged triangle soup (GeoBuilder) or instancing over new meshes; never add per-entity lights.

## Conventions

- Plain JS, no classes-for-their-own-sake; systems are functions over `W` slotted into `tickDay`'s order.
- 360-day year, day 0 = March 1, 1866 (`dateStr`, `seasonOf`). Notables are id-indexed via `notable(id)`; outfit ids are array indices (`-1` territory office, `-2` townsfolk).
- Content boundary: no Native American representation (see SPEC §2); keep dark subject matter (hangings, vigilantes) clinical, in-period, and brief.
