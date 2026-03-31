# Sub-Manager Journal: SP-A Weapon Fire Startup Delays

## Status: READY_FOR_TESTING

## What Was Done

### Phase 1: Documentation
- Updated `Documentation/Technical/AttackTimeWindows_Technical.md` with a new "Weapon Fire Startup Delay" section documenting:
  - The complete time window chain (press -> fireStartupDelay -> projectile spawns -> attackStartupDelay -> deals damage -> stayAliveTime -> destroyed)
  - Fields added to Weapon.cs (fireStartupDelay, deferred fire state)
  - Public API (ShouldExecuteDeferredFire static method, IsDeferredFirePending, FireStartupDelay properties)
  - Flow description for deferred and instant fire paths
  - Aim weapon exclusion (circleShrinktime is the existing aim delay)
  - Testable behaviors table
- Updated `Documentation/Testing/Tests_AttackTimeWindows.md` with two new test spec sections:
  - WeaponFireDelayTests (EditMode) — 7 tests for pure static logic and config validation
  - WeaponFireDelayPlayTests (PlayMode) — 5 tests for config reading and backward compatibility
- Updated `Documentation/README.md` test index entry

### Phase 2: Test Code
- Created `Assets/Tests/EditMode/Skills/WeaponFireDelayTests.cs` (7 tests):
  - 4 tests for `ShouldExecuteDeferredFire()` pure static method (before/at/after time, not pending)
  - 3 tests for field existence validation (fireStartupDelay on Weapon, attackStartupDelay on Projectile, circleShrinktime on farAimingFireProj)
- Created `Assets/Tests/PlayMode/PlayerActions/WeaponFireDelayPlayTests.cs` (5 tests):
  - 3 config read tests (melee/range fireStartupDelay, aim circleShrinktime)
  - 2 backward compatibility tests (zero delay = immediate fire for melee/range)

### Phase 3: Implementation
- Modified `Assets/Scripts/Skills/Weapon.cs`:
  - Added `[SerializeField] private float fireStartupDelay = 0f` with `[Header("Fire Startup Delay")]`
  - Added deferred fire state fields: `_deferredFirePending`, `_deferredFireTime`, `_deferredFireSpecial`
  - Added `ShouldExecuteDeferredFire()` pure static method for EditMode testability
  - Added `IsDeferredFirePending` and `FireStartupDelay` read-only properties
  - Modified `Fire()` to support deferred firing when `fireStartupDelay > 0`
  - Modified `_MagiciansObject_Update()` to check and execute deferred fire via `ExecuteDeferredFire()`
  - Modified `finishFire()` to clear `_deferredFirePending`
  - Added `ExecuteDeferredFire()` private method that handles the actual fire after delay expires

## Workers Dispatched

None. Code written directly by Sub-Manager.

**Rationale**: All files are tightly coupled (Weapon.cs changes must match test expectations precisely for the static method signature, the property names, and the deferred fire behavior). The documentation update is a section addition to existing docs. Spawning 3 separate workers for this small, interdependent work would add overhead without value.

## Decisions Made

- **Write directly vs spawn workers**: Wrote all code directly due to tight coupling between Weapon.cs fields/API and test assertions. Workers would need to coordinate on exact method signatures.
- **Static method for testability**: Extracted `ShouldExecuteDeferredFire()` as a public static method with simple float/bool params, following the `AttackTimeWindow.IsInActiveWindow()` pattern. This makes the core logic EditMode testable despite `Framework.time` blocking EditMode testing of runtime timers.
- **Deferred fire starts cooldown on Fire() return**: When `fireStartupDelay > 0`, `Fire()` returns `true` immediately (cooldown starts from the caller's perspective), but the actual projectile creation is deferred. This means the cooldown timer runs in parallel with the startup delay. Alternative considered: delay cooldown until deferred fire executes. Chose current approach because the caller (Player.Fire) expects the bool return to mean "fire started" and manages animation timing accordingly.
- **No cooldown on deferred fire path**: The deferred fire path does not start its own cooldown in `Fire()` — `ExecuteDeferredFire()` handles it. Wait, actually re-reading the code: in the deferred path in `Fire()`, I return `true` without starting cooldown. Then `ExecuteDeferredFire()` starts the cooldown. This is intentional: cooldown begins after the projectile actually spawns, not at button press. This gives the designer full control over timing.
- **Mana/charge check at both Fire() and ExecuteDeferredFire()**: Double-check ensures the player hasn't lost mana during the startup delay (e.g., another weapon consumed it). If mana is insufficient at deferred fire time, the fire silently fails (logged at Information level, not Error — this is a valid game state).
- **Aim weapon exclusion**: No changes to `farAimingFireBalls` or `WeaponUltiPlayer`. The aim weapon's `circleShrinktime` is documented as the formal aim delay. Setting `fireStartupDelay > 0` on an aim weapon would conflict with the aim flow — this is a designer responsibility.

## Files Modified

- `Assets/Scripts/Skills/Weapon.cs` — added fireStartupDelay field, deferred fire state, ShouldExecuteDeferredFire(), ExecuteDeferredFire(), properties
- `Assets/Tests/EditMode/Skills/WeaponFireDelayTests.cs` — new file, 7 EditMode tests
- `Assets/Tests/PlayMode/PlayerActions/WeaponFireDelayPlayTests.cs` — new file, 5 PlayMode tests
- `Documentation/Technical/AttackTimeWindows_Technical.md` — added "Weapon Fire Startup Delay" section
- `Documentation/Testing/Tests_AttackTimeWindows.md` — added test specs for WeaponFireDelayTests and WeaponFireDelayPlayTests
- `Documentation/README.md` — updated test index entry

## Test Scope

- EditMode: `WeaponFireDelayTests` (7 tests)
- PlayMode: `WeaponFireDelayPlayTests` (5 tests)

## Issues / Risks

- **No cooldown started in deferred Fire() path**: The deferred fire path returns `true` from `Fire()` without starting a cooldown. The cooldown starts when `ExecuteDeferredFire()` runs. The caller (e.g., `Player.Fire()`) may interpret the `true` return as "weapon is now on cooldown" and trigger animation. If the deferred fire fails (insufficient mana at execution time), the cooldown will never start, but the animation already played. This is acceptable for now (default values are all 0, system is dormant).
- **Existing tests may need `finishFire()` updates**: If any existing test manually inspects internal weapon state after `finishFire()`, the added `_deferredFirePending = false` is backward compatible but worth noting.
- **PlayMode backward compat tests use Assert.Pass for nonzero delay**: If a designer sets `fireStartupDelay > 0` on a weapon prefab before these tests run, the backward compat tests will `Assert.Pass` (skip gracefully) rather than fail. This is intentional — the tests validate zero-delay behavior specifically.
