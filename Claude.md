# Claude.md — Human Wrecking Ball (placeId 139473074201684)

Working notes so any session can pick up where we left off.
**Update this file after every change.** Last updated: 2026-10-03.

> There is also an old `Claude.md` (different capitalisation) in the repo with the v7 low-poly art direction
> (no lamps/flowers/textures). It is OUTDATED and contradicts the cozy direction below. Delete it so only this file remains.

---

## What we are doing right now

**Cozy but REALISTIC restyle in the spirit of the Roblox game "Build a Haven"** so the game attracts players.
(Owner asked on 2026-10-03. First pass was pastel/pink and too bright: owner said "not candy pink, more realistic, lighting too bright".
Second pass (same day) uses natural earthy colours, real materials and dim warm lighting. I could not find reliable info or screenshots of the real
game. If the owner shares reference screenshots, re-tune the palette.)

The map layout is unchanged: circular island, "+" shape, hub plaza in the centre, four arms. Each arm = plot on the
inner side, then that plot's lane going outward. Only colours, materials, shapes, lighting and decor changed.

### Status

| Item | State |
|---|---|
| "+" layout, plots inner / lanes outer, plot 120 x 120, lane 80 x 720 | DONE (unchanged) |
| Natural dim warm lighting (v2: no pink haze, low brightness) | DONE (`Gen.applyLighting`, runs inside `build()`) |
| Natural earthy palette + real materials (Cobblestone, Wood, Slate, Rock, Limestone, Grass) | DONE (v2) |
| Hub: cream plaza + pink ring, golden autumn wishing tree with falling leaves + fairy lights, 4 benches, 4 lanterns | DONE |
| Plots: grass yard, wood booth pads, wooden picket fences with fairy lights, flower beds, round shrubs, lanterns, arch with round finials | DONE |
| Lanes: natural biome floors (real materials), limestone walls with rounded lane-coloured wood rail + fairy lights, lanterns at the cannon pad | DONE |
| Rim wall: slate base, earth cliff, grass top | DONE |
| Decor rebuilt from scratch: round trees (30% autumn gold), wildflower bushes, slate boulders, 12 firefly zones | DONE |
| BoothBuilder: restored to the earlier natural-colour version (identical to `BoothBuilder_PreCozy`) | DONE |
| Numeric checks: PlotService contract, lane attributes, booth build levels 1-5 for all 3 kinds | DONE, all good |
| **Visual check by eye** (screenshots failed again: helper hit max_tool_calls) | **NOT DONE, look in Studio** |
| Playtest: join, claim plot, spawn, booth build, upgrades | NOT DONE |
| Lane wall blocks / cannon / wrecking-ball gameplay | NOT STARTED (no script yet) |

---

## Art direction (cozy + realistic, decided 2026-10-03)

- **Owner feedback:** NOT candy / pink / pastel. More realistic. Lighting was too bright. Keep it cozy through warm light, wood, stone, plants.
- Natural, slightly muted colours. Real materials: `Grass` (ground, yards, caps), `Cobblestone` / `Pebble` / `Slate` (plaza, kerbs), `Wood` / `WoodPlanks`
  (fences, benches, arch, pads, booths), `Limestone` (lane walls), `Ground` (dirt, soil, end wall, cliff), `Rock` / `Basalt` / `Slate` (lane floors, boulders),
  `Concrete` (cannon pad), `Neon` (lights only), `Glass` (shop display). Round shapes (balls, cylinders) are still used for trees, bushes, lantern globes, finials.
- Lane colours (muted): 1 brick red `184,92,84`, 2 slate blue `86,122,170`, 3 forest green `92,146,104`, 4 ochre `206,160,76`.
- Shared colours: linen `226,214,190`, stone sand `190,172,144`, slate tan `122,106,88`, wood `108,74,50`, light wood `160,120,82`, soil `84,60,42`, lantern `255,200,120`.
- Plants: greens `70,118,62 / 88,136,70 / 108,152,82`, autumn gold `200,150,60 / 184,120,48 / 214,168,76`, wildflowers yellow / white / lavender / red.
- Biome floors: Dirt `132,98,68` (Ground), Stone `128,130,134` (Slate), DeepRock `74,80,98` (Rock), Crystal `112,90,150` (Slate), Obsidian `50,42,62` (Basalt), MagmaCore `170,70,40` (Basalt).
- Ground greens: `88,128,64` / `96,138,70` / `104,148,76`, plot yards `110,152,80`.
- **Lighting (v2, dim + natural):** ClockTime 16.3, Brightness 1.5, ExposureCompensation -0.25, Ambient `70,74,84`, OutdoorAmbient `104,110,122`,
  ColorShift_Top warm, ColorShift_Bottom dark grey, `CozyAtmosphere` (Density 0.3, Haze 0.8, Glare 0, blue-grey, NO pink), `CozyColor` (Contrast 0.1, Saturation -0.05),
  `CozyBloom` (Intensity 0.15, Threshold 1.8), `CozySunRays` 0.03. Existing `Sky` kept. If still too bright: lower `L.Brightness` / `ExposureCompensation`.
  If too dark in lanes: raise `Ambient` a little. `Lighting.Technology` cannot be set from scripts: set **Future** by hand in Studio.
- **Booths stay plain on purpose.** The owner earlier asked to drop the ornate booth look (awning, valance, sign posts, finials, banners, crest, stars).
  BoothBuilder keeps the plain pitched roof + floating BillboardGui label. Do not add ornaments back unless the owner asks.
- Cozy touches live in the map, not on the booths: wishing tree, benches, fairy lights, lanterns, flowers, fireflies.

---

## Layout spec (studs)

Arm numbering is clockwise: **1 = North (-Z), 2 = East (+X), 3 = South (+Z), 4 = West (-X)**.
Plot N always pairs with Lane N. Distances below are measured outward from the centre.

| Thing | Distance from centre | Size |
|---|---|---|
| Hub plaza | 0 - 70 | radius 70 (+4 kerb ring), pebble ring dia 104, inner dia 84 |
| Wishing tree | centre | kerb dia 34, trunk 26 tall, canopy up to ~49 high |
| Plot | 60 - 180 | 120 x 120 |
| Booth ring (Shop, Sell, Smelter) | ring radius 36 around plot centre | pad diameter 48 |
| Cannon pad | 180 - 206 | 80 wide, 26 deep |
| Lane floors (6 biomes x 120) | 180 - 900 | 80 wide |
| Wall start (blocks begin) | 212 | WallLength 720, ends 932 |
| Lane side walls (limestone) | 180 - 932 | 8 thick, 44 high, round rail on top |
| End wall | 932 - 956 | 96 wide |
| Rim wall (96 segments) | radius 980 | 60 high |
| Grass ground disc | radius 1020 | |

Booth positions per plot (local frame): Shop left, Sell right, Smelter on the hub side. The lane side stays open so players can walk straight
to the cannon. All booths face the plot centre (PlotService `slotCFrame`). Spawn sits 18 studs lane-side of plot centre and faces down the lane.
Plot sign is on the hub-side arch and faces the hub.

---

## How the map is generated (read this before touching Workspace.Map by hand)

The map is **generated by a ModuleScript**, not hand-placed:

`ServerStorage.MapGenerator` (dev tool, edit mode only)

Run it from the command bar or MCP `execute_luau` (Edit datamodel):

```lua
local SS = game:GetService("ServerStorage")
local fresh = SS.MapGenerator:Clone()   -- clone, because require() caches the module
fresh.Parent = SS
print(require(fresh).build())
fresh:Destroy()
```

- All sizes live in `Gen.CFG` at the top. All colours are constants at the top (`LANE_COLORS`, `BIOME_COLORS`, `BIOME_MATERIALS`, `CREAM`, `WOOD`, ...). Change them there and re-run.
- `build()` wipes and recreates Map.Lanes, Plots, Structure, Hub, Ground, Rim and **Decor** (decor is now built from scratch, it is no longer re-placed
  from old templates), deletes Map.PlotFences, then calls `Gen.applyLighting()`.
- Plot/lane parts are still cloned from the existing map as templates (Sign + SurfaceGui, Spawn, Pad, Origin, Floor_*, CannonPad, ...). Do not rename them.
  Colour and material of every cloned part are overridden at the clone site, so the templates' old look does not matter.
  Style changes can be applied by patching the constants and `mk(...)` / `.Material = ...` lines, then re-running `build()`.
- Helpers inside `build()`: `mk` (plain part), `ball`, `bulb` (fairy light), `flower`, `lantern` (post + glowing globe + PointLight), `clone`, `at`, `place`.
- **The scripts live only in the Studio place file** (`.rbxlx` is gitignored; `src/server` only has a hello-world script). They are not in git.
  Export `MapGenerator` and `BoothBuilder` to files if you want them version-controlled.
- Backups of the pre-cozy scripts and map (`BoothBuilder` currently equals `BoothBuilder_PreCozy`): `ServerStorage.MapGenerator_PreCozy`, `ServerStorage.BoothBuilder_PreCozy`, `ServerStorage.OldMap_Backup`.
  **Delete all of these (and `MapGenerator`) from ServerStorage before publishing** if you do not want them shipped.

### Gotchas
- `require` caches. After editing the module Source you must clone it (as above) or the old code runs.
- Cylinder parts need `* CFrame.Angles(0, 0, pi/2)` to lie flat (axis is X). `Size.X` of a cylinder is its length / thickness.
- Ball parts are always built with equal X/Y/Z size (`ball()` takes one diameter).
- Decor models are built around the origin with the base at y = 0 and `WorldPivot = CFrame.new()`, then `place()` moves them with `PivotTo`.
- Roblox parts cap at 2048 per axis, so the ground disc is 2040 wide and the rim radius is 980.
- Do not put coplanar parts at the same Y (z-fighting). Ground discs and plaza discs are stacked 0.02-0.06 apart on purpose.
- Lantern base is y = 0.9 (plaza / cannon pad top is about 0.95).
- The screenshot helper subagent is unreliable here (blank images before, `max_tool_calls` error on 2026-10-03). It once left the camera `Scriptable`.
  If you try it, afterwards run `workspace.CurrentCamera.CameraType = Enum.CameraType.Fixed`.
- `execute_luau` cannot read `Lighting.Technology` (missing RobloxScript capability). Skip that property.

---

## Contracts other scripts rely on

**PlotService** (`ServerScriptService.PlotService`, unchanged) expects:
- `Workspace.Map.Plots.Plot1..4`, each with `Sign` (direct child, SurfaceGui > TextLabel named `Text`), `Spawn`, and models `Slot_Shop`, `Slot_Smelter`, `Slot_Sell` each containing an `Origin` part.
- Plot attributes `CenterX`, `CenterZ` (world coords of plot centre), `PlotIndex`, `LaneIndex`, `OwnerUserId`.
- `Workspace.Map.Lanes.Lane1..4` with attribute `OwnerUserId`.
- Map attribute `Plots` (max players, 4).
- `BoothBuilder.build(kind, cf, level, parent)` returns a Model with `InteractZone`, `UpgradeAnchor`, attributes `Station` and `Level`. Verified after the restyle.

**Lane attributes** (for the future wall / cannon script):
`LaneIndex, OwnerUserId, WallHeight (32), BlockSize (4), WallLength (720), WallWidth (80), LaneStartR (180), WallStartR (212), WallStart (Vector3, world, ground level, centre of lane), LaneDirection (Vector3, unit, outward), LaneYawDeg`.
Lanes point in four directions, so use `WallStart` + `LaneDirection`. `CenterX` and `WallStartZ` no longer exist.

**Map attributes:** `Layout="Plus", Style="Cozy", Plots, Lanes, BlockSize, LaneWidth, PlotSize, PlazaRadius, RingRadius, LaneStartR`.

---

## Next steps (in order)

1. **Look at it in Studio** (nobody has seen the cozy look yet): overhead of the hub, one plot with booths, the inside of a lane, the rim at the edge.
   Check: is the lighting dark enough now (owner said v1 was too bright), colours look natural and not candy, tree canopy size vs the plaza, fireflies visible.
2. Set `Lighting.Technology` to **Future** by hand (cannot be done by script).
3. **Playtest**: join, plot claim, spawn position and facing, all 3 booths build inside the plot, upgrade prompt, leave / rejoin.
4. Cozy polish ideas (only if wanted): ambient music / nature sounds, a pond or small bridge in a wedge, cottage props, warm UI theme (muted wood / cream panels), soft sound effects.
5. Lane gameplay: generate wall blocks from the lane attributes (pastel biome colours, rounded debris), wire the cannon at `CannonMount`, keep the lid (`LaneCeiling`) solid.
6. Tune plot size via `PlotSize` / `SlotRing` if 120 still feels too big or small.

## Open questions for the owner
- Does the v2 natural look match what you meant by "Build a Haven"? Screenshots of the parts you like would let me match it more closely.
- Is the lane length (720) right now that width is 80, or should lanes be shorter / longer? (Changing it means changing the rim radius too.)
- Should the lane ceiling stay as an invisible solid lid, or be removed?

---

## Change log

| Date | Change |
|---|---|
| 2026-10-02 | Reworked the map into the "+" on a circle layout (plots inner, lanes outer), lane width 80, plot 120, rim wall at radius 980. See older notes in git history. |
| 2026-10-03 | **Cozy restyle ("Build a Haven" inspired).** Rewrote `ServerStorage.MapGenerator` (pastel palette, cherry-blossom wishing tree + benches in the hub, picket fences + fairy lights, flower beds, round decor trees/bushes/pebbles, fireflies, cream lane walls with round lane-coloured rails, peach rim cliff, `Gen.applyLighting`). Recoloured `ServerScriptService.BoothBuilder` (pastel + SmoothPlastic, layout unchanged, kept plain). Rebuilt Workspace.Map (2359 descendants). Checked PlotService contract and booth levels 1-5 numerically. Backups: `MapGenerator_PreCozy`, `BoothBuilder_PreCozy`. Screenshot check failed. |
| 2026-10-03 (v2) | Owner said v1 was too candy / pink and too bright, wants it realistic. Patched `MapGenerator`: natural earthy palette (muted lane colours, greens, autumn-gold tree, wildflowers), real materials (Cobblestone, Pebble, Slate, Wood, Limestone, Ground, Rock, Basalt), dim natural lighting (Brightness 1.5, Exposure -0.25, blue-grey haze, low bloom). Restored `BoothBuilder` from `BoothBuilder_PreCozy`. Rebuilt the map. Checked lighting values, booth levels 1-5, PlotService contract numerically; no visual check yet. |
