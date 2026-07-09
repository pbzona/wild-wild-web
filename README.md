# The Gazette — a living frontier in a single file

> **Credit:** this is a Wild West re-imagining of [ANNALS — a living kingdom in a single file](https://github.com/emollick/annals-kingdom) by **Ethan Mollick**. The architecture and the "watch a history write itself" idea are his; this project converts that fantasy kingdom into a frontier territory.

A procedurally generated Wild West territory that lives on its own — harvests and cattle drives, elections and range wars, cholera and prairie fires, gold rushes that boom into tent camps and bust into ghost towns, one legendary outlaw gang with its eye on the bank, and, when the territory has proven itself, the railroad — while its newspaper writes itself in the margin. Watch it like a nature documentary, or reach in and turn the dials of fate.

Everything is one HTML file. No build step, no backend, no saves — the territory regenerates deterministically from a seed in the URL hash, so a link *is* a world: share `#s=your-seed-here` and the recipient surveys the exact same territory, down to the last tumbleweed.

## What lives in it

- **A surveyed territory** — fBm terrain with a mountain spine (and, on some seeds, terraced red-rock badlands), rivers that carve valleys, biomes from alkali flat to pine upland, wagon roads found by pathfinding, towns platted street by street with false fronts and boardwalks, cattle brands drawn outfit by outfit
- **A working economy** — seven goods, prices from scarcity, freight wagons and stagecoaches you can follow down the roads, stern-wheelers on the navigable river, cattle drives raising dust to the best market
- **The law and those against it** — a territorial governor appointed from Washington, county elections that get disputed, outfits with brands and grudges, range wars fought by posses, rustler camps, and one named gang that watches the bank's balance
- **The fates** — weather with mechanical teeth, drought and dust, flash floods, twisters that walk a line of ruin, cholera on the trade routes, locusts, killing winters, prairie fires that die at the wagon road
- **Gold** — strikes that found boomtowns overnight, migration rushes, lawless camps, played-out gravel, and ghost towns that stand forever after
- **The Iron Horse** — a railroad chartered mid-history that visibly grades and lays track across the map, opens depots, revives the dead, starves the bypassed, and gets robbed
- **An auto-director and Watch mode** — leave it alone and the camera drifts to whatever story is breaking, with a caption card; press `C` and the territory narrates itself
- **The Gazette** — every event printed with its date in a frontier editor's voice, filterable, exportable, with a Ledgers view of souls, bank, and prosperity across the decades
- **Acts of Providence** — a board of interventions: cholera, fire, twisters, gold strikes, assassinations, contested elections, or the railroad's charter, exactly when and where you click

## Running locally

Open `index.html` in a browser, or:

```
node server.js
```

and visit `http://localhost:8545`. The only network dependency is three.js r128 from a CDN.

## Controls

| Desktop | Touch |
|---|---|
| drag to orbit | one finger orbits |
| wheel / pinch to zoom | pinch zooms |
| right-drag to pan | two fingers pan |
| double-click to travel | double-tap travels |
| click to inspect | tap inspects |

`WASD` pan · `space` pause · `1–5` speed · `T` ranges overlay · `C` watch mode · `G` director · `L` labels · `H` hide the print shop · `F` fps · `Esc` release

## How it's built

Vanilla JavaScript and three.js r128, ~7,000 lines in one file. Four seeded RNG streams (worldgen, detail, ambient, history) keep generation and simulation deterministic and independent — the same seed always yields the same territory. Rendering is triangle-soup geometry merged per settlement with one patched Lambert material carrying seasons, snow, drought, burn scars, and window-glow, chunked instanced flora, and a canvas-texture overlay plane: the whole territory draws in well under 100 draw calls at 60 fps. A debug API rides along at `window.GAZETTE` (`simDays(n)`, `fate(name, x, z)`, `rail()`, `gang()`, …) for driving the simulation headlessly.

The design spec is in [SPEC.md](SPEC.md).

## License

[MIT](LICENSE)
