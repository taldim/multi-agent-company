# Sub-Manager Journal: SP-C Melee Commitment + Hit-Confirm Cancel

## Status: READY_FOR_TESTING

## What Was Done

### Phase 1: Documentation
- Updated `Documentation/GDD/GDD_AttackTimeWindows.md` — added "Melee Commitment" section covering rules, hit-confirm cancel, miss behavior, dormant-by-default design
- Updated `Documentation/Technical/AttackTimeWindows_Technical.md` — added full technical specification for MeleeCommitment static class, Weapon.cs fields, WeaponMeleePlayer.cs fields, Player integration, castableMovement integration, flow diagram, testable behaviors table
- Updated `Documentation/Testing/Tests_AttackTimeWindows.md` — added MeleeCommitmentTests.cs spec with 13 test cases across 5 categories

### Phase 2: Tests
- Created `Assets/Tests/EditMode/Skills/MeleeCommitmentTests.cs` — 13 EditMode tests for pure static MeleeCommitment logic + config validation

### Phase 3: Implementation
- Created `Assets/Scripts/Skills/MeleeCommitment.cs` — pure static class with 4 methods: ShouldActivateCommitment, IsCommitmentActive, ShouldBlockFire, ShouldBlockCastableMovement
- Modified `Assets/Scripts/Skills/Weapon.cs` — added `_isCommitted` and `_commitmentEndTime` fields, `IsCommitted` property (uses MeleeCommitment.IsCommitmentActive), `StartCommitment()` and `EndCommitment()` methods, auto-expiry in Update, clear on finishFire
- Modified `Assets/Scripts/Skills/WeaponMeleePlayer.cs` — added `meleeCommitmentDuration` serialized field (default 0 = dormant), `MeleeCommitmentDuration` test accessor, `OnMeleeHitConfirm()` method
- Modified `Assets/Scripts/Skills/meleeMagic.cs` — FirePrimary() now wires hit callback for commitment release (combined with combo callback via multicast delegate), calls StartCommitment after fire
- Modified `Assets/Scripts/Player/Player.cs` — `_canFireNow()` now checks melee weapon's IsCommitted to block all weapon fire during commitment
- Modified `Assets/Scripts/Movements/castableMovement.cs` — `canStartOther()` now checks player's melee weapon commitment to block counter/teleport

## Workers Dispatched

None — I wrote all code directly.

### Decision: Write Directly vs Spawn Workers
The files are tightly coupled (6 files must be self-consistent for compilation), the pure static class follows an established pattern (MeleeCombo.cs), and the tests follow existing patterns (WeaponFireDelayTests.cs, MeleeComboTests.cs). Spawning workers would add overhead without value — inconsistency between files would cause compilation failures.

## Decisions Made
- **Hit callback wiring**: Used multicast delegate (`System.Action hitCallback = null; hitCallback += OnComboHitConnected; hitCallback += OnMeleeHitConfirm`) to combine combo tracking and commitment release into a single callback. This preserves the existing combo system's behavior exactly while adding commitment release.
- **Finisher commitment**: Commitment applies to finisher hits too (not just jabs). Even a finisher can miss. The hit callback for commitment release is always wired when commitment is active. The combo callback is only wired for non-finisher jabs (existing behavior preserved).
- **Commitment expiry in Update**: Auto-expires in `_MagiciansObject_Update()` by checking `MeleeCommitment.IsCommitmentActive()`. This ensures the `_isCommitted` field gets cleaned up even if the projectile is destroyed without hitting anything.
- **Clear on finishFire**: `finishFire()` clears `_isCommitted = false` to ensure clean state during test setUp weapon resets.
- **Boundary behavior**: Commitment end time uses strict less-than (`currentTime < commitmentEndTime`), so at exact end time the commitment is expired. Consistent with the combo timeout boundary behavior.

## Shared Files Modified
- `Weapon.cs` — SP-D (Input Buffer) had already added `inputBufferWindow`, `_bufferedFirePending`, `_bufferedFireSpecial`, `_bufferedFireTime`, `IsBufferedFirePending`, `InputBufferWindow`, and input buffer logic in Update. My changes (commitment state, methods) are additive and do not conflict.
- `castableMovement.cs` — SP-D had already added `_bufferedMovementPending`, `_bufferedDirection`, `_bufferedMovementTime`. My change to `canStartOther()` is a simple additive check.
- `Player.cs` — No conflicts detected from other sub-plans.

## Files Modified
- `Assets/Scripts/Skills/MeleeCommitment.cs` (NEW) — 53 lines
- `Assets/Scripts/Skills/Weapon.cs` (MODIFIED) — added ~40 lines
- `Assets/Scripts/Skills/WeaponMeleePlayer.cs` (MODIFIED) — added ~15 lines
- `Assets/Scripts/Skills/meleeMagic.cs` (MODIFIED) — changed ~15 lines
- `Assets/Scripts/Player/Player.cs` (MODIFIED) — changed 5 lines
- `Assets/Scripts/Movements/castableMovement.cs` (MODIFIED) — added 4 lines
- `Assets/Tests/EditMode/Skills/MeleeCommitmentTests.cs` (NEW) — 155 lines
- `Documentation/GDD/GDD_AttackTimeWindows.md` (MODIFIED)
- `Documentation/Technical/AttackTimeWindows_Technical.md` (MODIFIED)
- `Documentation/Testing/Tests_AttackTimeWindows.md` (MODIFIED)

## Test Scope
- EditMode: `MeleeCommitmentTests` (13 tests)
- PlayMode: None written (PlayMode tests would require MeleeCommitmentPlayTests — future work after commitment duration is configured > 0 on a prefab)

## Issues / Risks
- **dotnet build cannot verify**: The `.csproj` is auto-generated by Unity and won't include new files until Unity regenerates it. Code correctness was verified by manual review against compilation rules.
- **System is dormant by default**: `meleeCommitmentDuration = 0` means no behavioral change. Tests validate the pure logic only. PlayMode integration tests require a non-zero duration to be set on the weapon prefab — this is a designer tuning step.
- **SP-D parallel changes**: Input Buffer sub-plan (SP-D) modified several of the same files. I preserved all SP-D additions. If SP-D's changes are reverted or modified, my code still works independently (no dependency on input buffer).
- **No .meta file for MeleeCommitment.cs**: Unity will auto-generate the `.meta` file when it detects the new file. Same for `MeleeCommitmentTests.cs`.
