# Project Guidelines: Evil Magicians

These are project-specific conventions, tools, paths, and patterns for the Evil Magicians Unity project. The Manager includes this file in every agent's prompt alongside their generic role definition.

## Project Overview

Evil Magicians is a Unity 2D multiplayer game (4-player couch co-op) with magic combat, AI-controlled monsters, and clan-based PvP. Unity 6, C#, MonoBehaviour-based.

## Stable State

The stable state of this project is **all tests passing**. Any pipeline that introduces failures must fix them before completing. The Debugger's job is to return the project to this stable state.

## Documentation Structure

| Type | Location | Naming |
|------|----------|--------|
| Game Design | `Documentation/GDD/GDD_{Feature}.md` | Player-facing behavior |
| Technical | `Documentation/Technical/{Feature}_Technical.md` | Developer-facing architecture |
| Test Specs | `Documentation/Testing/Tests_{Feature}.md` | Test scenarios in table format |
| Plans | `Documentation/Plans/{Module}/{Task}_Plan.md` | Task plans and status |
| Master Index | `Documentation/README.md` | Links to all docs |
| Plan Index | `Documentation/Plans/general.md` | All plans with status |

### Documentation Rules
- No hardcoded gameplay numbers — point to prefab/config as source of truth
- Three-tier test structure: Tier 1 (EditMode logic), Tier 2 (PlayMode API), Tier 3 (PlayMode full-scale)
- Update `Documentation/README.md` when creating new docs
- Update `Documentation/Plans/general.md` when completing work

## Error Handling Conventions

- **No silent failures.** Never use `default: break;` or `default: return;`
  - DO: `default: MagiciansLog.Error($"Unexpected state: {state}"); throw new InvalidOperationException(...);`
- **No defensive null checks on singletons.** If `AudioManager.Instance` is null, that's a bug — crash loudly
- **No fallbacks that hide configuration errors.** Missing config = `MagiciansLog.Error()` + fail visibly
- **Use `MagiciansLog.Error()`, not Warning**, for non-recoverable situations
- **Include context** in error messages: what was expected vs what was received

## Test Infrastructure

### Test Base Classes
- **EditMode**: Extend `EditModeTestBase` (from `Assets/Tests/Shared/EditModeTestBase.cs`)
  - Location: `Assets/Tests/EditMode/{Module}/`
  - Pure logic tests — no Unity scene, no MonoBehaviour
- **PlayMode**: Extend `PlayModeTestBase` (from `Assets/Tests/PlayMode/PlayModeTestBase.cs`)
  - Location: `Assets/Tests/PlayMode/{Module}/`
  - Real game scene loaded, real players, real monsters
  - `LogAssert.ignoreFailingMessages = true` at the START of each `[UnityTest]` method
  - Use `OnPerTestSetUp()` / `OnPerTestTearDown()` for per-test cleanup
- **Diagnostics**: `Record("key", value)` adds data to failure reports in `MyLogs/TestLogs/{TestClass}/{TestMethod}.log`

### Assembly Definitions
- `Assets/Scripts/GameCode.asmdef` — game code
- `Assets/Tests/EditMode/EditMode.asmdef` — EditMode tests
- `Assets/Tests/PlayMode/PlayMode.asmdef` — PlayMode tests

### Camera Lock (Automatic)
PlayModeTestBase automatically locks the camera to show the entire map via `LockCameraToFullMap()` in `SetUp()`. This eliminates camera-related test interference:
- No projectile destroyed by `MustBeOnScreen` (entire map is visible)
- Camera border walls are at map edges (nothing pushed back)
- Camera does not react to player movement or ghost state
- `UpdateCameraMode()` never runs, so `Framework.IsTraversing` is not reverted by camera

**Opt-out**: Tests needing manual camera control override `protected virtual bool AutoLockCamera => false;` (used by showcase tests).

**Helpers**: `GetCameraBounds()` returns cached `cameraBounds` instance. `RecordCameraState()` adds camera diagnostics to failure reports. `WaitForCameraStable(timeout)` waits for camera to settle after unlocking.

### Test Helpers
- **`FreezeMonsterBehavior(monster)`** — Stops behavior evaluation. Monster holds still, remains damageable. Use for projectile tests against stationary targets. Note: AnimatorMovement continues independently — use `PinMonsterPosition` alongside this for exact position control.
- **`PinMonsterPosition(monster)`** — Pins a monster at its current position by zeroing Rigidbody2D velocity and resetting `transform.position` every LateUpdate via an attached `MonsterPositionPin` MonoBehaviour. Use after `FreezeMonsterBehavior` to prevent AnimatorMovement drift. Call `UnpinMonsterPosition(monster)` to remove the pin.
- **`ForceMonsterAttack(monster, angle, weaponIndex)`** — Forces monster to fire weapon immediately. Bypasses behavior logic. Use for testing projectile mechanics.
- **`ClearCommonAttackBlockers(monster)`** (on `MonsterTestBase`) — Clears the 3 universal attack blockers: movement gate, per-player cooldowns, weapon cooldowns. Called internally by all `ForceStart*` TestContext methods. Use directly only when no monster-specific blocker clearing is needed.
- **`ClearMovementGate(monster)`** (on `MonsterTestBase`) — Clears only the movement gate (`_movementGateActive = false`). Use when only the gate needs clearing but weapon cooldowns must be preserved (e.g., testing CooldownWalk behavior which requires the weapon to be on cooldown). For full attack unblocking, use `ClearCommonAttackBlockers` instead.

### TestContext Pattern (Monster Tests)
Each monster type has a TestContext class (`Assets/Tests/PlayMode/Monsters/Contexts/{MonsterType}TestContext.cs`) that wraps a spawned monster and exposes:
- **Config properties** with doc comments (distances, cooldowns, thresholds) — read from the spawned instance, never hardcoded
- **ForceStart methods** that clear all behavior blockers for triggering a specific behavior (see Integration Test Isolation Checklist)
- **Helper methods** for common test operations (repositioning, suppressing specific behaviors)

Tests create a context object in setUp and use it throughout. This pattern ensures configurable values are read from runtime instances, not hardcoded, and blocker-clearing is complete (not partial).

### Test Batch Filter (PlayMode)
To run only a specific PlayMode test class, write its name to `test_batch.txt`:
```bash
bash .claude/cleanup-init-scenes.sh          # clean stale scenes first
echo "DemonBlockTests" > test_batch.txt       # filter to one class
# run tests via MCP
rm -f test_batch.txt                          # always clean up after
```
- Batch filter = exact test class name
- Omit the file to run all PlayMode tests
- Always clean up with `rm -f test_batch.txt` after tests

### Test Naming
- Test class: `{SystemUnderTest}Tests.cs` for logic, `{Feature}BehaviorTests.cs` for full-scale
- Test method: `{Method}_{Scenario}_{Expected}` (match existing convention in the module)

## Compilation & MCP

- Check compilation via MCP `read_console` after code changes
- Run tests via MCP `run_tests` + `get_test_job`
- If MCP stale: `refresh_unity(mode="force", scope="all", compile="request", wait_for_ready=true)`
- Unity Editor MUST be open for MCP to work
- **MCP tools may not be registered as deferred tools in all sessions.** If MCP tools are unavailable (not in the tool list and not fetchable via ToolSearch), fall back to raw HTTP calls: `curl -s -X POST http://localhost:8080/mcp -H "Content-Type: application/json" -d '{"tool":"run_tests","arguments":{...}}'`. This is a known operational issue — the MCP server must be running AND the tools must be registered in the session. If using curl fallback, parse JSON responses manually.
- **Compilation-only fallback (no MCP needed):** When MCP is unreachable but you only need to verify compilation (not run tests), use `dotnet build EvilMagicians.sln`. This does NOT require the Unity Editor or MCP. Use this as a fast check before attempting MCP-based test runs.

### Pre-Test Cleanup Protocol (MANDATORY before every test run)

The MCP test runner can get stuck in a `tests_running` state that persists across sessions (stored in Unity's `SessionState`). This blocks ALL subsequent test runs. **Always run this protocol before launching tests:**

1. **Clean stale scene files:** `bash .claude/cleanup-init-scenes.sh`
2. **Clean stale test logs:** `bash .claude/cleanup-stale-testlogs.sh`
3. **Clear batch filter:** `rm -f test_batch.txt`
4. **Check editor state** via MCP `editor_state` resource — look at `tests.is_running` and `tests.current_job_id`
5. **If `tests.is_running` is true but no tests are actually running:**
   - **Primary recovery: restart the MCP server.** In Unity Editor: `Window > MCP for Unity`, stop and restart the server. This clears all in-memory job state. The `clear_stuck=true` parameter exists in `RunTests.cs` source code but is **not reliably exposed in the MCP tool schema** — agents cannot depend on it working via MCP client calls.
   - **Fallback**: Run a trivial EditMode test (even one with 0 results resets `tests.is_running` to false). This can be done via MCP or batch mode.
   - **Last resort**: Close and reopen the Unity Editor entirely (clears `SessionState`).
6. **Verify** `editor_state` shows `ready_for_tools=true` and `tests.is_running=false`
7. **Now run actual tests**

### MCP Test Runner Diagnostics

The `get_test_job` response includes stuck-job diagnostic fields:
- **`stuck_suspected: true`** — MCP thinks the test job is stuck
- **`blocked_reason`** — one of: `editor_unfocused`, `compiling`, `asset_import`, or `unknown`

Auto-recovery thresholds built into the MCP test runner:
- **15 seconds**: Initialization timeout — if tests never start after `run_tests`, the job is marked stuck
- **60 seconds**: Per-test stuck detection — if a single test runs for >60s, `stuck_suspected` becomes true
- **5 minutes**: Domain reload orphan cleanup — jobs surviving past a domain reload are cleaned up

Use `blocked_reason` to take corrective action (e.g., click the editor to restore focus, wait for compilation to finish).

## Editor-Time Code Conventions

- **Editor scripts go in `Assets/Editor/`** — code in this directory is excluded from runtime builds and has access to `UnityEditor` APIs
- **Editor assembly**: `Assets/Editor/` uses a separate `.asmdef` (or the default editor assembly). Editor code cannot be referenced by runtime code.
- **Prefab creation**: Use `PrefabUtility.SaveAsPrefabAsset()` for creating prefabs at editor-time, not runtime `Instantiate`
- **EditorWindow**: Custom tools inherit from `EditorWindow` and register via `[MenuItem("Tools/...")]`
- **Undo support**: Editor tools that modify the scene should use `Undo.RegisterCreatedObjectUndo()` for undo support
- **Validation classes go in runtime code, not Editor**: When an editor tool needs to validate data before acting on it, put the validator in the runtime assembly (e.g., `Assets/Scripts/`) so it can be tested in EditMode. The editor tool calls the validator but does not own it. This keeps validation logic testable without editor assembly references.
- **Editor assembly testing**: EditMode tests can reference editor assemblies by adding the editor `.asmdef` to the test `.asmdef`'s references. However, prefer keeping testable logic in runtime classes (validators, pure static methods) so tests don't need editor assembly references at all.
- **SerializedObject for private fields**: When editor tools need to set `[SerializeField]` private fields on components (e.g., wiring references on `LevelManager`), use `SerializedObject` + `FindProperty()` + `ApplyModifiedProperties()`. This is the standard Unity Editor pattern for writing to serialized private fields.

## Code Patterns

### Architecture
- **MonoBehaviour** for game objects, **Singleton** for managers
- **State machines** for monster behaviors (via Unity Animator)
- **Coroutines** for timed events
- **Component architecture** via Unity's component system
- **Extract public static pure methods** for EditMode testability
- **Editor-only singletons** (`#if UNITY_EDITOR`): Singletons that exist only in editor builds (e.g., debug visualization managers) use `FindFirstObjectByType<T>()` instead of auto-creation. They must be placed on a scene GameObject for inspector toggle visibility. Callers use null-conditional access (`Instance?.Method()`) so the system is a no-op in builds and when the singleton isn't in the scene.

### Dormant-by-Default Pattern
When adding new configurable time windows, delays, or optional behaviors to existing systems:
- **Default all new serialized fields to 0** — the system is dormant (inactive) until a designer tunes the value via the prefab inspector
- **Guard activation with `> 0` checks** — when the field is 0, behavior is identical to before the feature was added
- **Tests must respect dormancy** — config-read tests should assert `>= 0` (field exists and is valid), not `> 0` (field is non-zero), unless the test explicitly sets a non-zero value
- This pattern keeps existing gameplay unchanged by default and prevents accidental activation

### OnValidate() for Inspector-Driven Runtime Behavior
`OnValidate()` fires when inspector fields change (both in edit mode and at runtime). Use it to detect toggle changes and trigger scans/cleanup. Rules:
- **Only expensive operations on toggle change** — compare current value to a cached `_prev` field, only act on transitions
- **Use `DestroyImmediate()` in OnValidate**, not `Destroy()` — `Destroy` is not allowed in edit mode
- **`FindObjectsByType<T>(FindObjectsSortMode.None)` for entity scans** — only call on toggle transitions, never per-frame. This is the Unity 6 replacement for `FindObjectsOfType<T>()`

### Key Systems
- `Player`, `Monster`, `Weapon` — core game objects
- `MonsterBehaviourManager` — manages monster AI states and transitions
- `AnimatorMonster` — enhanced monster with Animator integration
- `Framework` — core utility singleton for game state
- `GameManager` — central game state management

### Generator-Suggests / Runtime-Decides Pattern
When a generation system produces spatial data (positions, regions) that a runtime system uses, separate the concerns cleanly:
- **Generator** outputs candidate positions as raw data (`List<Vector2>`, structs)
- **Runtime** decides what to place at those positions (which monsters, difficulty scaling, etc.)
- The generation result struct carries spatial data only — no runtime logic, no references to runtime types
- This pattern applies to corridor spawn points (generator suggests positions, WaveCreator decides which monsters), arena content, and any future procedural placement system

## Test Scoping Defaults

When the user says "run all X tests," include:
1. **Primary**: All test classes in `Assets/Tests/EditMode/{X}/` and `Assets/Tests/PlayMode/{X}/`
2. **Adjacent**: Test classes in OTHER directories that test X-related infrastructure (e.g., for monsters: audio tests for monster sounds, movement tests for monster animations, tuning tests)
3. **Exclude**: Tests that only tangentially mention X but test a different system (e.g., a player test that spawns a monster as a target)

The Tester agent's inventory categorizes tests as "primary" vs "adjacent" — include both by default.

## Config-Driven Test Convention

PlayMode tests that validate timing values, distances, cooldowns, or other configurable parameters must **read the actual value from the config/prefab at runtime**, not hardcode magic numbers. This ensures tests automatically adapt when designers retune values.

**Patterns:**
- **Weapon timing**: Access the weapon on the test player via `testPlayer.getWeapon(type)`, then read properties like `getCurCooldown()`, `FireStartupDelay`, etc.
- **Movement configs**: Load ScriptableObjects via `Resources.Load<counterMovement_config>("Configs/Player/CounterMovementConfig")` (cached in `PlayModeTestBase.LoadCounterConfig()` / `LoadTeleportConfig()`)
- **Wait durations**: Compute from config values with a small margin: `WaitSeconds(config.cooldownTime + 0.2f)`
- **Animation/effect waits**: Short hardcoded waits (1-2 seconds) for "wait for effect propagation" are acceptable when no single config field governs the duration

## Contact Damage → Invulnerability → closestPlayer=-1 Cycle

When a monster's walk step moves it close enough for contact damage, this triggers `Player.getHit()` which applies 0.5s invulnerability. While invulnerable, `Player.isAttackable()` returns false. In `MonsterManagerContext.updateContext()`, non-attackable players get `float.MaxValue` distance. If all non-test players are already killed (via `KillOtherPlayers`) and the test player is temporarily invulnerable, ALL players have `float.MaxValue` distance. Since `float.MaxValue > 999999999` (the initial sentinel), `closestPlayer` stays at -1, which aborts the attack sequence.

**Symptoms**: Monster enters walk-toward-player, contacts player, then falls back to SmoothWalk and never attacks again.

**Fix in tests**: Continuously call `ClearPlayerInvulnerability(testPlayer)` inside WaitUntil loops alongside `ClearCommonAttackBlockers(monster)`. Alternatively, increase spawn distance so the walk step doesn't reach the player before the behavior is selected.

**Affected patterns**: Any monster that walks toward the player as part of its attack sequence (GrimRip walkAndAttack, WeedMommy charge, Demon chase-to-hit).

## Zero-Width Attack Distance Ranges

When a monster prefab has `attackDistanceMin == attackDistanceMax` (e.g., BigMagicDragon at 450/450), coroutine-based positioning cannot reliably trigger the attack behavior. The Unity execution order (Update runs behavior evaluation, then coroutine resumes) creates a timing gap where AnimatorMovement drift shifts the monster by sub-pixel amounts, causing the strict distance check to fail.

**Standard approach for zero-width ranges**: Use `FreezeMonsterBehavior`, set exact position, clear blockers, unfreeze, then immediately call `behaviourManager.Perform(interrupt: false, animationFinished: true)` in the same execution path. No yield between positioning and evaluation eliminates drift.

**Wide ranges (e.g., SmallMagicDragon 0-600)** can safely use the standard coroutine-loop pattern.

## Stationary Projectiles (speed=0) in Tests

Some monster projectiles have `my_Speed_multi: 0` (e.g., grimRipProjectile), meaning they are stationary and only damage on overlap. For these projectiles, the monster must be spawned very close to the player (e.g., 20 units) so the projectile spawns ON or overlapping the player. Standard spawn distances (100+ units) will never register a hit.

## Monster vs Player Projectile SpawnOffset Values

The spawnOffset values documented in "Projectile SpawnOffset and Test Distances" (meleePush=30, Fireball=50, magicMeleeProj=60, MagicianFire=120) are all **player** projectile offsets. Most **monster** projectiles (DemonAttackSmall, grimRipProjectile, dragonFire_multiTrail) have `spawnOffset: 0`. Always check the specific weapon/projectile prefab; do not assume monster projectiles share player projectile offsets.

## DeathMovement._state Persistence Across Tests

`DeathMovement._state` is NOT reset by setting `Player.IsGhost` or `Player.SetHP()`. If a player was killed in a previous test, `_state` remains `flying` or `findingBody`, causing `Player.isAlive()` to return false and all damage to be silently ignored. Use `RestorePlayerAlive(player)` (available on PlayModeTestBase) which properly resets death state, ghost mode, and HP via reflection.

## Common Gotchas

- `Framework.time` depends on Unity `Time.time` — blocks EditMode testing of timers/cooldowns
- `ReturnToCenter` (priority 140) overrides `AnimatorMovement` (priority 70) when monsters are far from map center
- `MonsterBehaviour.onNotSelected()` fires EVERY frame, not just on transition
- `MonsterBehaviourManager.Frozen` skips `Update()` behavior evaluation
- All 4 players have active AI — can interfere with isolated tests
- Camera border walls are physical BoxCollider2D — moving players to extreme positions shifts them (mitigated by automatic camera lock in PlayModeTestBase; only relevant if `AutoLockCamera` is overridden to `false`)
- `notSelectedCallback` fires every frame the behavior is NOT selected — use one-shot flags for transition logic
- PlayMode tests that trigger `SoundRegistry` log errors (e.g., "No clip found for (Demon, Walk)") fail as "Unhandled log message" unless `LogAssert.ignoreFailingMessages = true` is set at the start of the test. When diagnosing mass test failures, check if SoundRegistry missing-clip errors are the common cause before investigating individual test logic.
- **"Too many instant steps" / `ExitPlayModeTask` error**: Caused by stale `InitTestScene{guid}.unity` files in `Assets/`. Unity creates one per PlayMode test run; crashes leave them orphaned. Accumulation triggers `NullReferenceException` in `PlayModeRunTask`, which loops. Run `bash .claude/cleanup-init-scenes.sh` before test runs to prevent this. If the error appears, manually delete `Assets/InitTestScene*.unity` and their `.meta` files.
- **Background agents for workers**: Background agents isolate their file reads in their own context and let the Manager continue working. Use background by default. Note: background agents may not be able to write files if permissions aren't pre-approved — ensure `Company/pipelines/` has write permission in settings.
- **MCP is a single shared resource**: Unity Editor is one process. Never spawn two workers that use MCP tools in parallel (Tester, Implementer, Debugger all check compilation via MCP). Non-MCP workers (Documenter, Optimizer, Visionary) can run in parallel freely.
- **PlayMode fast-scan strategy**: When running many PlayMode test classes, run ALL tests in a single unfiltered batch first to get the total failure count. Then re-run only failing classes individually with batch filter for per-class diagnostic isolation. This is faster than running all 41+ classes one-by-one since most will pass.
- **MCP stale test job blocks all test runs**: The MCP test runner stores job state in Unity's `SessionState` (persists across domain reloads, survives editor restarts within the same session). A stuck `tests_running` state from a crashed or timed-out test run blocks ALL subsequent `run_tests` calls, returning `tests_running` immediately. Recovery: restart the MCP server via `Window > MCP for Unity` (stop/start). See Pre-Test Cleanup Protocol in "Compilation & MCP" section. Key indicator: `run_tests` returns `tests_running` but `editor_state` shows no actual tests executing.
- **Projectile spawnOffset and screen bounds**: See dedicated section "Projectile SpawnOffset and Test Distances" below for full details on spawnOffset overshoot, screen bounds destruction, and KillOtherPlayers camera narrowing.
- **Player.Fire() returns void, Weapon.Fire() returns bool**: `Player.Fire(false, weapon)` silently fails if the weapon is not ready. `weapon.Fire(...)` returns a bool indicating success. For tests that must confirm a weapon fired, use `weapon.Fire(...)` and assert the return value is true.
- **AI planner runs on a background thread**: `AIplanner.execute()` runs on `Task.Run`. Any property on `Weapon`, `Player`, or shared game objects that the AI planner reads must NOT call `Framework.time` (which calls `Time.get_time()`) unconditionally — this crashes with a Unity main-thread violation. Use short-circuit evaluation (e.g., `_flag && Framework.time < _deadline`) so the expensive call is only reached when the flag is true. This gotcha applies to any new state property added to `Weapon.cs` or `Player.cs` that the planner might query.

## Integration Test Isolation Checklist (PlayMode Monster Tests)

Monster behavior PlayMode tests require comprehensive isolation to prevent interference from background systems. Missing any of these steps causes flaky or consistently failing tests.

**In OnPerTestSetUp:**
1. **Suppress DynamicMonsterSpawner**: Call `LevelFlowManager.SetMode(GameFlowMode.Traversing)` to prevent background monster spawning. Restore to Arena in teardown.
2. **Stop AI weapon fire**: Set `OptionsData.StopAiMovement = true`. `KillOtherPlayers()` alone is NOT sufficient -- it hides players visually but the AI brain continues planning and firing weapons at monsters. Low-HP monsters (WeedMommy) can be destroyed within 23ms of spawn.
3. **Spawn far away**: Spawn test monsters 3000+ units from players to avoid init-time behavior callbacks (e.g., chase behaviors that set per-player cooldowns when a player is within chase range).
4. **Clear per-player cooldowns**: Some behavior callbacks set per-player attack cooldowns every frame during target selection. Use the appropriate `ForceStart*` method on the monster's TestContext (which calls `ClearCommonAttackBlockers` internally), or call `MonsterTestBase.ClearCommonAttackBlockers(monster)` directly. Clear continuously inside WaitUntil loops.
   - **Also clear player invulnerability** in WaitUntil loops: call `ClearPlayerInvulnerability(testPlayer)` alongside `ClearCommonAttackBlockers(monster)`. Contact damage during walk steps triggers invulnerability, which makes `closestPlayer=-1` (see "Contact Damage → Invulnerability → closestPlayer=-1 Cycle" section).
5. **Clear effects and projectiles**: In setUp, clean up lingering effects (set `timeRemaining = 0` + wait one frame for the effect manager to process, not just `destroy()`) and destroy lingering projectiles. `effect.destroy()` only removes visuals, not the effect from the manager list.
6. **Clear invulnerability**: Player invulnerability (0.5s) can silently block `player.hit()` calls between tests. Clear via reflection before kill attempts.
7. **LogAssert.ignoreFailingMessages = true**: Set at the START of each test method (SoundRegistry missing-clip errors).

**ForceStart behavior helpers (per-monster TestContext methods):**
Each monster type has a TestContext class in `Assets/Tests/PlayMode/Monsters/Contexts/` with a `ForceStart*` method that clears ALL blockers for triggering a specific behavior. These delegate common blockers to `MonsterTestBase.ClearCommonAttackBlockers(monster)` which clears: movement gate, per-player cooldowns (all 4 players), and weapon cooldowns.

| TestContext | Method | Monster-Specific Blockers |
|------------|--------|--------------------------|
| `WeedMommyTestContext` | `ForceStartChase(Player)` | Charge cooldown, chase trigger wiring |
| `DemonTestContext` | `ForceStartAttack()` | Post-hit cooldown |
| `MagicDragonTestContext` | `ForceStartRetaliation(Player)` | Retaliation state (active, end time, target index) |
| `GrimRipTestContext` | `ForceStartAttack()` | Frenzy state |

**Always use the ForceStart helper** instead of manually clearing blockers. Incomplete copies of blocker-clearing code that clear only a subset of blockers are the single most common test failure pattern.

**Contact test variant selection:**
- Small monster variants may have non-zero `ChargeStopDistance` that stops the charge before physical contact occurs. Use Big variants (ChargeStopDistance=0) for tests that require physical collider contact.

## Projectile SpawnOffset and Test Distances

Projectiles do NOT spawn at the weapon's origin. Each weapon/projectile prefab has a `spawnOffset` field that pushes the projectile forward from its origin point at spawn time. Tests that position entities at a specific distance must account for this offset.

**Known player spawnOffset values (check prefab for current values):**
- `meleePush` (player melee): spawnOffset=30
- `Fireball` (player range): spawnOffset=50
- `magicMeleeProj` (magic melee): spawnOffset=60
- `MagicianFire` (dragon fire, `dragonFire_multiTrail`): spawnOffset=120

**Monster projectile spawnOffset values are typically 0** — see "Monster vs Player Projectile SpawnOffset Values" section above.

**Rules for test distance calculations:**
- **Damage tests** (projectile must hit a target): `spawnDistance > spawnOffset` -- otherwise the projectile spawns PAST the target. E.g., DragonFire with spawnOffset=120 at spawnDistance=50 spawns 70 units PAST the player.
- **Collision tests** (two projectiles must meet mid-flight): `spawnDistance > playerSpawnOffset + monsterSpawnOffset` -- both projectiles must have room to travel. E.g., player range (offset=50) vs dragon fire (offset=120) needs spawnDistance > 170.
- **Screen bounds**: With camera locked to full map view (default in PlayModeTestBase), `MustBeOnScreen` destruction is no longer a concern — the entire map is visible. If a test overrides `AutoLockCamera => false`, projectiles with `MustBeOnScreen = OnScreen` are destroyed when outside the camera view — keep entities within ~250 units of the camera center in that case.

## FreezeMonsterBehavior Limitations

`FreezeMonsterBehavior(monster)` sets `MonsterBehaviourManager.Frozen = true`, which skips `Update()` behavior evaluation. However:
- **AnimatorMovement continues**: The animation system runs independently and sets Rigidbody velocity every frame, potentially moving the monster away from a test position.
- **Animation-complete callbacks bypass Frozen**: When an animation finishes, its completion callback calls `behaviourManager.Perform()` directly, bypassing the Frozen check. This can trigger a new behavior selection and animation even while frozen.
- **getHit() bypasses Frozen**: Calling `getHit()` on a monster runs a full behavior evaluation via `Perform()`, ignoring the Frozen state.
- **Mitigation**: Use `PinMonsterPosition(monster)` after `FreezeMonsterBehavior(monster)`. This attaches a `MonsterPositionPin` MonoBehaviour that zeros velocity and resets position every LateUpdate. Remove with `UnpinMonsterPosition(monster)`.

## Module Structure

~357 C# scripts across 12+ modules. Biggest: Common (~84), Waves (~40), Movements (~37), Monsters (~31), AI (29).

Key directories:
- `Assets/Scripts/` — all game logic
- `Assets/Monsters/` — monster prefabs
- `Assets/Attacks/` — attack/projectile prefabs
- `Assets/Tests/` — EditMode and PlayMode tests
- `MyLogs/TestLogs/` — test failure diagnostic logs

## KillOtherPlayers (Enhanced)

`PlayModeTestBase.KillOtherPlayers()` was enhanced in the `test-camera-infrastructure` pipeline (2026-03-29). It now provides full isolation:
- Sets `IsGhost = GhostMode.Nothing` (disables main collider)
- **Removes players from camera tracking** via `RemoveTargetTransform()` — prevents ghost players from influencing camera zoom
- **Disables ALL child colliders** via `GetComponentsInChildren<Collider2D>(true)` — eliminates deathCollider and weapon child colliders that previously intercepted projectiles
- Stores disabled colliders per-player for symmetric restoration in `ReviveOtherPlayers()`

`ReviveOtherPlayers()` reverses all three steps: re-registers with camera, re-enables only the colliders that were previously enabled, restores to spawn positions.

With the enhanced `KillOtherPlayers()` plus the automatic camera lock, manually repositioning other players for projectile path tests is no longer necessary.

## GrimRip PlayModeTestBase Sensitivity (RESOLVED)

GrimRip tests were sensitive to `KillOtherPlayers()` (not the base SetUp cleanup, as originally hypothesized). The root cause was a side effect in `GrimRipBase.isInCooldown()`.

**Root cause (fixed in forcestart-migration pipeline)**: `isInCooldown()` contained a side effect — when the cooldown state changed, it called `onNewCooldown()` which started `CoolDownWalkBehaviour`. This method was called by multiple condition callbacks during behavior evaluation. When only one player was attackable (KillOtherPlayers scenario), the interaction between multiple condition callbacks calling `isInCooldown()` created a state where `walkAndAttackBehaviour` could never be selected.

**Fix**: Separated the side effect into `updateCooldownTransition()`, called once per frame from `_updateMonster()` before behavior evaluation. `isInCooldown()` is now a pure query function with no side effects. Condition callbacks (`inCoolDownCallback`, `notInCoolDownCallback`, `chargeCallback`) call the pure `isInCooldown()` safely.

**Additional fix**: GrimRip spawn distance increased from 100 to 350 units. At close range, contact damage triggers player invulnerability (0.5s), which makes `GetClosestPlayer()` return -1 (no attackable player), which blocks `walkAndAttackBehaviour`. The 350-unit distance keeps the monster within `attackDistanceNormal` but avoids the contact damage cycle.

**All GrimRip tests now pass** without workarounds. The `yield break` override in `GrimRipBehaviorTests.OnPerTestSetUp` has been removed.
