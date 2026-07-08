# Spec: THE GAZETTE — a living frontier in a single file

Working title behavior: the app titles itself per seed — "The Dry Gulch Gazette", "The Alkali Springs Courier" — because the central conceit is that you are watching a frontier newspaper write itself. The territory is the simulation; the Gazette is the soul.

## 1. Vision

A procedurally generated Wild West territory (~1860s–1890s) rendered in real-time 3D that you can watch for five minutes or five hours. Zoom from a satellite view of the whole territory down to a saloon porch with laundry on the line. Time runs; harvests come in; freight wagons and stagecoaches crawl the trails; cattle drives raise dust on the horizon; a governor is shot without a clear successor and two cattle outfits take up arms over water rights; cholera follows the stage lines; a legendary outlaw gang notices the bank's vault has gotten heavy; and one day the railroad comes. Everything that happens is printed in a scrolling Gazette in the voice of a frontier newspaper editor, and a cinematic auto-director flies the camera to the story as it unfolds.

Four pillars, in priority order:

1. **Legible emergence.** Systems couple visibly: drought → failed harvest → hunger → lawlessness → range war. A viewer should be able to reconstruct cause from effect just by watching.
2. **Zoom is the reward.** Every altitude has something worth seeing — claims and ranges at 3 km, trade arteries and the rail line at 800 m, Main Street life at 40 m.
3. **The Gazette is the product.** Named people, named places, consequences that persist (ghost towns, grudges, epithets). The sim generates a story you could retell.
4. **Watchable by default.** With zero input it should behave like a nature documentary about a frontier territory.

## 2. Hard constraints

- Single HTML file, vanilla JS + Three.js from CDN (r128 API surface). No build step, no framework, no backend.
- Custom camera controller (no OrbitControls).
- No localStorage/sessionStorage. Persistence is the seed: URL hash `#s=1234567` fully determines the world. "Copy link" shares it. "Export gazette" downloads a .txt.
- Deterministic worldgen via seeded sfc32 RNG with separate streams for generation and history, so intervening in history never changes the map.
- Performance floor: 60 fps on a mid-tier laptop at every altitude; <250 draw calls via instancing and merged geometry; graceful degradation, never a crash.
- Everything procedural — no external textures or models. Vertex colors, tiny generated canvas atlases (brands, wanted posters), and shader work carry the visuals.
- Content note: the simulation portrays settler towns, ranches, and mining camps only. Native nations are deliberately not represented as game mechanics.

## 3. World generation pipeline

Runs once per seed, in order, with a brief "surveying the territory" progress line:

1. **Terrain.** 6×6 km territory. fBm heightmap with domain warping; one mountain spine, rolling lowlands; ~40% of seeds get a badlands edge — terraced red-rock benches and hoodoos along one side (replaces the fantasy coastline).
2. **Hydrology.** Flow carves one major river (always) plus creeks; lakes in basins; the main stem below an elevation cutoff is navigable — settlements near it get steamboat landings.
3. **Biomes.** Moisture × elevation → alkali flat, bottomland farmland, riparian cottonwood, pine upland, rocky peak, prairie grass, sagebrush scrub, red-rock badlands. Drives tree/scrub instancing, ground vertex color, and resource nodes (farmland, timber, ore vein, cattle range).
4. **Settlement siting.** Score cells by water access, flat buildable area, resource adjacency, landing bonus. Place 1 county seat (pop ~2,000), 2–3 towns (300–1,000), 3–5 hamlets/ranch clusters (50–250). The county seat gets a courthouse square and a stockade fort.
5. **Trails.** A* over a cost field (slope² penalty, water-crossing penalty that spawns bridges and fords, forest penalty) connecting all settlements. Trails are the circulatory system — freight, posses, cholera, and road agents all use them.
6. **Main Street & lots.** The widest entering trail becomes Main Street through the town center; 1–2 cross streets and back lanes. Lots pack along streets; the commercial strip (saloon, general store, bank, hotel, blacksmith) fronts Main; church and courthouse anchor a cross axis; houses fill behind. Hamlets are a crossroads, a well, a windmill, and a cluster.
7. **Buildings.** ~800–1,200 at year one (growing with the decades) from ~19 archetypes × 3 tiers (tent/dugout → false-front timber → brick/stone): house, saloon, general store, church, blacksmith/livery, bank, jail, hotel, grist mill (animated wheel on watercourses), grain elevator, warehouse, courthouse, stockade wall/gate, and era-spawned: train depot, water tower, mine headframe, windmill (animated fan). False fronts with painted signs, boardwalks, hitching posts, water barrels.
8. **Names & brands.** Place names from western banks (Dry/Silver/Coyote/Rattlesnake/Alkali + Gulch/Creek/Springs/Flats/Mesa/Bend/Crossing), with a Spanish-flavored bank stronger on badlands seeds. People from period given-name banks. Outfits take a **cattle brand**: modifier (Bar/Rocker/Lazy/Flying/Circle/Double/Slash) + core glyph (letter, digit, diamond, star, horseshoe, spur…), named from its composition — "Lazy J Bar", "Flying W" — rendered to a canvas atlas and used on range overlays, panel chips, ranch signs, and posse guidons.
9. **The cast.** Territorial governor + county officials (sheriff, marshal, judge, preacher, doctor, newspaper editor) + 4–5 outfits (cattle-baron families, a mining magnate, a merchant/banking family), each holding a range and a seat. ~100–140 living notables at any time.

## 4. Rendering, LOD & visual style

Aesthetic: **sun-bleached storybook Americana** — low-poly forms with painterly vertex-color gradients, like the engravings of a frontier newspaper come to life. One directional sun/moon light with a tight camera-tracking shadow frustum, height + distance fog, custom gradient sky dome with sun, moon, stars, and the occasional comet.

- Water: gerstner-lite displacement + fresnel on the river and lakes; the river visibly swells during floods.
- Day/night: full cycle; nights are deep indigo with warm saloon-window glow (per-vertex emissive in the bubble band), pooled lamp lights, campfires.
- Seasons (90 days each, 360-day year): spring greens and a snowmelt-fat river; summer straw-gold (drought pushes to dust); autumn russet cottonwoods; winter snow via normal-up whitening, snowfall, frozen lake tint.
- Weather: clear/overcast/rain/storm/snow (+ dust storms in drought); rain streaks, wet-darkened ground, lightning. Weather is mechanical, not decorative (fire spread, gunfight modifiers, flood meter, tornado spawns).
- Ambient life: stovepipe smoke, buzzards circling where something died, grazing cattle near ranches, **tumbleweeds rolling with the wind**, spinning windmill fans, banner-cloth idle on the courthouse.

LOD bands: Far (>2,200 m) merged color-block masses per district, posses → guidon icons, labels, range tint. Mid (500–2,200 m) instanced archetypes, full low-poly agents, instanced trees. Near (120–500 m) chimneys, signs, false-front detail, wells, wagons, stalls. Bubble (<120 m) window glow, laundry lines, hitching posts, citizens spawn (≤80). Citizens outside the bubble are statistics.

UI skin: aged newsprint (#E8DCC0) with iron-gall ink (#2B2320), rust-red accents (#8B3A2E), gold (#C9A227), outfit colors from brands. Gazette set in an old-style serif (Old Standard TT); masthead and act buttons in wood-type display (Rye). Acts read like headlines: "Strike gold", "Send a twister", "Contest the election".

## 5. Camera & interaction

Custom rig with three modes sharing one smoothed spring-damper transform:

- **Free:** orbit + pan + scroll zoom along a curved altitude path (map-like up high, street-like down low). Range ~5 m to ~4,000 m. Terrain collision.
- **Follow:** click any agent (wagon, stage, drive, posse, train, the gang) → low tracking shot with lead. Click empty ground to release.
- **Director:** see §13.

Raycast click opens the Inspector (§15). Hotkeys: Space pause, 1–5 speeds, H hide UI, C Watch mode, Esc deselect.

## 6. Simulation architecture

- Clock in fractional days; fixed-step accumulator; 1 tick = 1 sim-day. Speeds: pause / 0.1 / 1 / 5 / 30 / 120 sim-days per real second (at max, a year passes in 3 seconds; a governorship in under a minute).
- System order per tick: Weather → Fires → Cholera → Rustlers → Omens → Gang → Railroad → Strikes → Landmarks → Economy/Population → Politics → Trade → Gazette flush.
- Render decoupling: agents store route + departure day; the render loop interpolates positions from fractional sim-time, so motion is smooth at any speed.
- Stability rule: every positive feedback loop ships a predator. Population ↑ → land scarcity. Bank ↑ → gang and rustler attention. Herd ↑ → overgrazing and the Big Die-Up. Boomtown ↑ → the vein always plays out. Outfit power ↑ → federal suspicion. Range war ↑ → federal troops force a peace. Target: 200 unattended sim-years stay in sane bounds and stay interesting.
- Pacing target: a Gazette-worthy event every 15–45 sim-days at defaults.

## 7. Population & notables

Two-layer model:

- **Pools:** each settlement tracks population with birth/death/migration rates modified by food, cholera, war, prosperity, and gold rushes. Pools drive building growth, posse size, tax income, bubble density.
- **Notables:** ~100–140 named characters — officials, outfit families, plus flavor roles (preacher, Doc, a famous gunsmith, an Outlaw Boss when one arises, a doomsayer at Tall Tales High). Notables have age, sex, outfit, 2 traits, relationships (spouse, heirs, grudges), and an agenda line. They marry (alliance-valued), have children, and die by hazard curves plus events (gunfight, cholera, childbirth, the occasional hanging). Deaths and births are Gazette copy; important ones are Director beats.

## 8. Economy & trade

Legibility over realism. Seven goods: **grain, cattle, timber, ore, tools, whiskey, dry goods.**

- Production by geography: bottomland → grain; prairie ranches → cattle; pine → timber; veins → ore; towns convert ore+timber → tools and grain → whiskey; the county seat imports/produces dry goods (until the railroad undercuts it).
- Local prices from stock vs. need; **wagon freighters** (≤36) arbitrage price gaps along trails; **stagecoaches** run scheduled mail-and-gold routes (the premier holdup target); **steamboats** (≤4) trade between river landings; **cattle drives** (1–2/year per outfit) push instanced herds to the best market — later, to the railhead.
- Treasury = the territorial bank: taxes trade and harvests (default 12%); spends on posse wages, bounties, relief, rebuilding. The bank's balance is what the gang watches.
- Hunger cascade: food deficit → stores drawdown → hunger → mortality up, lawlessness up, migration out. Drought, cholera, standoffs, and robbed freight all feed this one pipeline.

## 9. Officials, outfits & politics

- **Governorship:** the territorial governor is appointed ("word from Washington") every 4–8 years or on death/scandal; appointment weight = outfit lobbying + Order. Each governorship is a Gazette era header.
- **Elections:** county elections every 4 years (sheriff, judge, mayors) + on vacancies. Candidates are outfit-backed notables; vote share from outfit influence, candidate traits, and incumbent performance. A margin under 8% with an aggressive loser → **disputed election**: rival claimants, courthouse standoff, Order −15, possible range-war spark.
- **Order (0–100)** on the territory: raised by convictions, hangings of outlaws, festivals (July 4th), rail completion, the gang brought in; lowered by unresolved holdups, vigilance committees, disputed elections, famine, dark omens at Tall Tales High.
- **Outfit standing (0–100 each):** drifts with taxes, water/grazing rights, shared range wars, marriages and business partnerships, slights and grudges.
- Lawlessness per settlement from hunger, taxes, feuds, cholera, gold rushes → saloon riots (small fires, trade pause) → a settlement flips to outfit control at extremes.
- **Range war trigger:** standing <30 on an outfit with muscle + a spark (disputed election, fence-cutting, a murdered kin, drought over waterholes, the "Contest the election" button). Outfits declare by standing, marriage, and grudge. The Ranges overlay animates as brands change.

## 10. Range wars

- **Posses:** raised from settlement pools (strength = f(population, prosperity, fortifications)); a posse is one entity rendered as up to ~100 instanced riders under a brand guidon (icon beyond 800 m). Posses ride trails, eat supply, and are gorgeous in Follow cam.
- **Gunfights:** posses in contact → 2-sim-day abstract resolution in three phases (skirmish/shootout/rout) with muzzle flashes, dust, and casualties trickling both pools. Odds from strength × morale × terrain × weather × leader grit. Losers scatter; named leaders can die or win epithets ("Deadeye Cole").
- **Standoffs:** a posse at a fortified town pitches a visible camp. Countdown = f(fortifications, stores, season); assaults roll on leader boldness. A fallen town is looted (prosperity hit, fire risk, Gazette lament) or occupied (brand flips).
- War score from gunfights, standoffs, and months held; at ±100 or exhaustion → peace: a new governor, an exiled outfit (brand struck through), ceded range, or a hollow truce. A war past a year draws **federal troops** who force the peace. Burned blocks and boot hills persist; battlefields get a marker naming the fight. Scars are content.

## 11. The fates

Each threat couples into existing systems:

- **Cholera:** SIR-lite per settlement; imported by freight and stage arrivals (the trade network is the infection network). Settlements above 15% sick refuse traffic — emergent quarantine that visibly starves trade. Boomtown camps are twice as susceptible.
- **Town fire:** ignition (lightning, saloon riot, looting, a knocked stove, trigger) spreads through building adjacency, biased by density, dryness, wind; rain suppresses. Burned lots → charred ruins → prosperity-gated rebuild.
- **Prairie fire:** a cellular front through grass and scrub, drought-biased, wind-driven; **roads and the river are firebreaks** — fires visibly die at the wagon road. Char fades over a season or two.
- **Flash flood:** snowpack meter; heavy melt + spring storms swell the river, damaging riverside lots and bridges.
- **Drought:** summer event; harvest ×0.4, grass to dust, fire danger up, dust storms at the worst — feeds the hunger cascade.
- **Tornado:** rare storm-spawned funnel that walks a 1–3 km corridor; buildings in the swath splinter, secondary stove fires, agents caught are lost. The path scars the prairie.
- **Locust swarm:** late-summer plague that eats standing grain along its track.
- **The Big Die-Up:** a hard winter kills 30–60% of standing herds → beef scarcity, outfit fury, range-war fuel.
- **Rustlers & road agents:** camps spawn in the breaks when lawlessness is high or war rages; visible stage holdups and cattle rustling; the county posts bounties and sends posses → camp cleared, or a named **Outlaw Boss** emerges if ignored.
- **The gang** (always on): a legendary named outlaw gang lairs in the badlands at worldgen. Wake probability scales with the bank's balance (later, with gold shipments on the rail). A raid is a Director-mandatory beat: riders gallop in, hit the bank or stop a train, and ride out with the gold. The marshal responds per grit — posse showdown (victory = gold recovered + "the man who got —" epithet + Order surge; failure = a dead marshal and a legend) or amnesty/payoff (treasury drain, five quiet years). A fallen gang's seat never stays empty long.
- **Gold strikes:** a prospector finds color in the hills → boomtown rush (tent camp, migration flood, lawlessness, whiskey demand ×3) → the vein plays out → bust → **ghost town**: windows go dark, boards weather grey, the label fades — and it stands forever. The gang dens in one; the railroad might revive one.
- **Omens (Tall Tales ≥ Low):** comet (crosses the sky), eclipse, red moon, ghost lights near the badlands, a white buffalo. At Low they move lawlessness and flavor the Gazette; at High they get teeth (a comet year multiplies event rolls), a wandering doomsayer preaches against the governor, and a **Thunderbird** glides a storm front, throwing lightning-fires. It never takes gold — the gang owns that niche.
- **Tall Tales dial** (Off / Low / High, default Low) lets the same engine serve dime-novel-grounded and weird-West moods.

## 12. The Gazette

A scrolling newsprint panel printing dated entries in frontier-editor voice via a template grammar with slots, tone variants, and occasional editorializing ("this paper has said before that no good comes of a comet"). Entry classes: births/deaths/marriages/appointments, harvests and famines, trade and town charters, war declarations/gunfights/standoffs/peaces, disasters, omens, rustler and gang affairs, gold strikes, railroad progress, festivals.

- Every entry is clickable → the camera flies to the spot (or to the ruin, boot hill, or ghost town that remains).
- Era headers divide the scroll: "THE GOVERNORSHIP OF ABEL CRANE, BEGUN 1871."
- Filter chips (Law / Range / Trade / Fates). Export saves the full archive as .txt — a shareable artifact of the run.
- Fully local, no API.

## 13. The auto-director

An event bus emits beats with priorities (gang raid/train robbery 10, tornado 9, gold strike 9, election crisis 9, range-war gunfight 8, town/prairie fire 8, cholera arrival 7, railroad milestone 7, wedding 6, town chartered 6, stage holdup 5, drive arrival 5, festival 3). The director:

- Holds shots 8–20 s with eased fly-tos; per-category cooldowns prevent fire-fixation; higher priority interrupts lower.
- Shot grammar: wide establishing → push-in; low tracking shots for drives, stages, and construction crews; slow orbit for standoffs and festivals; high wide for range flips.
- When idle, drifts between golden-hour orbits of settlements — occasionally a ghost town at dusk.
- Renders a lower-third beat card with the Gazette line for the current shot.

**Watch mode (C):** hides everything but the date and beat card — the screensaver-documentary mode. Manual camera input politely suspends the director for 30 s.

## 14. Control panel

Left drawer, four tabs. Time always in HUD: ⏸ ▶ ▶▶ ▶▶▶ ▶▶▶▶ → 0 / 0.1 / 1 / 5 / 30 / 120 sim-days per second.

- **Rates** — ×0.25–×4.0 sliders: Harvest yield · Birth/Mortality · Trade volume · Tax rate (5–30%, default 12%) · Cholera virulence/lethality · Disaster frequency · Rustler activity · Outfit aggression · Family fertility · Tall Tales Off/Low/High.
- **Acts** (⌖ = cursor placement): Unleash cholera ⌖ · Start a fire ⌖ · Flash flood · Declare a drought · Send a twister ⌖ · Strike gold ⌖ · Rouse the gang · Raise a rustler camp ⌖ · Assassinate the governor · Contest the election · Broker a marriage · Bless the harvest · Send a comet · Charter a town ⌖ · Grant $5,000 · Raze building ⌖ · Charter the railroad.
- **Overlays** (one at a time): Ranges (brand colors, borders animate in war) · Trade (flow lines incl. rail, width = volume) · Prosperity · Cholera · Lawlessness · Terrain.
- **World:** seed field + Resurvey · Copy link · Export gazette · Labels · Watch mode.

## 15. HUD & inspector

Top strip: date & season glyph · weather icon · governor name + brand chip · bank balance · territory population · time controls. Right: the Gazette. Left: drawer. Everything hides with H.

Inspector (raycast click): buildings (name, type, tier, household, stored goods); settlements (population, prosperity, lawlessness, stores, fortifications, landing/rail flags); notables (brand, age, traits, spouse/heirs, grudges, agenda — outlaws get their wanted poster); posses (strength, morale, supply, leader); wagons/stages/drives/trains (route, cargo, value); the gang (name them — outlaws deserve names — mood, loot, bounty). A compact Outfits panel shows brands, standing bars, and the line of appointment.

## 16. Data model sketch

Plain objects: `World { seed, heightmap, biomes, rivers[], settlements[], trails[], outfits[], notables[], posses[], agents[], strikes[], rail, gang, threats[], clock, treasury, order }` · `Settlement { name, pos, pop, prosperity, lawlessness, stores{7}, fort, buildings[], owner, landing, rail, ghost, infected{S,I,R} }` · `Outfit { name, brand, seat, standing, might, wealth, herd, grudges{} }` · `Rail { chartered, line[], builtM, depots[], trains[], bypassed[] }` · `Event { day, class, priority, pos, actors[], text }` → feeds Gazette + Director. Systems are pure-ish functions over World in tick order; one rand(stream) gateway for all stochastic draws.

## 17. Explicit non-goals

No per-citizen simulation beyond bubble ambience. No Native American representation. No playable gunfights (they resolve abstractly under good dressing — this is a newspaper, not a shooter). No building interiors. No save-states (the seed regenerates year 0; the exported gazette is the record). No diplomacy UI beyond marriages and standings. No frameworks, no server, no assets.

## 18. Build phases (each ships a runnable artifact)

| Phase | Scope | Acceptance |
|---|---|---|
| 1 — The Territory | Gen pipeline, terrain/water/sky, trails & bridges, buildings + LOD, camera, day/night, seasons, seed URL | Fly from 3 km to a saloon porch at 60 fps; two seeds feel like different territories |
| 2 — The Trail | Clock & speeds, economy, freighters/stages/steamboats, cattle drives, population, growth/decay, HUD, inspector, Gazette v1, Rates | 10 min at ▶▶▶: visible growth, ≥15 entries, a drive completes, no runaway values |
| 3 — The Law | Notables, outfits & brands, Order/standing/lawlessness, elections, range wars, posses/gunfights/standoffs, Ranges overlay, political Acts | "Contest the election" → war within a sim-year, visible rides, a peace, a coherent arc |
| 4 — The Fates | Disaster suite, rustlers, gang & Tall Tales, gold strikes & ghost towns, omens, auto-director + Watch mode, remaining Acts/overlays, perf pass | The five-minute test (pre-rail) |
| 5 — The Iron Horse | Railroad charter, visible construction, depots, boom/bypass, trains & robberies, drive retargeting | 200-year run: line completes visibly; a town booms, a town fades, a train is robbed |

## 19. Acceptance: the five-minute test

Defaults, Tall Tales Low, speed ▶▶▶, no input. Within 5 minutes a viewer sees: a season change; ≥8 Gazette entries including ≥1 named-character event; ≥1 visible incident (fire, holdup, twister, omen, festival, drive) that the director covers; and at least one block visibly grow. Within 30 minutes: an election crisis or a range war. After 200 unattended sim-years: population, bank, and outfit count in sane bounds, the railroad long since woven in, at least one ghost town standing — and the territory still producing story.

If the five-minute test passes, the thing is fascinating to watch. Everything above serves that sentence.
