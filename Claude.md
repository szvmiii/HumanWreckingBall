# Claude.md — Human Wrecking Ball (Roblox)

> **Rule for Claude:** After every change (script added/edited, design decision, new task), update the
> **Current Status**, **TODO** and **Change Log** sections below so the next session knows exactly where we are.

- Studio place: `[💥] Human Wrecking Ball` — placeId `139473074201684`
- Genre: launch / dig / progression (incremental-style loop with physical destruction)

---

## 1. Game Concept

Every player owns a **cannon**. They fire themselves out of it, smash through a wall of layered
blocks, and collect **ores** based on how deep they dig. Ores are smelted into **ingots**, which are
sold or used to **craft/upgrade** cannon parts, letting players dig deeper on the next launch.

### Core loop
1. Load into the cannon and set launch power (timing bar / charge).
2. Launch and crash through the wall. Each block costs momentum; run out of momentum and you stop.
3. Ores drop based on depth reached and blocks destroyed. Auto-collected to inventory.
4. Return to the hub: smelt ores into ingots, sell, or craft upgrades.
5. Upgrade the cannon, then launch again to go deeper.

### Depth = reward
- Walls get **harder** the deeper you go (more HP per block, needs more power).
- Deeper layers = **rarer ores** and higher value.
- Suggested layers (tunable):

| Depth layer | Hardness | Ores |
|---|---|---|
| Dirt / Clay | 1 | Coal, Copper |
| Stone | 3 | Iron, Tin |
| Deep Rock | 8 | Silver, Gold |
| Crystal Caves | 20 | Amethyst, Sapphire |
| Obsidian | 50 | Ruby, Diamond |
| Magma Core | 120 | Mythril, Star Fragment |

---

## 2. Multiplayer Design

- Cannons are **lined up next to each other** in a shared hub/firing line so players see and hear each other.
- Each player has a **private lane/tunnel** (own wall section) so nobody can steal their loot.
  - Lane is assigned on join, released on leave.
  - Server owns the lane: drops are created **only for the lane owner** (server-authoritative).
  - Collision groups so other players cannot enter or interfere with a lane.
  - Wall regenerates (or resets) between runs.
- Social hooks: see neighbors launching, leaderboard for **deepest depth**, shared "boss wall" events later.

---

## 3. Progression & Upgrades

**Currency/materials:** Ores → Ingots (smelter) → Coins (sell) or Parts (craft).

| Upgrade | Effect |
|---|---|
| Cannon Power | Higher launch speed / more momentum |
| Drill Head | Worn on the player's head, pierces blocks, less momentum lost per block |
| TNT Charge | Bigger break radius per impact, more drops per hit |
| Armor / Helmet | Fewer momentum penalties on hard layers |
| Magnet | Ore auto-collect range |
| Luck | Higher rare-ore chance |
| Auto-Smelter | Ingots processed offline / faster |

Two upgrade paths to keep choices interesting: **sell for coins** (flexible) vs. **craft parts**
(specific, needs particular ingots).

---

## 4. Visuals & Game Feel ("Juice")

Satisfying visuals are a top priority. Keep players hooked with:

- **Hit-stop** (tiny freeze) on heavy block breaks.
- **Camera shake + FOV kick** scaled to impact strength.
- **Debris particles** flying out from every broken block; block chunks that fade quickly.
- **Shockwave ring / dust puff** on big breaks and TNT blasts.
- **Slow-motion moment** when hitting a rare ore or breaking a layer boundary.
- **Ore reveal:** ores glow and pop out with a sparkle; rarity color beams for rare drops.
- **Ore magnet:** drops fly to the player with a pitch-rising collect sound.
- **Combo / chain counter** and floating damage or drop numbers.
- **Pitch-ramping sound** on consecutive breaks; distinct heavy "thud" for hard layers.
- **Trail + speed lines** while launching; fire/smoke trail on cannon upgrades.
- **Depth meter** and layer-change banner ("Entered Deep Rock!").
- **Cannon upgrades visibly change** the cannon (bigger barrel, glow, particles).
- **Smelting animation** (furnace glow, ingot pop) so the hub is satisfying too.
- **End-of-run summary** with tallying numbers, sound ticks, and a "new record" flourish.

---

## 4b. Art Direction (decided)

- **Style: low poly, NOT blocky, colorful with clearly separated colors** (Islands-inspired shapes, but flat colors). The lamp image was only a rough
  reference — **no lamps**. Do NOT add lamps, lanterns, flowers, bushes, pebbles, outlines, studs or realistic textures.
- **No textures:** every part is `Material.SmoothPlastic` (flat color), Smooth surfaces, Reflectance 0. Only `Neon` is allowed, for glows (Smelter furnace opening).
- **"Not blocky" tricks used:** outer walls lean outward 10 degrees (canyon-like) with a flat grass cap; divider tops have a diamond ridge (45-degree rotated prism);
  booths have diamond finials on the sign and a zigzag diamond valance under the awning; trees are faceted pines (stacked rotated cubes); hub end is a
  16-segment half circle of flat triangles. Prefer angled / faceted / rotated-cube shapes and wedges over plain boxes when adding new objects.
- **Separated palette (each area has its own hue):**
  walls terracotta `206,118,70` + bright grass cap `108,214,84` + darker base band; tunnel end wall violet `88,58,150`; dividers teal `46,176,190` with light ridge;
  hub floor sunny sand `255,226,154` / `250,200,118`; lane accents pink / yellow / mint / blue (pads, spawns, lane labels);
  zones Dirt orange, Stone blue-gray, DeepRock indigo, Crystal magenta-violet, Obsidian deep purple, MagmaCore red-orange;
  booths Sell green / Smelter orange / Shop blue with dark wood platform, cream + theme striped awnings, dark violet sign boards; pines two greens.
- **Lighting:** warm golden hour (ClockTime 16), warm sun, cool ambient, light haze Atmosphere (`IslandsAtmosphere`), ColorCorrection (`IslandsColor`),
  Bloom (`IslandsBloom`), SunRays (`IslandsSunRays`). Can be simplified (less haze) if colors look washed out. Future lighting is NOT needed anymore
  since textures/lamps are gone (`Lighting.Technology` cannot be set from scripts anyway).
- Sized for the power of the launch: v3 scaled x2 (`S = 2`): lanes 32 wide, wall blocks 4 studs, cannon pads 32 x 24.
- World label font: `FredokaOne`.

---

## 5. Technical Notes / Plan

- **Server authority:** server validates launches, block damage, and drops. Client only handles visuals.
- **Performance:** wall as pooled chunks/parts; cosmetic debris created **client-side** and cleaned
  with `Debris`/pooling; avoid unanchored physics for thousands of parts.
- **Data:** player profile (coins, ores, ingots, upgrade levels, best depth) saved with DataStore
  (consider ProfileService/ProfileStore).
- **Suggested structure:**
  - `ReplicatedStorage.Shared` — config modules (layers, ores, upgrades)
  - `ServerScriptService` — LaneService, WallService, DropService, DataService, UpgradeService
  - `StarterPlayerScripts` — CannonController, VFXController, UIController
  - `ServerStorage` — wall block templates, ore models

---

## 6. Map Layout (built, v7)

Rectangular map. **Hub is behind the cannons (+Z) and ends in a low-poly half circle; tunnels run toward -Z.**
Everything lives under `Workspace.Map` (~300 instances). There is **no baseplate / outside world**.

```
        .-"""-.                 half-circle hub end (16-gon, radius 73), center Z=70
      /  Shop   \  booths on the arc at radius 46: Sell 30 deg, Smelter 90 deg, Shop 150 deg, all facing the hub center
     | Smelter   |
 Z=70 |___________|  hub rectangle Z 0..70, 4 spawns at Z=40 (one per lane, lane colors), open plaza in the middle
 Z=0  |  cannon line: CannonPad Z 0..-24
      |[L1][L2][L3][L4]  lanes 32 wide, 6-stud dividers, interior width 146 (X -73..73)
 Z=-32|  wall starts (WallStartZ)   6 zones x 120 studs
 Z=-752  tunnel end (plum wall)
 -Z
```

- **4 lanes**, each 32 wide, 6-stud dividers (44 tall). Lane centers X = -57, -19, 19, 57.
- **Outer walls** (`Structure.OuterWalls` Model): flat-colored dirt body leaning outward 10 degrees + flat grass cap + base band + grass lip, 24 thick, 60 tall, around the hub arc
  (16 segments), both tunnel sides and the tunnel end. `Structure.Dividers` is a Model (teal dividers + `Ridge` diamond prisms on top).
- **Booths** (`Hub.Stations`: `SellBooth`, `SmelterBooth`, `ShopBooth`): each ~30 x 24 studs, wood studded platform, colored back/side walls,
  striped awning, counter, sign board (SurfaceGui text), plus props. Sell = gold ingot pile + crates, Smelter = stone furnace with Neon glow +
  PointLight, chimney, anvil, Shop = shelves with crates + small cannon model. Each has an invisible `InteractZone` (24x8x14, CanTouch) in front and
  the model attribute `Station` = "Sell" / "Smelter" / "Shop" for ProximityPrompt / touch scripts later.
- **Depth zones** (120 studs each; `Zone`/`DepthStart`/`DepthEnd` attributes on floor parts):
  Dirt 0-120, Stone 120-240, DeepRock 240-360, Crystal 360-480, Obsidian 480-600, MagmaCore 600-720.
- Each `Lane<i>` model: `LaneBounds` (invisible volume, PrimaryPart), `CannonPad` (lane color), `CannonMount`
  (invisible anchor for the cannon + "LANE i" label), `Floor_<Zone>` x6, `LaneCeiling` (invisible, keeps players in).
- **Lane attributes** (for future scripts): `LaneIndex`, `WallStartZ=-32`, `WallLength=720`, `WallWidth=32`,
  `WallHeight=32`, `BlockSize=4`, `CenterX`, `OwnerUserId=0`. Wall grid per slice = 8 x 8 blocks of 4 studs.
- Wall blocks are **not generated yet**: lanes are empty tunnels. Full wall = 180 slices x 64 blocks = ~11.5k blocks per lane,
  so **WallService must generate/stream in chunks** and pool parts.
- Hub floor: `HubFloor` (studded rectangle) + `ArcFloor` (16 flat triangles made of thick wedges, alternating cream/peach). Raycast-verified: no gaps.
  Booth footprints verified to sit fully inside the hub.
- `Workspace.Map.Decor`: 6 faceted low-poly pines. Lamps, lanterns, outlines and all textures were removed. Only 1 PointLight remains (Smelter furnace glow).
- Map attributes: `Lanes=4`, `LaneWidth=32`, `BlockSize=4`, `Scale=2`.
- The map is generated by a Luau script run in Studio. To rebuild: be in **Edit mode** (stop any playtest), delete `Workspace.Map`, rerun the generator.
- **Studio connection:** the MCP `studio_id` changes when Studio reconnects. Always call `list_roblox_studios` when a call says the id is not connected.
- Camera lesson: the screen_capture subagent once left `workspace.CurrentCamera` on `Scriptable`, freezing the Studio camera. Avoid it.
  Claude has not seen the map visually, only verified via raycasts/object tree.

## 7. Other Studio State

- ReplicatedStorage: `Shared` (folder, 1 ModuleScript, not yet inspected)
- StarterPlayer: `StarterPlayerScripts` (1 LocalScript, not yet inspected)
- `ServerScriptService`, `ServerStorage`, `StarterGui`, `StarterPack`: not yet inspected

## 8. Current Status

**Phase:** Map v7 done (flat-color low poly, not blocky, separated colorful palette, no lamps/textures/outlines/studs). Lanes are empty. Next: cannon + launch mechanic and wall generation.


## 9. TODO (next steps)

- [x] Rebuild map as a rectangle: hub near, tunnels in one direction
- [x] Widen lanes (20 -> 32) and switch to low-poly stud style with cute lighting
- [x] Map v3: 4 lanes, smaller/player-sized, half-circle hub, enclosed by dirt+grass walls with details
- [ ] User: check the v7 look (colors, leaning walls, ridges, booths) and request tweaks
- [ ] Inspect existing scripts (`Shared` module, LocalScript) and check they don't reference the old Hub/Plaza
- [ ] Lane assignment system (owner per lane, collision groups so others can't enter)
- [ ] Wall generation per lane with depth layers and per-block hardness
- [ ] Cannon model on `CannonMount` + launch mechanic (timing bar) and momentum model
- [ ] Block damage / momentum loss on impact
- [ ] Ore drops + inventory + data saving
- [ ] Smelter + sell shop + craft/upgrade UI at the hub stations
- [ ] First juice pass (camera shake, particles, sounds, hit-stop)
- [ ] Upgrades: Drill Head, TNT, Power, Magnet
- [ ] Map polish: lighting/atmosphere, hub decoration, lane ceilings/visual walls, layer-change signs

## 10. Change Log

| Date | Change |
|---|---|
| 2026-09-28 | Created Claude.md with game concept, design, and plan. Scanned Studio tree (shallow). |
| 2026-09-28 | Deleted old square Hub + Baseplate. Built `Workspace.Map`: 8 lanes, 6 depth zones, hub with stations and 8 spawns, dividers, bedrock end wall. |
| 2026-09-28 | Restyled to low-poly/stud/pastel look, widened lanes 20 -> 32 (map now 292 wide, tunnels end at Z=-640, hub 160 deep), added tree/bush decor and cute lighting. |
| 2026-09-28 | Map v3: shrunk to player size, 4 lanes x 16 wide, low-poly half-circle hub with 3 stations on the arc, big dirt walls with grass caps + details enclosing everything (no outside world), colorful lane accents. Removed old ground/baseplate. |
| 2026-09-28 | Fixed frozen Studio camera (CameraType was left Scriptable by the screenshot helper; reset to Fixed and moved behind the hub looking down the lanes). |
| 2026-09-28 | Map v4: scaled everything x2 (lanes 32 wide, tunnels 720 long ending at Z=-752, hub 60 deep + radius-73 half circle with 16 segments, walls 24 thick x 60 tall, blocks 4 studs) so the launch feels powerful and has room for drill/blast upgrades. Stopped a running playtest to edit. |
| 2026-09-28 | Requested art pass (Islands-style outlines, booths, no flowers, more hub space) written into section 8b; not yet built because Studio tools were unavailable. |
| 2026-09-28 | Map v5 applied: removed flowers/bushes/pebbles/tufts, clean walls, hub 60 -> 70 deep with spawns at Z=40, 3 real booths (Sell/Smelter/Shop) with InteractZones, Islands-style Highlight outlines (15), 6 natural trees. Studio had reconnected with a new studio_id. |
| 2026-09-28 | Map v6: removed outlines and studs; textured materials (Ground/Grass/Slate/Cobblestone/Wood/Fabric/Metal), richer colors, crooked lamp posts + booth lanterns with warm PointLights (from the Islands lamp reference), golden-hour lighting with atmosphere/color grading/bloom/sun rays. Lighting.Technology could not be set by script (user must set Future manually). |
| 2026-09-28 | Map v7: deleted lamps/lanterns and all textures (everything SmoothPlastic). Colors made flat and clearly separated per area. Less blocky: outer walls lean outward 10 deg, diamond ridge on dividers, diamond finials + zigzag valance on booths, faceted pines. |
