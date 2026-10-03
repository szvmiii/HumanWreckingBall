# Claude.md — Human Wrecking Ball (Roblox, placeId 139473074201684)

Working notes so context survives between sessions. **Update this file after every change.**
Last updated: 2026-10-03

## What the game is
Tycoon + wrecking-ball. Each player gets a plot (Shop / Smelter / Sell booths) and a lane with the same
number. A cannon on the lane's pad launches the player down a very long lane of destructible walls.
Max 4 players (4 plots, 4 lanes).

## What we are doing right now
Reshaping the map into a **true "+" seen from above**, with very long / wide / tall lanes.
Status of the current request:

- [x] Remove the circular grass meadow, rim wall, and all trees/bushes/boulders outside the plus
- [x] Hub (spawn area) bigger: circle r=70 -> **square 260 x 260** (HubHalf 130)
- [x] Plots smaller: 120 x 120 -> **120 wide x 88 deep** (about 27% less area)
- [x] Remove all unnecessary land (only hub + 4 plots + 4 lanes exist now; void elsewhere)
- [x] Lanes really long / wide / tall (see numbers below) with strong biome changes
- [ ] Playtest: walk a plot -> lane, check cannon pad, fall-off/void behaviour, lighting in the void
- [ ] Decide wall destruction approach (block count is huge, see "Open concerns")

## Map spec (all numbers live in `ServerStorage.MapGenerator` -> `Gen.CFG`)
| Thing | Before | Now |
|---|---|---|
| Hub | circle, radius 70 | square, half-size 130 (260 x 260) |
| Plot | 120 x 120 | 120 wide x 88 deep |
| Booth ring radius | 36 | 28 (pads 38 diameter) |
| Lane inner width | 80 | **104** (arm outer width 120 = plot width, keeps the plus clean) |
| Lane length | 720 (6 x 120) | **1800** (6 biomes x 300) |
| Lane start (from centre) | 180 | 218 (PlotR0 130 + PlotD 88) |
| Cannon pad depth | 26 | 40 |
| Destructible wall height | 32 | **64** (WallHeight attr), starts 48 after lane start |
| Side wall height | 44 | **110** |
| Lane bounds / invisible lid | 44 | **120** |
| Footprint | circle r~1000 | plus, +-2044 studs end to end |

Arms: 1 = North (-Z), 2 = East (+X), 3 = South (+Z), 4 = West (-X). Plot i uses Lane i.

Lane biomes in order from the cannon: Dirt, Stone, Deep Rock, Crystal Cavern, Obsidian, Magma Core.
Each has its own floor colour/material, tinted side wall section, glowing neon strip along the wall base,
and coloured rail lights. A lit gate lintel with a billboard label ("CRYSTAL CAVERN / 900 studs deep")
sits at each biome change (gates are non-collidable).

Hub: square cobble plaza, 1.4x scaled cherry-blossom wishing tree in the centre, 4 benches (lane colours),
4 lanterns, 4 corner flower beds, cobble path to each plot, low kerb wall around the edge with an opening at
every arm, a few fireflies. No other decor exists.

## How to rebuild the map
Edit mode, command bar or MCP `execute_luau`:
```lua
require(game.ServerStorage.MapGenerator).build()
```
- It clears and regenerates Lanes, Plots, Structure, Hub, Ground, Decor (and deletes Rim if present).
- It harvests template parts (Sign gui, Spawn, Pad, Floor_*, etc.) from the existing Plot1 / Lane1 first,
  so **do not delete Plot1 / Lane1 by hand** without keeping those template part names.
- Use a fresh `:Clone()` of the module before `require` if you re-run after editing it (require caches).
- Tweak sizes in `Gen.CFG`; everything else derives from it.
- Backups of the previous look: `ServerStorage.MapGenerator_PreCozy`, `BoothBuilder_PreCozy`
  (older style; the current MapGenerator is the circular-hub version replaced by this plus version).

## Scripts and what they rely on
- `ServerScriptService.PlotService` — assigns plot/lane on join, builds booths on `Slot_*` models,
  uses plot attributes `CenterX/CenterZ` (booths face that point), the `Spawn`, and the `Sign` SurfaceGui.
- `ServerScriptService.BoothBuilder` — builds Shop/Smelter/Sell booths (platform 30 x 24).
  Its invisible `InteractZone` reaches 26 studs in front; nothing uses it yet.
- Nothing references `Map.Rim`, `RingRadius`, or `PlazaRadius` anymore (removed).
- Lane attributes (for the future wall/cannon scripts): `WallHeight, BlockSize, WallLength, WallWidth,
  LaneStartR, WallStartR, WallStart, LaneDirection, LaneYawDeg, LaneHeight, LaneLength, OwnerUserId`.
- Map attributes: `Layout=Plus, Style=Cozy, Plots, Lanes, BlockSize, LaneWidth, PlotSize, PlotDepth,
  HubHalf, LaneLength, LaneHeight, LaneStartR`.

## Open concerns / next ideas
1. **Wall block count is huge.** At BlockSize 4 each lane wall is about 104 x 64 x 1752 -> roughly
   182k blocks per lane, 730k total. Do NOT spawn that many physical parts. Options: bigger BlockSize
   (8 = about 8x fewer), spawn walls in chunks ahead of the player, or fake distant walls and only
   build real blocks near the ball.
2. Gate labels are visible from up to 1000 studs and clutter the hub view; lower `MaxDistance` if annoying.
3. Void around the plus: players that fall respawn via FallenPartsDestroyHeight. Consider a soft
   skybox/void colour in Lighting so the empty space looks intentional.
4. Plot depth is 88; booths + arch + fences fit but are tight. Do not shrink PlotD further without
   moving the booth ring (SlotRing) or the arch.
5. Cannon, launch, wall destruction, Coins system (`DEV_FREE_UPGRADES = true` in PlotService) are still TODO.

## Change log
- 2026-10-03: Rebuilt as plus: removed circle meadow/rim/trees, square 260 hub, plots 120x88,
  lanes 104 wide x 1800 long x 110 tall walls, per-biome walls/strips/gates. MapGenerator rewritten.
  Verified: 4 plots / 4 lanes / 3 slots each, spawns at +-194 from centre, floors contiguous 218 -> 2018.
