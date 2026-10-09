# Claude.md — Human Wrecking Ball (placeId 139473074201684)

Working notes so any session can pick up where we left off.
**Update this file after every change.** Last updated: 2026-10-09.

---

## What we are doing right now

**Current focus (2026-10-09): launch feel (arc, wall resistance) and readability. Round 3 is the latest, see below.** Round 1: blocks were huge, wall too close to the cannon, blocks not removed fast enough, camera inside blocks -> fixed (see "Wall + launch fixes 2026-10-09"). Round 2 (same day): flight was too curved and dropped to the floor, player should stop inside the wall, and the flight was too fast with too many effects -> fixed (see "Round 2"). Waiting for the owner to try it by eye.

Before that: launch mechanic (barrier, default cannon, timing bar, server-side physics flight, wall blasting) and the "+" map layout on a circle (below).

### Status

| Item | State |
|---|---|
| "+" layout generated, plots inner / lanes outer | DONE |
| Circular island + rim wall + decor in wedges | DONE |
| Launch barrier on every lane, default cannon per lane owner | DONE |
| Cannon -> timing bar -> server-scored press -> server-physics flight -> result -> back to plot | DONE |
| Destructible wall (`WallBuilder`), blast cone, speed drain per hardness | DONE (rebuilt 2026-10-09: 4 stud blocks, tiles, 160 stud runway) |
| Chase camera that stays inside the tunnel | DONE 2026-10-09 (tested: never inside a block) |
| **Owner check by eye: block size, runway length, camera feel** | **NOT DONE** |
| Ore drops from blasted blocks, upgrades (Cannon Power, Blast level) | NOT STARTED (BlastLevel player attribute exists, nothing sets it) |
| Playtest: join, claim plot, booth upgrades, leave / rejoin | NOT DONE |
| Hub tree lights (`Hub.TreeLights`) | DONE, not yet seen in viewport |

---

## Wall + launch fixes 2026-10-09

All edits are in Studio only (not in the Rojo `src/` folders).

| Problem | Fix |
|---|---|
| Blocks 8 studs = too big | `LaunchConfig.WALL_BLOCK = 4`. Cells are still data; one slab per **tile** (`WALL_TILE` 4 x 4 blocks = 16 studs wide/high, `WALL_CHUNK_LAYERS` 4 layers deep). A tile becomes real 4-stud blocks only when a blast touches it. 4900 slabs per wall at start, a launch fractures only the tiles along its tunnel. |
| Wall started 8 studs behind the pad | `LaunchConfig.WALL_GAP = 160`. `WallBuilder` shifts the wall start by this much (lane attributes untouched, so barrier / map generator are unchanged). Hardness depth is still measured from `LaneStartR`, so the biome floors match the wall (the wall starts around hardness 1.7). |
| Blocks around the camera / not destroyed instantly | (1) Server also clears a ball just ahead of the nose (`BLAST_LOOKAHEAD_TIME 0.05` s, max `BLAST_LOOKAHEAD_MAX 14` studs, radius x0.9; skipped when about to get stuck). (2) Blast uses distance to the block BOX, not its centre, so even a small blast clears the cell the player is in and the body (radius 3) is really free. (3) Client chase camera now rides the flown path: `CAM_BACK 30` studs behind along the path, `CAM_LIFT 2` above it (extra height only while the flight is still short), so it is always inside the tunnel the blasts cleared. |
| Camera / character jumped to the flight before the server launched | Client holds the character in the barrel and keeps the aim camera until `LaunchStarted` arrives (`launched` flag in `LaunchController`). |

Test result (Studio, real client, every press forced to PERFECT in a play session only): wall built with 4900 slabs, front exactly 160 studs behind `WallStart`; flight entered the wall, 64 tiles / ~2700 blocks fractured along the tunnel, camera was never inside a block at any sample.

### Round 2 (2026-10-09, later): straight flight, stop in the wall, clarity

- **Flight is now our own simulation.** `CannonService.trackFlight` keeps its own `vel` (no longer reads the velocity back from the engine; that read-back was the cause of the unexplained extra slowdown). Only `LaunchConfig.FLIGHT_GRAVITY` (0.04) of gravity acts; the engine's own gravity step is added back to the written velocity while airborne. Pitch 5 deg, `MAX_SPEED 220` (was 400), `DRAG 0.1`, `DRAIN_K 0.0006`.
- **Stops inside the wall:** when stuck (blast too small or speed < `MIN_SPEED`) the player is frozen in place (`frozenAt`), 0.4 s later anchored where they are; `sendHome` un-anchors. Test (forced PERFECT): flew about 6 s on a nearly straight line (peak y about 83), embedded at y 77, about 570 studs into the wall (36 % of its length), distance 757. Weaker presses stop earlier.
- **Clarity:** `Vfx` toned way down: shake about 10 x weaker, blast effects at most every 0.2 s (was 0.07) with fewer / slower debris, dust and sparks, small point light, no shockwave or screen flash except a single soft shockwave on the final impact; fire trail / smoke much smaller. `LaunchController`: launch flash 0.1, rating text smaller, moved to the top and hidden after 1.2 s, result text at the top, FOV kick 10 (was 38), `CAM_BACK 36`.
- Not checked by eye yet (only logged positions): how readable it really is. Further knobs: `MAX_SPEED`, `CAM_BACK` / `CAM_LIFT`, `Vfx.blast` throttle, or a slow-motion factor on the flight.

### Round 3 (2026-10-09, later): real arc + the wall must slow you down

Owner feedback on round 2: too "locked in", not a trajectory, and blasting through 720 studs of blocks at the start gives no feeling of resistance.
- **Arc is back, moderate:** `PITCH_DEG 12`, `FLIGHT_GRAVITY 0.3` (30 % of gravity in open air). Forward speed / apex are readable over the 160 stud runway (peak about y 50).
- **Wall resists strongly:** `DRAIN_K 0.006` (10 x round 2). Test (forced PERFECT): hits the wall at about 230 studs/s, down to about 45 within roughly 130 studs (about 1 s), stuck and embedded at y 55, distance 317. Weaker presses stop sooner; going deeper is meant to come from upgrades (cannon power -> `MAX_SPEED`, blast level). Harder biomes drain more because drain = `DRAIN_K * hardness`.
- **No nosedive inside the wall:** `WALL_GRAVITY 0.15` multiplies gravity again while inside the wall, so the player holds their height and freezes mid-air where they run out of speed (before, they slid to the floor).
- **Camera less rigid:** in open air it floats `CAM_LIFT_AIR 9` above the flown path (so the arc reads), and drops smoothly to `CAM_LIFT 2` (inside the tunnel) once the camera point is within 14 studs of the wall. FOV kick 14 scaled by speed, so it visibly narrows as the wall slows you.
- Not checked by eye. Knobs: `DRAIN_K` (resistance), `MAX_SPEED`, `FLIGHT_GRAVITY` / `PITCH_DEG` (arc), `CAM_LIFT_AIR`, `CAM_BACK`.

Known / not fixed:
- At the end of a flight the camera can hang above the player (30 studs back along a descending path). Fine for now.
- Part count: a long tunnel can leave several thousand real blocks. If the server hitches, raise `WALL_BLOCK` back to 5-6 or reduce `WALL_HEIGHT`.
- `LaunchConfig.simulateFlight`, `pathAt`, `velocityAt`, `flight` are old and unused (the server now uses real physics in `CannonService.trackFlight`).

---

## Cannon + launch system (current behaviour)

| Script | What it does |
|---|---|
| `ReplicatedStorage.Shared.LaunchConfig` (ModuleScript) | All tunables + pure math (bar pointer, power, hardness per depth, blast radius). |
| `ReplicatedStorage.Shared.Vfx` (ModuleScript) | Client VFX: screen flash, shake, blast burst, fire trail. |
| `ServerScriptService.CannonBuilder` (ModuleScript) | `build(lane, parent)` cannon, `buildBarrier(lane)`, `fire(cannon)`. Cannon has attributes `SeatCFrame`, `FlightDirection`, child `LaunchZone`. |
| `ServerScriptService.WallBuilder` (ModuleScript) | Destructible wall (see contract at the top of the script). Built per lane owner by CannonService, reset when the lane is reclaimed. |
| `ServerScriptService.CannonService` (Script) | Collision groups, remotes, barrier per lane, cannon + wall per owned lane, enter / press / flight / result flow. |
| `StarterPlayerScripts.LaunchController` (LocalScript) | LAUNCH button (shown inside the cannon's `LaunchZone`), timing-bar UI built in code, holds the character in the barrel, camera, FOV, blast VFX, result text. |

**Flow:**
1. Owner stands in `Cannon<i>.LaunchZone`, client shows LAUNCH -> `RequestEnter`. Server checks owner + zone, puts the character in group `Flying`, PlatformStand, sends `EnterCannon(seatCFrame, barStart)`.
2. Client holds the character at the seat, fixed camera behind the cannon, bar pointer = triangle wave of server time.
3. Press (Space / FIRE button): client sends its server-time of the press. Server clamps it (max 0.35 s in the past), scores PERFECT / GREAT / GOOD / WEAK, fires the cannon, sets the character's network owner to nil and launches it with a real velocity (`MAX_SPEED * power`, `PITCH_DEG` 22). Fires `LaunchStarted`.
4. `trackFlight` (server, every Heartbeat): adds forward drag + ground friction, then for the path since last frame (sub-steps of half a block): blast radius from speed / hardness / BlastLevel, destroy blocks (`wall:applyBlast`), drain speed (`DRAIN_K * hardness` per stud), stuck when radius < `MIN_BLAST` or speed < `MIN_SPEED`. Blast events go to the client live (`LaunchVfx`).
5. Result: `LastDistance` / `BestDistance` (player attributes), `LaunchFinished`; after `RESULT_TIME` the player is teleported to the plot `Spawn` (`ExitCannon`). LEAVE only while aiming.

**Collision groups:** `Flying` (collides only with `LaneFloor`), `LaneFloor` (Floor_* + CannonPad), everything else `Default`. The `LaunchBarrier` is in `Default`, so flying players pass through it.
**Remotes** (`ReplicatedStorage.LaunchRemotes`): RequestEnter, EnterCannon, PressLaunch, LeaveCannon, LaunchStarted, LaunchVfx, LaunchFinished, ExitCannon.
**Player attributes:** `CannonState` ("Aiming" / "Launching" / nil), `LastDistance`, `BestDistance`, `BlastLevel`.

**Key tunables (`LaunchConfig`):** `BAR_SPEED 1.1`, zone half widths `0.04 / 0.09 / 0.24`, `PERFECT_BONUS 1.15`, `MAX_SPEED 220`, `PITCH_DEG 12`, `GRAVITY 196.2`, `FLIGHT_GRAVITY 0.3`, `WALL_GRAVITY 0.15`, `DRAG 0.1`, `SLIDE_DECEL 120`; wall: `WALL_BLOCK 4`, `WALL_TILE 4`, `WALL_CHUNK_LAYERS 4`, `WALL_GAP 160`, `WALL_HEIGHT 112`; blast: `BLAST_BASE 22`, `BLAST_MAX 56`, `MIN_BLAST 3.2`, `DRAIN_K 0.006`, `MIN_SPEED 45`, `BLAST_LOOKAHEAD_TIME 0.05`, `BLAST_LOOKAHEAD_MAX 14`; hardness per 300 stud biome `{1, 2, 3.5, 5.5, 8, 12, 18}`.

**Testing trick (play session only, nothing saved):** in a Server `execute_luau`, `require(ReplicatedStorage.Shared.LaunchConfig).PERFECT_HALF = 0.5` makes every press PERFECT. Teleport the character to `Cannon1.LaunchZone` from the server, fire `RequestEnter` from the Client, press Space with `user_keyboard_input`. Restart play to get a fresh wall.

---

## Hub tree lights (redone 2026-10-05)

`Hub.TreeLights` (Model, 64 parts, all `CanCollide=false`, `CastShadow=false`) around the wishing tree `Hub["Tree 2"]`: small ring radius 9 at y=15 (sag 1.2), 8 `FairyLight` Neon balls (warm `255,200,120`) on black stubs, 40 black `Cable` pieces forming a closed loop, 4 `TrunkCable` cables to the trunk at y=16.5 ending in `TrunkClamp` balls. Tweak `ringR, ringY, sag, count, steps, attachY, trunkR` in the tree lights block of `ServerStorage.MapGenerator` or edit the live parts. Re-running `build()` wipes `Hub` (and `Tree 2` and the leaderboards with it), so do not rebuild without re-adding them. `studio_id` changes between sessions: always call `list_roblox_studios` first.

---

## Layout spec (studs)

Arm numbering is clockwise: **1 = North (-Z), 2 = East (+X), 3 = South (+Z), 4 = West (-X)**. Plot N always pairs with Lane N. Distances are measured outward from the centre.

| Thing | Distance from centre | Size |
|---|---|---|
| Hub plaza | 0 - 70 | radius 70 (+4 kerb ring) |
| Plot | 60 - 180 | 120 x 120 |
| Booth ring (Shop, Sell, Smelter, Forge) | ring radius 36 around plot centre | pad diameter 48 |
| Lane floors (6 biomes) | see verified lane attributes below | |
| Rim wall | radius 980 (older number, check) | 60 high |
| Grass ground disc | radius 1020 | |

Booth positions per plot (local frame): Shop left, Sell right, Smelter on the hub side. The lane side stays open. All booths face the plot centre. Spawn sits 18 studs lane-side of plot centre facing down the lane. Plot sign is on the hub-side arch.

Biome order along each lane: Dirt, Stone, DeepRock, Crystal, Obsidian, MagmaCore (300 studs each).

**Lane attributes verified in Studio (2026-10-07, newer than the older numbers):** `WallWidth 104`, `LaneStartR 218`, `WallStartR 266`, `WallLength 1752`, `WallHeight 64` (attribute only; the real wall height is `LaunchConfig.WALL_HEIGHT` 112), `LaneHeight 120` (invisible ceiling at y 120.5), `LaneLength 1800`, `WallStart` (Vector3, world, ground level, centre of lane; Lane1 = 0,0,-266), `LaneDirection` (unit, outward), `BlockSize`, `LaneYawDeg`, `LaneIndex`, `OwnerUserId`. Cannon pad = R 218-258. The wall itself starts `WALL_GAP` studs after `WallStart` (R 426 on every lane).

---

## How the map is generated (read this before touching Workspace.Map by hand)

`ServerStorage.MapGenerator` (dev tool, edit mode only). Run from the command bar or MCP `execute_luau` (Edit datamodel):

```lua
local SS = game:GetService("ServerStorage")
local fresh = SS.MapGenerator:Clone()   -- clone, because require() caches the module
fresh.Parent = SS
print(require(fresh).build())
fresh:Destroy()
```

- Sizes live in `Gen.CFG`. `build()` wipes and recreates Map.Lanes, Plots, Structure, Hub, Ground, Rim.
- Re-uses existing parts as templates (Sign + SurfaceGui, Spawn, Pad, Floor_* ...), do not rename them.
- Backup of the previous T-layout map: `ServerStorage.OldMap_Backup`. Remove the ServerStorage backups before publishing if you do not want them shipped.

### Gotchas
- `require` caches. Clone the module after editing it (as above).
- Cylinder parts need `* CFrame.Angles(0, 0, pi/2)` to lie flat.
- Parts cap at 2048 per axis.
- No coplanar parts at the same Y (z-fighting).
- Studio must be in **Edit** mode to modify scripts; during Play only Server / Client datamodels exist.
- StreamingEnabled is on: the client only sees part of the 4900 wall slabs at a time, so part counts on the client are lower than on the server.

---

## Contracts other scripts rely on

**PlotService** (`ServerScriptService.PlotService`, unchanged) expects: `Workspace.Map.Plots.Plot1..4`, each with `Sign`, `Spawn`, models `Slot_Shop`, `Slot_Smelter`, `Slot_Sell` (and `Slot_Forge`) each containing an `Origin` part; plot attributes `CenterX`, `CenterZ`, `PlotIndex`, `LaneIndex`, `OwnerUserId`; `Workspace.Map.Lanes.Lane1..4` with attribute `OwnerUserId`; map attribute `Plots` (4). It sets the `PlotIndex` player attribute that CannonService uses (plot N <-> lane N / cannon N).

**BoothBuilder** (v2 "Village Stalls", backup `ServerStorage.BoothBuilder_V1_Backup`): `MAX_LEVEL` 5, `build(kind, cf, level, parent)` returns Model `<Kind>Booth` with `UpgradeAnchor` (CanQuery true), attributes `Station`, `Level`, footprint 30 x 24, parts `InteractZone` and `Platform`.

**Map attributes:** `Layout="Plus", Plots, Lanes, BlockSize, LaneWidth, PlotSize, PlazaRadius, RingRadius, LaneStartR`.

---

## Next steps (in order)

1. Owner: press Play, launch, and tell me if the 4 stud blocks, the 160 stud runway and the camera feel right (tweak `WALL_BLOCK`, `WALL_GAP`, `CAM_BACK` / `CAM_LIFT` in `LaunchController`).
2. Owner: judge readability of the flight (speed, effects, camera) and tell me which knob to turn; then balance distance vs hardness.
3. Ore / coin drops from destroyed blocks, wire `BlastLevel` and cannon power to upgrades.
4. Playtest plot claim, spawn, booths, upgrades, leave / rejoin; booth footprint at level 5 vs plot edge.
5. Look at the tree lights and the empty wedges in Studio; polish if bare.

## Open questions for the owner
- Is 4 studs the right block size, or should it be even smaller / a bit larger?
- Is 160 studs of runway before the wall right?
- Should the lane ceiling stay as an invisible solid lid, or be removed?
