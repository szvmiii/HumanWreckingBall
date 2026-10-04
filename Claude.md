# Claude.md — Human Wrecking Ball (Roblox)

Working context for Claude. Update after every change so the next session knows what we are doing right now.

- **Studio place:** `[💥] Human Wrecking Ball` (placeId `139473074201684`)
- **Last updated:** 2026-10-04

---

## Current focus

**Booth visual rework ("Village Stalls") — DONE and deployed, awaiting user review.**

User feedback that started this: the booths were ugly, didn't fit the map's vibe, and almost nothing changed between upgrade levels. We rebuilt `BoothBuilder` from zero.

### What is true right now
- `ServerScriptService.BoothBuilder` is the **new v2 builder** (~32 KB).
- Old builder is saved at `ServerStorage.BoothBuilder_V1_Backup` (the previous "candy box" version). `ServerStorage.BoothBuilder_PreCozy` is an even older backup — leave both alone.
- Verified: all 4 kinds × all 5 levels build with no errors (scratch test, since deleted). Verified in a real playtest that `PlotService` builds all four booths on Plot1, each with an `UpgradeAnchor` + working ProximityPrompt ("Upgrade to Lv 2 (free)"), and the console is clean.
- **Not yet verified:** clicking the upgrade prompt in-game to watch L1 → L5 rebuilds live (tool can't fire prompts). The `build()` path is identical, so low risk, but worth one manual click-through.

---

## Map / vibe reference

Warm cozy village: timber, terracotta, cobble, grass, warm lantern light. Palette used by the booths (sampled from the gate, fences, lantern posts):

| Role | RGB |
|---|---|
| Wood dark / mid | 108,74,50 / 160,120,82 |
| Terracotta (gate roof) | 184,92,84 |
| Cream / path | 190,172,144 · 232,218,190 |
| Stone | 122,106,88 |
| Lantern glow | 255,200,120 |
| Grass | 110,152,80 |

Station accents (muted so they sit in the palette): Sell green `104,152,104`, Smelter ember `204,116,66`, Shop blue `84,130,182`, Forge purple `140,98,168`.

---

## Contracts that must NOT break (PlotService depends on them)

- `BoothBuilder.MAX_LEVEL` (= 5)
- `BoothBuilder.build(kind, cf, level, parent)` → returns a `Model` named `kind .. "Booth"` (`SellBooth`, `SmelterBooth`, `ShopBooth`, `ForgeBooth`)
- Model contains a part named **`UpgradeAnchor`** (PlotService attaches the ProximityPrompt to it; it must keep `CanQuery = true`)
- Model attributes `Station` and `Level`
- Footprint stays **30 × 24 studs** at every level (plot slots are laid out around it). Height may grow.
- `cf` = platform-bottom center; `LookVector` = booth front. Local space: front = −Z, floor top = y 1.
- Parts named `InteractZone` (invisible, in front) and `Platform` (PrimaryPart) are kept.

Related: `ServerScriptService.PlotService` (calls `BoothBuilder.build`, owns upgrade logic, `UPGRADE_COSTS`, `DEV_FREE_UPGRADES = true`). Booth slots are `Plot*/Slot_*` models; `slot.Origin.CFrame` is passed as `cf`.

---

## Level progression (every level changes silhouette/materials, not just color)

| Lv | Look |
|---|---|
| 1 | Striped canvas lean-to awning, plank walls, plain counter, a few goods, small unlit lanterns |
| 2 | Timber-shingle gable roof, accent fascia, hanging banners, cloth runner on counter, more goods, lit lanterns |
| 3 | Terracotta roof, plaster + half-timber back wall, stone footings + stone counter skirt, flower planters, bigger lit lanterns, signature prop |
| 4 | Carved pediment with emblem + finials, bunting, iron-banded posts, rarest prop |
| 5 | Gold trim everywhere, glowing cupola with gold finial + sparkles, fairy lights, fireflies, embers/sparks |

Hanging sign board replaces the old floating text: shows station name + 5 level pips (gold = reached).

Signature props per station (grow with level):
- **Sell:** gold ingot pyramid, coin chest + coin stacks, sacks/crates → L3 balance scale → L4 iron vault → L5 sparkling gold hoard
- **Smelter:** stone furnace + chimney with smoke, counter anvil, copper ingots → L2 coal bin, hot ingot, arch → L3 crucible → L4 bellows → L5 embers, gold chimney band
- **Forge:** floor anvil w/ hot ingot, hearth, grindstone, hammer rack → L2 quench barrel → L3 crossed swords → L4 rune strip → L5 glowing glyphs + sparks
- **Shop:** back shelves w/ crates, counter cannon + cannonballs → L2 second shelf + jars → L3 glass display case → L4 barrels → L5 glowing crystal on pedestal

Part counts per booth: ~60 (L1) up to ~170 (L5). 16 booths max (4 plots × 4) → fine.

---

## Open items / ideas (not started)

- [ ] Manually click through upgrades L1 → L5 in a playtest and eyeball each station in the real map lighting.
- [ ] L1 awning stripes are fairly saturated (purple/orange). If they still feel loud next to the map, mute `ACCENT` colors slightly.
- [ ] Optional: small "level up" effect (puff/sparkle burst) when a booth rebuilds — would live in `PlotService.buildStation`, not in the builder.
- [ ] Optional: roof gable ends are open (exposed truss). Could add triangular infill if it looks hollow from the side.
- [ ] `PlotService` has `DEV_FREE_UPGRADES = true` and a TODO for the Coins system — unrelated to this task, left untouched.

## Rollback

To revert booths: copy `ServerStorage.BoothBuilder_V1_Backup.Source` into `ServerScriptService.BoothBuilder.Source` (Edit mode). No other scripts were modified.

## Notes for next session

- Studio must be in **Edit** mode to modify scripts (`multi_edit` / `Source` writes); during Play only the Server/Client datamodels are available.
- Verify visuals with `screen_capture` (signs/SurfaceGuis can take a moment to stream in — retake before assuming text is missing).
- Test builds go in a scratch folder far from the map (e.g. `workspace._BoothTest` at ~(1000,100,1000)); delete it afterwards.
