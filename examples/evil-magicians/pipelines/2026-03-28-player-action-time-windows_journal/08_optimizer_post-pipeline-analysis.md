# Journal: Optimizer — Post-Pipeline Analysis

## What I Did

Analyzed all 6 Sub-Manager journal entries, the pipeline state file, and the execution log. Traced 3 issues to their root causes and applied improvements to 4 files.

## Issues Analyzed

### 1. Thread-Safety Bug (SP-C) — Most Significant
`Weapon.IsCommitted` property called `Framework.time` (→ `Time.get_time()`) unconditionally. The AI planner reads weapon properties from a background thread, causing a Unity main-thread violation crash. This is the only issue that required a code fix during full regression.

**Root cause chain**: SP-C Sub-Manager extracted pure static `MeleeCommitment.IsCommitmentActive()` correctly, but the `Weapon.IsCommitted` property that calls it passes `Framework.time` as an argument — evaluated before the static method can short-circuit. The fix was to add `_isCommitted &&` before the call so `Framework.time` is never reached when the system is dormant.

**Why it escaped scoped tests**: SP-C only wrote EditMode tests for pure static logic. The thread-safety issue requires PlayMode with 4 active AI players all querying `IsCommitted` from their background planning threads. No scoped test could catch this.

**Preventive measures applied**: Sub-Manager Rule 10 (thread safety), Implementer "Thread Safety for Shared Properties" subsection, project-guidelines Common Gotchas entry for AI background thread.

### 2. Non-Existent Enum Value (SP-F) — Trivial
Used `MagiciansLog.logType.common` which doesn't exist. Should be `.players`.

**Root cause**: SP-F didn't read the `MagiciansLog` source to verify enum values. A simple compilation check would have caught this, but SP-F couldn't verify compilation (MCP session issue).

**Preventive measure**: Sub-Manager Rule 9 (verify enum/API names against actual code).

### 3. Assertion Contradicting Design (SP-A) — Trivial
`AimWeapon_CircleShrinktime_ReadFromPrefab` asserted `> 0` for a field whose design default is 0.

**Root cause**: SP-A wrote the test assuming a non-zero value without checking the prefab or re-reading the blueprint's "all defaults are 0" design decision.

**Preventive measure**: Tester Rule 13 (assertions must align with stated design).

## What Went Well

1. **Zero Debugger dispatches within sub-plans** — all 6 Sub-Managers produced compilable, test-passing code on the first attempt
2. **Pure static class pattern** (MeleeCommitment, InputBuffer, CastableMovementStartup) enabled comprehensive EditMode testing without runtime dependencies
3. **"Write code directly" decision** was correct for all 6 Sub-Managers — tight coupling between docs/tests/code in each sub-plan made worker spawning overhead unjustified
4. **Shared file management** (SP-C and SP-D) — SP-D carefully documented what SP-C had already added, preserving all changes with additive edits
5. **Dormant-by-default pattern** kept all existing tests passing — no behavioral change until designers tune values
6. **Full regression caught the thread-safety bug** — the pipeline's Phase 5 safety net worked as designed

## Files Modified

- `Company/roles/sub-manager.md` — Rules 9, 10
- `Company/roles/tester.md` — Rule 13
- `Company/roles/implementer.md` — Thread Safety subsection
- `Company/project-guidelines.md` — AI thread gotcha, Dormant-by-Default pattern, Config-Driven Test Convention
- `Company/learnings.md` — Changelog entry
- `Company/pipelines/2026-03-28-player-action-time-windows.md` — Optimizer Findings section

## Notes for Future

This is the cleanest Expand pipeline to date: 6 sub-plans, 0 in-plan debug iterations, 2 Manager quick-fixes, 1 regression fix. The combination of pure-static-method extraction + dormant-by-default + Sub-Managers writing code directly is a proven pattern for tightly coupled, configurable feature additions. The only failure mode was a cross-cutting concern (thread safety) that scoped tests cannot catch by design — the full regression phase is the correct safety net for these.
