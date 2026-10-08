# Claude.md — Human Wrecking Ball (placeId 139473074201684)

Working notes so any session can pick up where we left off.
**Update this file after every change.** Last updated: 2026-10-07.

---

## What we are doing right now

**Current focus (2026-10-07): the launch mechanic.** Done: invisible launch barriers, a default cannon per lane owner, the timing-bar launch (see "Cannon + launch system"). Next: wall blocks and the momentum model.

Before that the map was reworked into a "+" shape on a circle (below):

Reworking the map area into a **"+" shape on a circle**:

- Circular island, centre at (0, 0, 0), rim wall at radius 980.
- Circular hub plaza in the centre (radius 70) with a monument.
- Four arms (N, E, S, W). Each arm = **plot on the inner side**, then that plot's **lane** going outward.
- Lanes are large (80 wide, 720 long). Plots are deliberately small (120 x 120) so walking across one takes about 7s at WalkSpeed 16.

### Status

| Item | State |
|---|---|
| "+" layout generated, plots inner / lanes outer | DONE |
| Plot size reduced (180 -> 120), booths ring a circular plaza | DONE |
| Lane width 48 -> 80 | DONE |
| Circular island + rim wall + decor placed in wedges between arms | DONE |
| Old "PLOT 3" label bug on Plot2 (old numbering was scrambled) | FIXED (plot i is always arm i) |
| Lane ceiling was an opaque white slab (blocked sun, hid biome floors) | FIXED (invisible, still solid, no shadow) |
| Numeric checks (positions, slots, attributes, pad overlap) | DONE, all good |
| Launch barrier on every lane (invisible, blocks walking into Stone / ore zones) | DONE, playtested |
| Default cannon for every lane owner (built on claim, removed on release) | DONE, playtested (look not checked by eye) |
| Enter cannon -> timing bar -> server scores press -> flight -> result -> back to plot | DONE, playtested end to end |
| Wall blocks, momentum loss per block, ore drops | NOT STARTED (flight is free flight with constant deceleration for now) |
| Hub tree lights: 8 bulbs on a small black cable ring, tied to the trunk with 4 cables (`Hub.TreeLights`) | DONE (not yet seen in viewport) |
| **Visual check of lane interior / booths in play** | **NOT DONE** (viewport screenshots came back blank; check by eye in Studio) |
| Playtest: join, claim plot, spawn, booth build, upgrades | NOT DONE |
| Lane wall blocks / cannon / wrecking-ball gameplay | NOT STARTED (no script yet) |

---

## Cannon + launch system (built 2026-10-07, playtested with a real client)

All four scripts live in Studio only (not in the Rojo `src/` folders):

| Script | What it does |
|---|---|
| `ReplicatedStorage.Shared.LaunchConfig` (ModuleScript) | All tunables + the pure math (bar pointer position, power from pointer, flight speed / time / distance). Server and client both use it, so what the player sees is what the server scores. |
| `ServerScriptService.CannonBuilder` (ModuleScript) | `build(lane, parent)` cannon model, `buildBarrier(lane)`, `fire(cannon)` (recoil + smoke). Style copied from BoothBuilder (village palette, lane colour on hubs + mouth ring). |
| `ServerScriptService.CannonService` (Script) | At start: one invisible `LaunchBarrier` per lane, `Workspace.Map.Cannons`, `ReplicatedStorage.LaunchRemotes`, collision group `Flying`. Builds `Cannon<i>` when `Lane<i>` attribute `OwnerUserId` becomes a user id (PlotService sets it), destroys it when it goes back to 0. Runs the launch flow. |
| `StarterPlayerScripts.LaunchController` (LocalScript) | Builds the timing-bar UI in code (no StarterGui objects), holds the character in the barrel, plays the flight, camera + FOV kick, hides the Enter prompt for non-owners. |

**Barrier:** `Lane<i>.LaunchBarrier`, created at runtime, so it is not visible in Edit mode. Invisible, solid, `CanQuery=false`. 112 x 120 x 4, centred 4 studs in front of `WallStart` (R 260-264, between the pad end at 258 and the first wall block at 266). Players in collision group `Flying` pass through it.

**Flow:**
1. ProximityPrompt `Cannon1..4.PromptAnchor.EnterPrompt` (range 20, instant). Server checks lane owner. Player goes into group `Flying` (collides with nothing) + `PlatformStand`; server sends `EnterCannon(seatCFrame, barStart)`.
2. Client holds the character at the seat (inside the barrel, head first), fixed camera behind the cannon, bar shown. Pointer = triangle wave of server time (`GetServerTimeNow()`), so server and client agree without sending positions.
3. Press (Space or the LAUNCH button): client sends its server-time of the press. Server clamps it to at most 0.35 s in the past, computes pointer position -> power + rating (PERFECT / GREAT / GOOD / WEAK) -> speed, duration, distance. Sends `LaunchStarted(rating, power, origin, direction, speed, startTime)`; the flight starts 0.25 s later (cannon fires then).
4. Client moves its own character along the lane with `LaunchConfig.distanceAt` (same formula as server). Server never trusts the client position. After the flight the server stores `LastDistance` / `BestDistance` (player attributes) and sends `LaunchFinished(distance, isBest)`; 2.5 s later the player is teleported to the plot `Spawn` (`ExitCannon`).
5. LEAVE button = `LeaveCannon` remote, only while aiming. Respawn / leaving mid-launch clears the state.

**Player attributes:** `CannonState` ("Aiming" / "Launching" / nil), `LastDistance`, `BestDistance` (studs from the seat, not yet "depth into the wall").
**Remotes** (`ReplicatedStorage.LaunchRemotes`): EnterCannon, PressLaunch, LeaveCannon, LaunchStarted, LaunchFinished, ExitCannon.

**Tunables (`LaunchConfig`):** `BAR_SPEED 1.1`, zone half widths `PERFECT 0.04 / GREAT 0.09 / GOOD 0.24`, `MIN_POWER 0.2`, `PERFECT_BONUS 1.15`, `MAX_SPEED 400`, `DECEL 120`. With these a PERFECT flies about 880 studs in 3.8 s, GREAT about 470, the worst hit about 25. Cannon geometry (barrel height 10.5 above the pad top, barrier offset) is at the top of `CannonBuilder`.

**Playtest results (Studio, real client):** walking into the lane stops at the barrier (also after a launch); prompt enters the cannon; Space scored GREAT, flew 473 studs, client end point matched the server distance, player came back to the plot spawn, camera / FOV / collision group restored; LEAVE works; prompt hidden while inside. NOT checked by eye: cannon look, bar layout, camera feel, mobile.

**Known simplifications / ideas:**
- Flight is straight and horizontal at height about 11.5 (barrel height), no arc, gravity ignored. The wall is 64 high, so right now a launch hits the lower part of it. Options: raise the barrel, or tilt the flight upward.
- Momentum model is just constant deceleration. When wall blocks exist, replace `distanceAt` with the real model (block hardness drains momentum) and compute it on the server along the same straight path.
- Cannon upgrades (Cannon Power, ...) should change `MAX_SPEED` per player and the cannon model look.

---

## Hub tree lights (redone 2026-10-05)

The wishing tree in the hub centre (`Hub["Tree 2"]`, trunk radius 1.3-3 up to y about 17, branches and canopy from y 18 to 41) first had 24 loose neon balls. Then a big ring (radius 19.5) that floated around the canopy. Now `Hub.TreeLights` (Model, 64 parts, all `CanCollide=false`, `CastShadow=false`):
- Small ring: radius 9, hangs at y=15 (under the canopy, so it is visible), sags 1.2 studs between bulbs.
- 8 `FairyLight` Neon balls (1.4 studs, warm `255,200,120`), each on a short black `BulbStub`. 40 black `Cable` pieces (0.3 thick) form one closed loop.
- 4 `TrunkCable` cables (every second bulb) run from the ring to the trunk at y=16.5. End points were raycast to the trunk surface (trunk radius 1.3-2.8 at the 4 angles) and each ends in a `TrunkClamp` ball (0.8).
- Tweak `ringR, ringY, sag, count, steps, attachY, trunkR` in the tree lights block of `ServerStorage.MapGenerator` (`Gen.build`), or edit the live parts.
- The generator was updated too (compiles), but its fixed `trunkR = 2.4` is for the imported `Tree 2`; the generator still builds its own procedural tree. Re-running `build()` wipes `Hub` (and `Tree 2` and the leaderboards with it), so do not rebuild without re-adding them.
- Studio `studio_id` changed during this session (reconnect): always call `list_roblox_studios` first.

---

## Layout spec (studs)

Arm numbering is clockwise: **1 = North (-Z), 2 = East (+X), 3 = South (+Z), 4 = West (-X)**.
Plot N always pairs with Lane N. Distances below are measured outward from the centre.

| Thing | Distance from centre | Size |
|---|---|---|
| Hub plaza | 0 - 70 | radius 70 (+4 kerb ring) |
| Plot | 60 - 180 | 120 x 120 |
| Booth ring (Shop, Sell, Smelter) | ring radius 36 around plot centre | pad diameter 48 |
| Cannon pad | 180 - 206 | 80 wide, 26 deep |
| Lane floors (6 biomes x 120) | 180 - 900 | 80 wide |
| Wall start (blocks begin) | 212 | WallLength 720, ends 932 |
| Lane side walls (teal) | 180 - 932 | 8 thick, 44 high |
| End wall | 932 - 956 | 96 wide |
| Rim wall (96 segments) | radius 980 | 60 high |
| Grass ground disc | radius 1020 | |

Booth positions per plot (local frame): Shop left, Sell right, Smelter on the hub side. The lane side stays open so players can walk straight to the cannon. All booths face the plot centre (PlotService `slotCFrame` does this using the plot's `CenterX` / `CenterZ`).

Spawn sits 18 studs lane-side of plot centre and faces down the lane.
Plot sign is on the hub-side arch and faces the hub.

Biome order along each lane: Dirt, Stone, DeepRock, Crystal, Obsidian, MagmaCore.
Per-arm colours come from the existing CannonPad colour of each lane (Lane1 pink; defaults pink / blue / green / amber).

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

- All sizes live in `Gen.CFG` at the top of the module. Change numbers there and re-run.
- `build()` wipes and recreates: Map.Lanes, Plots, Structure, Hub, Ground, Rim, and deletes Map.PlotFences (fences now live inside each plot).
- It re-uses existing parts as templates (Sign + SurfaceGui, Spawn, Pad, Floor_* ...) so look and feel carry over. Do not rename those parts.
- Decor trees and boulders are re-placed in the wedges between arms (seeded random, deterministic). 70 extra trees go in `Decor.WedgeTrees`.
- Backup of the previous T-layout map: `ServerStorage.OldMap_Backup`. Delete both ServerStorage items before publishing if you do not want them shipped.

### Gotchas
- `require` caches. After editing the module Source you must clone it (as above) or the old code runs.
- Cylinder parts need `* CFrame.Angles(0, 0, pi/2)` to lie flat (axis is X).
- Roblox parts cap at 2048 per axis, so the ground disc is 2040 wide and the rim radius is 980.
- Do not put coplanar parts at the same Y (z-fighting). Ground discs are stacked 0.02-0.05 apart on purpose.

---

## Contracts other scripts rely on

**PlotService** (`ServerScriptService.PlotService`, unchanged) expects:
- `Workspace.Map.Plots.Plot1..4`, each with `Sign` (direct child, SurfaceGui > TextLabel named `Text`), `Spawn`, and models `Slot_Shop`, `Slot_Smelter`, `Slot_Sell` each containing an `Origin` part.
- Plot attributes `CenterX`, `CenterZ` (world coords of plot centre), `PlotIndex`, `LaneIndex`, `OwnerUserId`.
- `Workspace.Map.Lanes.Lane1..4` with attribute `OwnerUserId`.
- Map attribute `Plots` (max players, 4).

**Lane attributes** (for the future wall / cannon script):
`LaneIndex, OwnerUserId, WallHeight (32), BlockSize (4), WallLength (720), WallWidth (80), LaneStartR (180), WallStartR (212), WallStart (Vector3, world, ground level, centre of lane), LaneDirection (Vector3, unit, outward), LaneYawDeg`.

NOTE: old lanes used `CenterX` / `WallStartZ` because every lane ran along -Z. Lanes now point in four directions, so use `WallStart` + `LaneDirection` instead. `CenterX` and `WallStartZ` no longer exist.

**Lane attributes verified in Studio 2026-10-07** (the numbers in the layout table above are older): `WallWidth 104` (lane floors are 104 wide, dividers at +-56), `LaneStartR 218`, `WallStartR 266`, `WallLength 1752`, `WallHeight 64`, `LaneHeight 120` (invisible ceiling at y 120.5), `LaneLength 1800`. Cannon pad = R 218-258 (40 deep). Plots also have a `Slot_Forge` booth now.

**Map attributes:** `Layout="Plus", Plots, Lanes, BlockSize, LaneWidth, PlotSize, PlazaRadius, RingRadius, LaneStartR`.
(`HubHalfWidth`, `HubDepth` removed.)

---

## Next steps (in order)

00. Press Play and look at the cannon, the timing bar (size, colours, mobile) and the launch camera; tell me what to tweak.
0. Look at the new tree lights in Studio (ring visible under the canopy, spokes reaching the trunk, nothing clipping into branches at y 18-24). If it clips, change `ringY` / `attachY`.

1. **Look at it in Studio**: overhead view, a plot with booths, the inside of a lane. Note anything mis-scaled against the character (booths go up to about 1.45x at level 5, sign stars reach about 30 studs high).
2. **Playtest**: join, check the plot claim, spawn position and facing, that all 3 booths build inside the plot, upgrades via the ProximityPrompt, and leave / rejoin.
3. Check booth footprint at level 5 against the plot edge and neighbouring booths (calculated to fit, never seen in game).
4. Polish pass on the empty wedges (more variety, paths, props) if it feels bare. Currently only grass, trees and boulders.
5. Wall blocks: generate them from the lane attributes (start at `WallStartR` 266, 104 x 64, `BlockSize` 4), stream in chunks. Then replace the constant deceleration with the real momentum model (see the cannon section).
6. Optional: tune plot size via `PlotSize` / `SlotRing` if 120 still feels big or small.

## Open questions for the owner
- Is the lane length (720) right now that width is 80, or should lanes be shorter / longer? (Changing it means changing the rim radius too.)
- Should the lane ceiling stay as an invisible solid lid, or be removed?
