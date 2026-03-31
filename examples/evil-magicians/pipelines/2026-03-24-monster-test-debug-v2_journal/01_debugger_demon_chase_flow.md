# Journal: Debugger -- DemonChaseFlowTests.FullCycle_SmoothToChaseToHitToChase

## What I Did

- Diagnosed failure in `DemonChaseFlowTests.FullCycle_SmoothToChaseToHitToChase`
- Root cause: test placed player at 300 units from BigDemon, which is exactly at `attackDistanceClose` (300, from prefab). BigDemon has no post-hit delay (`IsBigMonster=true`), so after GetHit, AttackBehaviour fired immediately since player was within attack range.
- Fix: moved player from 300 to 400 units from demon -- outside attack range (300) but inside chase range (500). This prevents immediate retaliation while keeping the player in chase range for ChaseWalkBehaviour to activate.
- Updated comments to accurately describe BigDemon behavior (no post-hit delay, unlike Small Demon).
- Verified: all 3 DemonChaseFlowTests pass (3/3, 0 failures).

## Root Cause Analysis

**Classification**: Stale test -- the test was written with `300` as the player distance, which happened to be exactly the BigDemon's `attackDistanceClose` value from the prefab. The test's Phase 4 assumed ChaseWalk would resume after GetHit, but BigDemon can retaliate immediately (no `postHitAttackDelay` condition callback for `IsBigMonster=true` -- see `DemonBase.cs` lines 274-281). The test's own sibling test `PostHitDelay_FullGameFlow` (line 152-153) documents this: "Post-hit delay is Small Demon only -- BigDemon can retaliate immediately."

**Why the test was wrong**: The original test used `Vector3.left * 300` for player positioning. The code default for `attackDistanceClose` is 100 (DemonBase.cs line 26), but the BigDemon prefab overrides it to 300 (BigDemon.prefab, serialized field). The test author likely used the code default, not the prefab value.

**Diagnostic timeline from log**:
1. `21:32:21.036` -- GetHit animation completes
2. `21:32:21.045` -- ChaseWalkBehaviour selected (correct, briefly)
3. `21:32:21.078` -- AttackBehaviour fires (33ms later, immediate retaliation)
4. Test assertion fails: expected ChaseWalkBehaviour, got AttackBehaviour

## Files Modified

- `Assets/Tests/PlayMode/Monsters/DemonChaseFlowTests.cs` -- Changed player distance from 300 to 400 in Phase 2 and Phase 4. Updated comments.

**Production assets modified**: None

## Debug Iteration 1

**Error**: Phase 4 expects "ChaseWalkBehaviour" but got "AttackBehaviour"
**Test-Doc Alignment**: Mismatch -- test doc says "Chase resumes after getHit animation + postHit delay" but BigDemon has no postHit delay. The test positioned the player at exactly the attack distance boundary.
**Root Cause**: Stale test -- player distance (300) matched BigDemon prefab attackDistanceClose (300), enabling immediate retaliation after GetHit.
**Fix**: Changed player distance from 300 to 400 (outside attack range 300, inside chase range 500). Updated comments.
**Files Modified**: Assets/Tests/PlayMode/Monsters/DemonChaseFlowTests.cs
**Production assets modified**: None
**Result**: Fixed -- all 3 DemonChaseFlowTests pass (3/3)

## Notes for Optimizer

- The test batch filter file (test_batch.txt) was repeatedly overwritten by another concurrent agent during debugging, causing significant delays. Consider a locking mechanism or per-agent batch filter files.
- The MCP test runner stuck job state is difficult to clear from within an agent. The ClearStuckJob() method exists but is internal. I had to create a temporary editor script to clear SessionState keys directly. Consider adding clear_stuck as a documented first-class parameter in the MCP run_tests tool schema (it exists in the code at line 23 of RunTests.cs but is not in the schema).
- When tests reference distances, the Tester role should always check the prefab serialized field values, not the code defaults. Code defaults are often overridden by prefab values.
