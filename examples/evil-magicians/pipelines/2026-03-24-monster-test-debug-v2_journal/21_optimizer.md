# Journal: Optimizer — Post-Pipeline Analysis

## What I Reviewed
- Pipeline state file: `.claude/company/pipelines/2026-03-24-monster-test-debug-v2.md`
- All 19 journal entries (01-19) from 5 Debugger dispatches, 1 Visionary, 1 Documenter
- All 7 role files (debugger, tester, implementer, documenter, visionary, task-manager, optimizer)
- Project guidelines, learnings file, both SKILL files (company-plan, company-execute)

## Key Observations

### Pipeline Health: Fair
- 44 initial failures reduced to 10 remaining (77% fix rate)
- 1 genuine production bug found (DynamicCollider.cs rotation)
- 13 test files modified across Demon, GrimRip, BigMagicDragon, WeedMommy
- ~15 Debugger dispatches needed — high iteration count driven by WeedMommy's complex behavior system

### Dominant Failure Pattern: Incomplete Test Setup (Tester role)
The Tester role caused 90%+ of all failures. The same 4 setup mistakes appeared across 10+ test files:

1. **ForceStartChase 3-blocker pattern** — The helper was duplicated across 5 WeedMommy test files with varying completeness. Some versions cleared 1 blocker, others 2, only one cleared all 3. This single pattern caused failures in journals 07, 08, 10, 12, 13, 14, 15, 16, 17, 18.

2. **DynamicMonsterSpawner suppression** — Missing from most WeedMommy test setUp methods. Every test class needed `LevelFlowManager.SetMode(Traversing)` but only WeedMommyDebuffTests had it initially.

3. **Hardcoded prefab values** — Tests used literal numbers (300, 450, 100) for distances/cooldowns instead of reading runtime values. When designers tuned prefabs, tests broke silently.

4. **Wrong monster variant for contact tests** — Small variants with non-zero ChargeStopDistance were used for tests requiring physical contact, which is impossible with that configuration.

### Secondary Pattern: Inter-Test State Contamination
Journal entries 18 and 19 revealed that teardown gaps allow state to leak:
- `OptionsData.StopAiMovement = false` in teardown gives AI one frame to fire weapons
- Player invulnerability persists across tests (0.5s timer)
- Effect cleanup via `destroy()` only removes visuals, not the effect from the manager's list
- Projectiles from AI fire during teardown persist into the next test

### Production Bug (DynamicCollider.cs)
Journal 04 found a real production bug: `DynamicCollider.Apply()` overwrites the projectile's firing-angle rotation every frame with raw keyframe rotation (always 0). Fix: capture `baseRotation` in `Start()` and add as offset in `Apply()`. This was the only code bug — everything else was test setup.

### Debugger Performance
- Debugger #1 (Demon, journal 01): Excellent. Clean diagnosis, minimal fix, no regressions.
- Debugger #2 (WeedMommy systemic, journal 02): Good. Correctly identified KillOtherPlayers insufficiency.
- Debugger #3 (BigMagicDragon, journals 03-05): Two iterations needed. First attempt (03) missed AnimatorMovement drift. Second attempt (05) nailed it with DriveToWalkAndAttack coroutine.
- Debugger #4 (DynamicCollider, journal 04): Excellent. Found genuine production bug through careful code tracing.
- Debuggers #5-15 (WeedMommy individual, journals 07-19): Mixed. Correctly identified root causes but created a pattern where each Debugger fixed its assigned tests without checking consistency with other test files in the same module. Journal 18 explicitly documents regressions caused by this.

### Documenter Performance
- Journal 09: Good output. Created WeedMommy_ChaseCharge_Technical.md that subsequent Debuggers referenced. Correctly identified gotchas (Section 8) that matched actual failure patterns.

### Visionary Performance
- Journal 06: Good strategic audit. Identified 40-50 hardcoded values across 15 files. Created actionable plan with prioritized task list and model pattern (WeedMommyBehaviorTests as gold standard).

### Manager Performance
- Good: Monster-by-monster processing prevented cross-contamination
- Good: Re-verified full scope after each Debugger
- Could improve: Many sequential Debugger dispatches for the same root cause (ForceStartChase 3-blocker) — once the pattern was found in journal 07, all remaining WeedMommy files should have been fixed in one batch

## Role File Changes Applied

### tester.md — Added "Integration Test Setup Rules" section
5 new rules addressing the dominant failure pattern:
1. Never hardcode configurable values — read at runtime
2. Reuse existing test helpers — don't write incomplete copies
3. Verify setup preconditions match runtime state — animation drift, floating-point, inter-test state
4. Choose the right test subject variant — verify the variant supports the behavior being tested
5. Isolate from background systems — suppress AI, spawners, verify kill/hide is sufficient

### debugger.md — Added Rule 11: "Apply fixes consistently across the module"
When fixing a test class, scan all other test classes in the same module for the same vulnerability. If ForceStartChase needs 3 blockers in one file, it needs 3 blockers in every file that uses it. Do not fix one file and leave identical bugs in sibling files.

### debugger.md — Added Rule 12: "Clean up diagnostic artifacts"
Remove debug logs, temporary helpers, and diagnostic code added during debugging before reporting the fix as complete. Do not leave `[TEST_DEBUG]` or `[DIAG]` logs for future Debuggers to clean up.

### project-guidelines.md — Added "Integration Test Isolation Checklist"
Project-specific checklist for PlayMode monster tests:
- DynamicMonsterSpawner suppression (LevelFlowManager.SetMode Traversing)
- AI weapon suppression (OptionsData.StopAiMovement = true)
- Spawn distance (3000+ units to avoid init-time behavior callbacks)
- Per-player cooldown clearing
- Effect cleanup (timeRemaining=0 + frame wait, not just destroy())
- Projectile cleanup in setUp

### project-guidelines.md — Added "FreezeMonsterBehavior limitations"
Documented that AnimatorMovement continues during Frozen, and animation-complete callbacks bypass the Frozen check, running a full behavior evaluation.

### project-guidelines.md — Added "KillOtherPlayers is insufficient"
Documented that KillOtherPlayers only hides players visually but AI brain continues planning and firing. Must also set OptionsData.StopAiMovement = true.

### company-execute/SKILL.md — Added guidance for repeated root cause patterns
When the same root cause appears in multiple Debugger reports, batch remaining instances together in a single Debugger dispatch instead of sequential one-at-a-time fixes.

## Proposed Changes (PENDING USER REVIEW)

### Shared WeedMommy Test Base Class
**File**: project-guidelines.md (recommendation, not direct edit)
**Proposed change**: Extract a `WeedMommyTestBase` class with the shared ForceStartChase (3-blocker), SuppressAttackBehaviour, MoveMonsterForLongChase, PrepareForHandsAttack, and DynamicMonsterSpawner suppression. Currently these are duplicated across 5+ test files with varying completeness.
**Reason**: The ForceStartChase duplication-with-divergence pattern caused 15+ test failures. A shared base would eliminate this entire failure class.
**Status**: Pending

## Notes for Optimizer (self)
- This pipeline had unusually high Debugger iteration count (15 dispatches for 40 failures). The root cause pattern (ForceStartChase 3 blockers) was identified in journal 07 but not propagated to all files until journals 10-17. Future guidance: when a Debugger identifies a systemic pattern, the Manager should batch-fix all affected files in one dispatch.
- The Tester role file had no guidance on integration test setup. The new "Integration Test Setup Rules" section addresses this gap directly. Monitor whether the next pipeline with PlayMode tests shows improvement.
