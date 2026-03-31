# Pipeline: Monster Test Debug v2
**ID**: 2026-03-24-monster-test-debug-v2
**Status**: Running
**Type**: Debug
**Task**: Run all monster PlayMode tests, debug errors. Process monster-by-monster: Demon → GrimRip → MagicDragon → WeedMommy → Generic.

## Test Scope

### Demon (7 classes)
- DemonBehaviorTests
- DemonBlockTests
- DemonChaseFlowTests
- DemonChaseTests
- DemonHitResponseTests
- DemonLifecycleTests
- DemonPostHitDelayTests

### GrimRip (2 classes)
- GrimRipBehaviorTests
- SmallGrimRipTests

### MagicDragon (2 classes)
- BigMagicDragonTests
- SmallMagicDragonTests

### WeedMommy (11 classes)
- WeedMommyBehaviorTests, WeedMommyChargeCooldownTests, WeedMommyChargeTriggerTests
- WeedMommyChaseWalkTests, WeedMommyCooldownTests, WeedMommyDebuffTests
- WeedMommyGateChargeTests, WeedMommyHandsAttackTests, WeedMommyRayAttackTests
- WeedMommyTests, SmallWeedMommyTests

### Generic (6 classes)
- AnimationValidationPlayModeTests, MonsterAnimBugTests, MonsterAttackDamageTests
- MonsterSoundIsolationTest, MonsterSpawnSuppressionTests, MovementGatePlayModeTests

## Execution Log

### Phase 1: Run All Tests — Complete
- Total tests: ~500+ PlayMode
- Total failures: 44

#### Results by Monster
| Monster | Classes | Failures | Key Issues |
|---------|---------|----------|------------|
| Demon | 7 | 1 | Behavior cycle expectation (BigDemon retaliates immediately) |
| GrimRip | 2 | 1 | Charge timing (hit arrives after charge ends) |
| MagicDragon | 2 | 1 | Dragon stuck in chase, never attacks (15s timeout) |
| WeedMommy | 11 | 36 | **Systemic**: WeedMommy destroyed by AI within 23ms of spawn |
| Generic | 6 | 5 | Attack timeouts (3), MissingRef (1), chase not activating (1) |

#### Failure Clusters
**Cluster A — Monster destroyed by AI (24 failures)**: WeedMommy/BigWeedMommy/GrimRipBig destroyed during tests. AI players kill monster before test can run. Root cause: tests don't isolate monster from AI.
**Cluster B — Chase/behavior not activating (10 failures)**: Chase trigger conditions not met. Likely caused by Cluster A (dead monster can't chase) or test setup issues.
**Cluster C — Timing/behavior expectations (6 failures)**: Demon retaliation, GrimRip charge timing, BigMagicDragon stuck chase, MonsterAttackDamage timeouts.
**Cluster D — Damage/debuff assertions (4 failures)**: Charge contact debuff, ray mechanics. May overlap with Cluster A.

### Phase 1.5: Re-verification — Flaky Tests Eliminated
Re-ran failing classes individually to confirm:
- **GrimRipBehaviorTests.BlockedByCharge**: FLAKY — passed on rerun
- **MovementGatePlayModeTests (2 tests)**: FLAKY — passed on rerun
- **MonsterAttackDamageTests.GrimRipAttack**: FLAKY — passed on rerun

**Confirmed persistent failures: 40** (was 44)
- DemonChaseFlowTests: 1 → Fixed by Debugger #1
- BigMagicDragonTests: 3 (confirmed, up from 1)
- MonsterAttackDamageTests.DragonFire: 1 (confirmed)
- WeedMommy: 36 (systemic)

### Phase 2: Debug — In Progress
- **Debugger #1 (Demon)**: Fixed DemonChaseFlowTests.FullCycle — verified passing
- **Debugger #2 (WeedMommy)**: Fixing 36 systemic failures — monster destroyed by AI, adding KillOtherPlayers to setUp
- **Debugger #3 (BigMagicDragon + DragonFire)**: Fixing 3 BigMagicDragonTests + 1 DragonFire failure — dragon stuck in chase / projectile timeout

## Optimizer Findings

### Pipeline Health: Fair
- Phases completed without debug: 1/2 (Phase 1 ran clean, Phase 2 required 15 Debugger dispatches)
- Debug iterations needed: 15 (journals 01-05, 07-19)
- Total roles dispatched: 17 (15 Debugger, 1 Documenter, 1 Visionary)

### Role Performance
| Role | Grade | Notes |
|------|-------|-------|
| Tester | D | Caused 90%+ of failures: hardcoded values, incomplete helper copies, wrong variants, missing isolation |
| Debugger | B+ | Correct diagnoses, good journal entries, but inconsistent cross-file application of fixes |
| Documenter | A | WeedMommy tech doc was referenced by multiple subsequent Debuggers |
| Visionary | A | Prefab value audit was thorough and actionable |
| Manager | B | Good monster-by-monster scoping, but too many sequential dispatches for the same root cause |

### Root Cause Analysis

**DemonChaseFlowTests (1 failure)**: Stale test. Player positioned at exactly attackDistanceClose (300, from prefab). Tester used code default not prefab value. -- Debugger fixed correctly in 1 iteration.

**BigMagicDragonTests (3 failures)**: Stale test + test setup bug. Hardcoded distance 300/500 but prefab attackDistanceMin=attackDistanceMax=450 (zero-width window). AnimatorMovement drift during Frozen. -- Required 2 Debugger iterations (03 missed drift, 05 fixed with DriveToWalkAndAttack).

**MonsterAttackDamageTests DragonFire (1 failure)**: Code bug (DynamicCollider.cs) + stale test (spawn distance). DynamicCollider overwrote projectile rotation every frame. Only genuine production bug in the pipeline. -- Debugger found via careful code tracing.

**WeedMommy systemic (36 failures)**: Test setup bug. KillOtherPlayers insufficient for AI isolation; needed OptionsData.StopAiMovement. -- Debugger fixed in 1 iteration but discovered deeper issues in subsequent iterations.

**WeedMommy behavior/charge/contact (remaining ~20 failures)**: Test setup bug. ForceStartChase cleared 1 of 3 blockers. DynamicMonsterSpawner not suppressed. Wrong variant for contact tests. Inter-test state contamination. -- Required 12 Debugger iterations across journals 07-19.

### Role File Changes Applied
- roles/tester.md: Added "Integration Test Setup Rules" section (5 rules) -- Tester had zero guidance on integration test setup
- roles/debugger.md: Added Rule 11 "Apply fixes consistently across the module" -- Debuggers fixed one file, left identical bugs in siblings
- roles/debugger.md: Added Rule 12 "Clean up diagnostic artifacts" -- Debug logs left behind for others to clean up
- project-guidelines.md: Added "Integration Test Isolation Checklist" (7-step, project-specific) -- WeedMommy isolation pattern
- project-guidelines.md: Added "FreezeMonsterBehavior Limitations" -- AnimatorMovement drift gotcha
- company-execute/SKILL.md: Added batch-fix guidance for repeated root cause patterns

### Proposed Changes (PENDING USER REVIEW)
- Extract shared WeedMommy test base class with ForceStartChase (3-blocker), SuppressAttackBehaviour, MoveMonsterForLongChase, PrepareForHandsAttack, and DynamicMonsterSpawner suppression. Currently duplicated across 5+ files with varying completeness. Would eliminate the dominant failure class.

### Manager Autonomy Audit
- No unnecessary questions asked during this pipeline (Debug type, scope was clear)
- Good: Manager stated scope assumptions in the pipeline state file without asking

### Pipeline Design Issues
- **Sequential dispatches for same root cause**: The ForceStartChase 3-blocker pattern was identified in journal 07 but not propagated to all files until journals 10-17 (7 more dispatches). Once a systemic pattern is found, batch all remaining affected files into one dispatch. Added guidance to company-execute/SKILL.md.
- **No shared test helper infrastructure**: The ForceStartChase helper was duplicated-with-divergence across 5 test files. A shared base class would prevent this entirely. Flagged as proposed change for user review.
