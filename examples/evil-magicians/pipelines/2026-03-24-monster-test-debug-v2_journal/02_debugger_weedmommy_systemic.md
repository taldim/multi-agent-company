# Journal: Debugger -- WeedMommy Systemic Test Failures (36 failures across 9 classes)

## What I Did
- Diagnosed the root cause of 36 WeedMommy test failures across 9 test classes
- Added `OptionsData.StopAiMovement = true;` to `OnPerTestSetUp()` in 8 WeedMommy test classes
- `WeedMommyDebuffTests` already had it; no change needed there

## Root Cause Analysis

**Surface symptom**: WeedMommy (SmallMonster, very low HP) destroyed within ~23ms of spawning, before test logic can execute. MissingReferenceException or assertion failures on a dead monster.

**Deeper cause**: `KillOtherPlayers()` was already present in all 9 failing classes but is INSUFFICIENT for AI isolation. It sets `GhostMode.Nothing` (invisible, collider off) and stops weapons once, but does NOT prevent the AI brain from planning and firing weapons on subsequent frames. The AI system (`Brain.cs`, `BrainForcer.cs`) has no awareness of ghost mode.

**Root cause**: `OptionsData.StopAiMovement` is the global flag that controls whether AI brains can plan and fire. `PlayModeTestBase.FreezeAllAIPlayers()` sets it to `true` during the one-time scene load. However, `WeedMommyDebuffTests.OnPerTestTearDown()` sets it to `false`, and AI decision test classes (`AIMeleeDecisionTests`, `AIDecisionRateTests`) also set it to `false` within their tests. When test execution order causes these classes to run before other WeedMommy classes, the global AI freeze is lost. The WeedMommy classes relied solely on `KillOtherPlayers()`, which only hides players visually but leaves their AI brain actively firing weapons at monsters.

**Evidence chain**:
1. Failure logs show all 4 players alive at full HP at failure time -- `KillOtherPlayers()` doesn't reduce HP
2. "Player Fire Failed" messages in logs confirm AI is actively trying to fire
3. `destroyObject: WeedMommy(Clone)` within 23ms of spawn
4. `ActiveMonsterCount = 0` at failure time
5. `BrainForcerTutorial.canPlan()` returns `!OptionsData.StopAiMovement` -- when flag is false, AI is active

**Which role caused the failure**: Tester -- the original WeedMommy tests used `KillOtherPlayers()` without `OptionsData.StopAiMovement = true`, which is insufficient for preventing AI weapon fire. This is a test setup bug, not a code bug.

## Fix Applied

Added `OptionsData.StopAiMovement = true;` as the first line of `OnPerTestSetUp()` in each of the 8 classes that were missing it. This ensures AI brains are frozen regardless of test execution order, preventing players from firing weapons at the spawned WeedMommy.

This is safe because:
- Monster behavior runs via `MonsterBehaviourManager`, completely independent of player AI
- All player weapon firing in these tests is done explicitly via test helpers (`HitWeedMommy()`, `ForceStartChase()`)
- No WeedMommy test relies on natural player AI behavior

## Files Modified (test-only, no production code)
- `Assets/Tests/PlayMode/Monsters/WeedMommyBehaviorTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyChargeTriggerTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyChaseWalkTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyChargeCooldownTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyRayAttackTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyCooldownTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyGateChargeTests.cs` -- added `OptionsData.StopAiMovement = true;`
- `Assets/Tests/PlayMode/Monsters/WeedMommyHandsAttackTests.cs` -- added `OptionsData.StopAiMovement = true;`

**Production assets modified**: None

## Notes for Optimizer
- `KillOtherPlayers()` alone is NOT sufficient for AI isolation. Any test that spawns fragile monsters (low HP) must ALSO set `OptionsData.StopAiMovement = true` to prevent AI weapon fire.
- Consider adding `OptionsData.StopAiMovement = true;` to the base `PlayModeTestBase.SetUp()` (not just `LoadGameScene`) so it is re-asserted before EVERY test. Tests that need AI active can opt-in by setting it to false. This would prevent this class of bug from recurring.
- The `WeedMommyDebuffTests` teardown setting `StopAiMovement = false` is the trigger for other classes' failures. While each class now protects itself, the underlying pattern of test classes modifying global state in teardown and not restoring to the base default is fragile.
- The CLAUDE.md memory should be updated to note: "KillOtherPlayers() prevents collisions but not AI weapon fire. Use OptionsData.StopAiMovement = true for full AI isolation."
