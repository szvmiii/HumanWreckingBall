# Claude.md — Human Wrecking Ball (placeId 139473074201684)

Working notes so any session can pick up where we left off.
**Update this file after every change.** Last updated: 2026-10-09.

---

## What we are doing right now

**Current focus: polish the launch + destructible wall (built 2026-10-09, playtested headless, NOT yet seen by eye).**

State of the game: a lane owner walks into the launch area of their cannon -> LAUNCH button -> timing bar -> the
cannon shoots them up and forward (22 deg), gravity brings them down, they bore a cone through a destructible wall
and slide to a stop -> result -> back to their plot. VFX are streamed live during the flight.

Previous session (a different model, ran out of usage) left `CannonService` with a missing `end` (it did not
compile, so no remotes existed and nothing worked), a wall only 32 studs thick at radius 568, and blast VFX that
were sent after the flight. All of that was rewritten on 2026-10-09 (see below).

### Status

| Item | State |
|---|---|
| "+" map (hub, 4 plots, 4 lanes, rim) | DONE (see Layout) |
| Booths (Village Stalls, 5 levels, 4 kinds) | DONE (see Claude_old.md for the booth design notes) |
| LAUNCH button in the launch area (no E prompt) | DONE, playtested |
| Timing bar -> server-scored power | DONE, playtested (PERFECT) |
| Angled cannon, gravity, drag, slide | DONE, playtested |
| Destructible wall from parts, cone-shaped hole | DONE, playtested (hole 23 blocks wide at entry, 3 at 340 studs in) |
| Live VFX (trail, debris, sparks, shockwave, flash, shake, FOV) | DONE, NOT seen by eye |
| **Look at it in Studio / mobile** | **NOT DONE: do this first** |
| Upgrades (BlastLevel, Cannon Power) wired to coins/shop | NOT STARTED (`BlastLevel` player attribute exists, defaults 0) |
| Ore drops / coins from destroyed blocks | NOT STARTED |
| Wall rebuild between runs | Wall persists until the lane is released; no per-launch reset yet (decide: reset on each launch, or progressive digging?) |

---

## Launch + wall system (all scripts live in Studio only, not in the Rojo `src/` folders)

| Script | What it does |
|---|---|
| `ReplicatedStorage.Shared.LaunchConfig` | All tunables + pure math (bar position, power, blast radius, hardness by depth). |
| `ReplicatedStorage.Shared.Vfx` | Client effects: `blast`, `attachTrail`, `flash`, `shake`, `update` (returns the camera shake offset). |
| `ServerScriptService.CannonBuilder` | Cannon model (angled barrel), `LaunchZone`, `Muzzle`, invisible `LaunchBarrier`, `fire()`. |
| `ServerScriptService.WallBuilder` | The wall of a lane (see below). |
| `ServerScriptService.CannonService` | Creates remotes + collision groups, builds cannon + wall when a lane gets an owner, runs the whole launch on the server. |
| `StarterPlayerScripts.LaunchController` | LAUNCH button, timing bar UI, chase camera (smoothed, FOV grows with speed), plays the VFX events. |

### Flow
1. Owner stands in `Cannon<i>.LaunchZone` -> client shows the LAUNCH button -> `RequestEnter`. Server checks lane owner + zone, puts the character into collision group `Flying` + `PlatformStand`, sends `EnterCannon(seatCFrame, barStart)`.
2. Client holds the character in the barrel; pointer = triangle wave of `GetServerTimeNow()`. Press (Space / FIRE) -> `PressLaunch(serverTime)`. Server clamps the time (max 0.35 s in the past), scores it (PERFECT / GREAT / GOOD / WEAK), sends `LaunchStarted(rating, power)`, fires the cannon and sets a REAL velocity on the HumanoidRootPart (network owner set to nil for the flight, so the engine does gravity and the floor).
3. `trackFlight` (server, every Heartbeat): applies drag, ground friction, then walks the real path in half-block sub-steps: `wall:baseHardness` -> blast radius -> drain speed -> `wall:applyBlast`. Blast events go to the client every frame via `LaunchVfx` (list of `{pos, radius, hardness, stuck}`). Stuck = blast radius < `MIN_BLAST` or speed < `MIN_SPEED`: the player stops against the wall and falls.
4. Flight ends when the player is on the ground and nearly stopped (or `MAX_FLIGHT_TIME`). Server stores `LastDistance` / `BestDistance`, sends `LaunchFinished(distance, isBest)`, 2.5 s later sends the player to the plot `Spawn` (`ExitCannon`). A safety timer also sends them home.

### Wall (WallBuilder)
- Fills the whole lane: starts at `WallStartR` (266), `WallLength` 1752, `WallWidth` 104, height `LaunchConfig.WALL_HEIGHT` (112), blocks `WALL_BLOCK` = 8 studs (13 x 14 x 219 cells = ~40k cells per lane).
- Cells are DATA (`broken` set). What you see is one slab per chunk (`WALL_CHUNK_LAYERS` = 2 layers = 16 studs) in the colour of the lane biome (Floor_Dirt ... Floor_MagmaCore). When a blast touches a chunk, the slab is swapped for its individual blocks (minus the blasted ones). A launch only fractures the chunks along its own tunnel (about 8k parts after a 383-stud launch).
- Hardness comes from `LaunchConfig.hardnessAt(depth)` (biome table `BIOME_HARDNESS` 1, 2, 3.5, 5.5, 8, 12, 18 per 300 studs of lane).
- Cone: `blastRadius = BLAST_BASE * (1 + BlastLevel * 0.15) * (speed/400)^0.5 / hardness^0.6`, capped by `BLAST_MAX` (56). Speed drains `exp(-DRAIN_K * hardness * distance)` for every stud travelled inside the wall (also through the already blasted tunnel). Harder + slower = smaller radius = cone, until the player gets stuck.
- API: `build(lane, parent)`, `baseHardness`, `hardnessAt`, `peekBlast`, `applyBlast`, `reset`.

### Collision groups
`Flying` collides ONLY with `LaneFloor` (lane `Floor_*` parts + `CannonPad`, set in CannonService at start). Walls, barrier, ceiling, other players are ignored while flying. After the run the character goes back to `Default`.

### Tunables (`LaunchConfig`) and what they do right now
`PITCH_DEG 22`, `MAX_SPEED 400` (PERFECT = x1.15), `GRAVITY 196.2` (info only: the engine uses Workspace.Gravity), `DRAG 0.35`, `DRAIN_K 0.0012`, `MIN_BLAST 3.2`, `MIN_SPEED 45`, `SLIDE_DECEL 120`, `LAND_HEIGHT 6`, `BLAST_BASE 22`, `BLAST_PER_LEVEL 0.15`. A PERFECT with no upgrades flew 383 studs (apex y about 71, lands at about 1.5 s, slides a bit). `simulateFlight`, `pathAt`, `velocityAt`, `flight` in LaunchConfig are leftovers from the old client-side flight and are unused (safe to delete).

### Playtest results (Studio, 2026-10-09)
Console clean. Player claimed Lane1, cannon + wall built (110 slabs), client entered the cannon, PERFECT press, 29 live blast events, max speed 452, flew 383 studs, ended with the player back at the plot spawn. NOT checked by eye: cannon look, bar layout, camera feel, trail/debris look, performance with 8k blocks, mobile, two players at once.

### Known issues / ideas
- Drain makes the first flight end around 380 studs; tune `DRAIN_K` / `BLAST_BASE` to taste, then make upgrades (Cannon Power -> `MAX_SPEED`, Blast Radius -> `BlastLevel`, maybe a Toughness/Drain upgrade) matter.
- Wall persists between launches (each launch continues in the old tunnel unless it is wider). Decide the design (reset per launch vs progressive digging; a reset is `walls[i]:reset()` in CannonService).
- Slab to blocks swap shows a grid (blocks are 0.2 smaller than the cell) only where a blast hit; looks fine in theory, check by eye.
- Ground slide uses the engine's friction + our `SLIDE_DECEL`; the pose is flat superman. A tumble/ragdoll would look more fun.
- Debris is real physics parts (Debris service, 1.4-2.2 s); rate limited to one burst per 0.07 s.
- Ideas: coins/ore from destroyed blocks, distance milestones, sound (none yet), leaderboards, level-up VFX for booths.
- Edit-mode tip: `Source` of scripts can be set from `execute_luau` (long string brackets) and checked with `loadstring`.

---

## Hub tree lights (2026-10-05)

`Hub.TreeLights` (Model, 64 parts, all `CanCollide=false`, `CastShadow=false`): small ring radius 9 at y=15, 8 neon bulbs, 40 black cable pieces, 4 trunk cables with clamps. Tweak the block in `ServerStorage.MapGenerator` (`Gen.build`) or edit the live parts. Re-running `build()` wipes `Hub` (and `Tree 2` and the leaderboards), so do not rebuild without re-adding them. Not yet seen in the viewport.

---

## Layout spec (studs)

Arm numbering is clockwise: **1 = North (-Z), 2 = East (+X), 3 = South (+Z), 4 = West (-X)**. Plot N always pairs with Lane N.

| Thing | Distance from centre | Size |
|---|---|---|
| Hub plaza | 0 - 70 | radius 70 (+4 kerb ring) |
| Plot | 60 - 180 | 120 x 120 |
| Booth ring (Shop, Sell, Smelter, Forge) | ring radius 36 around plot centre | pad diameter 48 |
| Cannon pad | R 218 - 258 | 104 wide, 40 deep |
| Wall (blocks) | R 266 - 2018 | 104 wide, 112 high (WallLength 1752) |
| Lane floors (6 biomes x 300) | R 218 - 2018 | 104 wide |

Lane attributes (verified): `LaneIndex, OwnerUserId, WallHeight (64, old, unused by WallBuilder), BlockSize (4, old, unused), WallLength 1752, WallWidth 104, LaneStartR 218, WallStartR 266, WallStart (Vector3, ground level, centre), LaneDirection (unit, outward), LaneYawDeg, LaneHeight 120 (invisible ceiling at y 120.5), LaneLength 1800`.
Biome order along each lane: Dirt, Stone, DeepRock, Crystal, Obsidian, MagmaCore.

NOTE: the older notes mentioned lane radius 980 / rim wall; the lane actually runs out to R 2018. Check the rim and ground disc if you ever look at the outer edge.

---

## How the map is generated

`ServerStorage.MapGenerator` (dev tool, edit mode only). Run from the command bar or MCP `execute_luau` (Edit datamodel):

```lua
local SS = game:GetService("ServerStorage")
local fresh = SS.MapGenerator:Clone()   -- clone, because require() caches the module
fresh.Parent = SS
print(require(fresh).build())
fresh:Destroy()
```
`build()` wipes and recreates Map.Lanes, Plots, Structure, Hub, Ground, Rim. Sizes live in `Gen.CFG`. Backups: `ServerStorage.OldMap_Backup`, `MapGenerator_PreCozy`, `BoothBuilder_V1_Backup`, `BoothBuilder_PreCozy` (delete before publishing).
Gotchas: `require` caches; cylinders need `* CFrame.Angles(0, 0, pi/2)` to lie flat; parts cap at 2048 per axis; no coplanar parts at the same Y.

---

## Contracts other scripts rely on

**PlotService** expects `Workspace.Map.Plots.Plot1..4` with `Sign`, `Spawn`, `Slot_Shop/Smelter/Sell/Forge` (each with `Origin`), plot attributes `CenterX, CenterZ, PlotIndex, LaneIndex, OwnerUserId`, `Workspace.Map.Lanes.Lane1..4` with `OwnerUserId`, Map attribute `Plots`. PlotService sets player attribute `PlotIndex` (CannonService uses it) and lane `OwnerUserId` (CannonService builds / removes cannon + wall on change).
**BoothBuilder** contract and design: see `Claude_old.md`.

---

## Next steps (in order)

1. **Press Play and look**: cannon, LAUNCH button, timing bar, chase camera, trail, debris, the tunnel in the wall from inside and from the side. Tell Claude what to change (colours, sizes, camera, how loud the shake is).
2. Decide the wall persistence (reset per launch or progressive digging) and the progression numbers (how far a PERFECT should go at start).
3. Wire upgrades: Cannon Power (`MAX_SPEED`), Blast Radius (`BlastLevel` attribute), a drain/toughness upgrade; hook into the coins system (PlotService has `DEV_FREE_UPGRADES = true` + a TODO).
4. Rewards: coins/ore from destroyed blocks (data is in `WallBuilder.applyBlast`), distance milestones, leaderboard.
5. Sound (launch boom, wall crunch, stuck thud) and a level-up VFX for booths.
6. Playtest with 2+ players (4 lanes) and on mobile; check part count / performance after many launches.
7. Remove the unused leftovers in `LaunchConfig`; move the Studio-only scripts into Rojo `src/` if we want them in git.
