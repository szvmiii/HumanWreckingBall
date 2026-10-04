# CLAUDE.md - Human Wrecking Ball (placeId 139473074201684)

Working notes so any session can pick up where we left off.
**Update this file after every change.** Last updated: 2026-10-04.

NOTE: older versions of this file described a circular island with a circular hub. That is outdated.
The real map is a square hub with a "+" of four arms (see below).

---

## What we are doing right now

Plot layout rework (DONE, needs a visual check by eye):

- Each plot now has **two booths on the left and two on the right** of a clear cobbled path (the middle stays empty,
  spawn to cannon is a straight walk).
- A **4th booth, Forge** (crafting / upgrades), was added so both sides are symmetrical.
- **4 leaderboards** stand in the four hub corners (placeholders, no data yet).

### Status

| Item | State |
|---|---|
| "+" map: square hub, 4 arms, plot (inner) + lane (outer) | DONE |
| Plot booths: Shop + Smelter left, Sell + Forge right, middle path empty | DONE (checked numerically + playtest) |
| Forge booth (BoothBuilder, PlotService costs / levels) | DONE |
| Leaderboard boards in hub corners | DONE (static placeholders) |
| Plot entrance gates: wooden gatehouse with open plank doors (no metal) | DONE (checked numerically, not seen by eye) |
| Hub benches redesigned (slatted, armrests, cushion + pillow) | DONE (not seen by eye) |
| Autumn leaves at plot spawns | REMOVED on request (owner did not want them) |
| Playtest: plot claim, 4 booths built, upgrade prompts on all 4 | DONE (Plot1 checked) |
| Visual check by eye (booths facing the path, Forge look, boards readable, plot flower beds removed) | NOT DONE |
| Leaderboard data (OrderedDataStore) | NOT STARTED |
| Lane wall blocks / cannon / wrecking-ball gameplay | NOT STARTED |

---

## Layout spec (studs)

Arm numbering is clockwise: **1 = North (-Z), 2 = East (+X), 3 = South (+Z), 4 = West (-X)**. Plot N pairs with Lane N.
Local arm frame in the generator: `u` = across the arm (negative = left when looking outward), `r` = distance outward from the hub centre.
All sizes live in `Gen.CFG` in `ServerStorage.MapGenerator`.

| Thing | Value |
|---|---|
| Hub | square slab, HubHalf 130 (260 x 260), plaza half size 116, wishing tree in the centre |
| Plot | r 130 - 218 (PlotD 88), 120 wide, yard + cobbled `PlotPath` 36 wide down the middle |
| Booth platform | 30 x 24 studs (BoothBuilder), height about 16 with roof |
| Booth centres | u = +-44 (`SlotX`), r = plot centre +- 18 (`SlotZ`) |
| Booth footprint | u 32..56 on each side, so the walkable middle is 64 wide (path 36 cobbled) |
| Booth facing | straight across the path (left side faces +u, right side faces -u) |
| Spawn | on the path, 20 studs lane-side of plot centre, faces down the lane |
| Plot gate | wooden gatehouse on the hub edge of the plot (r 131), see below; sign faces the hub |
| Lane | 104 wide, 6 biomes x 300 = 1800 long, starts r 218 (LaneStartR) |
| Cannon pad | 104 x 40, wall starts 48 studs after the lane start |
| Lane side walls | 8 thick, 110 high, biome tinted, glowing strips, biome gates |

Booth assignment per plot (looking outward from the hub):

| | Hub side (r - 18) | Lane side (r + 18) |
|---|---|---|
| Left (u -44) | Shop | Smelter |
| Right (u +44) | Sell | Forge |

Smelter and Forge are on the lane side so players returning from a run reach them first. To swap booths, edit the `slots` table in `Gen.build()`.

Each plot: `Slot_Shop / Slot_Smelter / Slot_Sell / Slot_Forge`, each containing an `Origin` part (position AND facing, set by the generator)
and a flat wooden `Pad`. The old flower beds in the plots were removed (they sat where the booths now stand).

### Plot gate (`buildGate` in MapGenerator)

All wood and stone, no metal. Parts live in `Plot<i>.Gate` (Model); `Plot<i>.Sign` stays a direct child of the plot (PlotService needs it).
- Two timber posts (u +-31, 25 tall) on slate footings, with base band and cap, diagonal braces to the crossbeam.
- Crossbeam, gabled roof in the lane colour, ridge pole, ball finials on the ridge ends.
- Sign hangs from the crossbeam on two ropes (fabric), lane-coloured trim strip under it.
- Two open plank doors (5 planks, 2 rails, Z-brace, wooden hinge straps and latch), hinged at u +-29.4 and swung 80 degrees into the plot.
  They stay inside u 23..31, so the 36-wide path and the booths (u >= 32) are not blocked.
- Short two-rail fences run from each gate post to the plot corner (closes the front of the plot), fairy light on each post.
- A lantern hangs from a wooden arm on the hub side of each post (warm PointLight).

### Hub benches

- 4 benches, one per hub corner, facing the tree, built by `bench()` in MapGenerator: sled feet, front legs, tall rear posts, armrests,
  4-slat seat, 2-slat back + top rail, lane-coloured cushion and pillow.
- Autumn leaves (on the ground at plot spawns and on the benches) were tried and removed. Do not re-add them unless asked.

### Leaderboards

- Four boards in the hub corners at (+-98, +-98), facing the plaza centre. Board colour = lane colour of the corner.
- Corner order: (+x,+z) Depth, (+x,-z) Coins, (-x,+z) Ores, (-x,-z) Launches.
- Screen part: `Workspace.Map.Hub.Leaderboard_<Stat>` with attribute `Stat` (Depth / Coins / Ores / Launches).
  SurfaceGui children: `Title` and `Row1..Row10` TextLabels (placeholder text `n.  ---`).
- A future `LeaderboardService` should find parts by name prefix `Leaderboard_`, read `Stat`, and fill the rows from an OrderedDataStore (refresh about every 60 s).
- The hub corner flower beds were replaced by these boards.

---

## Booths (Shop / Smelter / Sell / Forge)

`ServerScriptService.BoothBuilder` builds one booth. Kinds and theme colours: Sell green, Smelter orange, Shop blue, **Forge violet**.
Upgrades (Lv 1-5) only deepen colour and add glowing trim / lanterns, the footprint never changes.
Forge props: anvil with a glowing ingot, grindstone, hammer rack; Lv 3 quench barrel; Lv 4 glowing rune strip on the back wall.

`ServerScriptService.PlotService` claims a plot per player, builds the 4 booths on the plot slots (using `slot.Origin.CFrame` for position and facing),
and adds an upgrade ProximityPrompt per booth. `DEV_FREE_UPGRADES = true` until the Coins system exists.
Upgrade costs live in `UPGRADE_COSTS` (Forge: 200 / 800 / 2000 / 4500).

---

## How the map is generated (read before touching Workspace.Map by hand)

`ServerStorage.MapGenerator` (dev tool, edit mode only). Run from the command bar or MCP `execute_luau` (Edit datamodel):

```lua
local SS = game:GetService("ServerStorage")
local fresh = SS.MapGenerator:Clone()   -- clone, because require() caches the module
fresh.Parent = SS
print(require(fresh).build())
fresh:Destroy()
```

- `build()` wipes and recreates Map.Lanes, Plots, Structure, Hub, Ground, Rim, Decor, and re-applies lighting (cozy golden hour).
- It harvests existing parts as templates (Sign + SurfaceGui, Spawn, Origin, Ground ...). Do not rename those parts.
- Backups: `ServerStorage.MapGenerator_PreCozy`, `BoothBuilder_PreCozy`, `OldMap_Backup`. Delete them before publishing if you do not want them shipped.
- The scripts `MapGenerator`, `BoothBuilder`, `PlotService` live in the Studio place only (they are not in `src/`, Rojo does not sync them).

### Gotchas
- `require` caches. After editing the module Source you must clone it (as above) or the old code runs.
- Cylinder parts need `* CFrame.Angles(0, 0, pi/2)` to lie flat (axis is X).
- Parts cap at 2048 per axis.
- Do not put coplanar parts at the same Y (z-fighting).
- The MCP `studio_id` changes when Studio reconnects: call `list_roblox_studios` again.

---

## Contracts other scripts rely on

**PlotService** expects `Workspace.Map.Plots.Plot1..4` with: `Sign` (SurfaceGui > TextLabel `Text`), `Spawn`,
models `Slot_Shop`, `Slot_Smelter`, `Slot_Sell`, `Slot_Forge` each with an `Origin` part (facing = booth front).
Plot attributes: `PlotIndex`, `LaneIndex`, `OwnerUserId`, `CenterX`, `CenterZ`, `PlotSize`, `PlotDepth`, `HalfWidth`.
`Workspace.Map.Lanes.Lane1..4` with attribute `OwnerUserId`. Map attribute `Plots` (4).

**Lane attributes** (for the wall / cannon script): `LaneIndex, OwnerUserId, WallHeight (64), BlockSize (4), WallLength, WallWidth (104),
LaneStartR, WallStartR, WallStart (Vector3, world, ground level, lane centre), LaneDirection (unit, outward), LaneYawDeg, LaneHeight, LaneLength`.
Lanes point in four directions, so always use `WallStart` + `LaneDirection`.

---

## Next steps (in order)

1. **Look at it in Studio**: a plot from above (booths face the path? 36-wide path ok?), the gate (proportions, roof, doors), the Forge booth, the hub corner boards.
2. Leaderboard data: a DataService (coins, ores, best depth, launches) then a LeaderboardService that fills `Row1..Row10`.
3. Lane gameplay: generate wall blocks from the lane attributes, wire the cannon at `CannonMount`, keep the lid (`LaneCeiling`) solid.
4. Forge gameplay: crafting recipes (ingots to cannon parts), hook into the Coins / inventory system once it exists.
5. Polish: plot yard decoration (the removed flower beds could return along the plot fence), hub props.

## Open questions for the owner
- Is the path width (36 cobbled, 64 between booths) right, or should the booths move closer / further from the centre (`SlotX`)?
- Which exact stats for the 4 leaderboards? (currently Depth, Coins, Ores, Launches)

---

## Change log

| Date | Change |
|---|---|
| 2026-10-04 | Hub benches redesigned (slatted seat and back, armrests, posts, cushion + pillow). Autumn leaves were added at the plot spawns and benches, then deleted on request (code removed from MapGenerator). |
| 2026-10-04 | Plot gate rebuilt as a detailed wooden gatehouse (posts on stone footings, braced crossbeam, shingled roof, hanging sign, open plank doors, wing fences, lanterns). Old arch posts / metal-looking finials removed. |
| 2026-10-04 | Plot layout: booths moved to the left / right of a cobbled middle path, 2 per side. Added the Forge booth (BoothBuilder + PlotService costs / levels). Booths now face the path via the slot `Origin` facing. Plot flower beds removed. 4 leaderboard boards added in the hub corners (placeholders). Rebuilt map, verified no overlaps, playtest ok (4 booths + prompts). Rewrote this file to match the real map (square hub, not circular). |
