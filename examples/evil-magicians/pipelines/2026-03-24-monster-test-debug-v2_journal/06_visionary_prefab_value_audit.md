# Journal: Visionary — Prefab Value Audit of Monster PlayMode Tests

## What I Reviewed

Performed a systematic audit of all 30 test files in `Assets/Tests/PlayMode/Monsters/`. For each file, I:
- Identified every hardcoded numeric value used for positioning, timing, or counting
- Cross-referenced with serialized fields on the corresponding monster base classes (`DemonBase`, `MagicDragonBase`, `GrimRipBase`, `weedMommyBase`)
- Checked `PlayModeTestBase` for existing config-reading helpers (none exist)
- Noted which test files already follow the correct pattern of reading from spawned instances

## Key Observations

**Systemic pattern**: The Demon and MagicDragon test families are the worst offenders. DemonBlockTests alone has 6 instances of hardcoded `3` for block charges and multiple hardcoded cooldown durations. BigMagicDragonTests hardcodes `450` for attack distance in 3 places.

**WeedMommy tests are the gold standard**: `WeedMommyBehaviorTests`, `WeedMommyChargeCooldownTests`, and `WeedMommyHandsAttackTests` already read all configurable values from spawned instances via public fields or reflection. These were likely written later and learned from earlier mistakes.

**Partial fix exists**: `DemonBlockTests.SmallDemon_BlockDepletion_EntersCooldown` already reads `hitsAmountBeforeBlock` via reflection. But other tests in the same file still hardcode `3`.

**Scale of the problem**: 8 HIGH-priority tasks, 4 MEDIUM, 5 LOW. Approximately 40-50 individual hardcoded values need replacement across ~15 files.

**PlayModeTestBase needs helpers**: There are no existing config-reading helpers. Each test class reinvents reflection access to the same fields. Task 0 (add helpers) is the critical prerequisite that makes all other tasks simple.

## Recommendations Filed

Plan document written to `Documentation/Plans/Monsters/PrefabValueRefactor_Plan.md` with:
- 24 tasks (Task 0 = helpers, Tasks 1-23 = per-class refactors)
- 8 files at HIGH priority, 4 at MEDIUM, 5 at LOW, 9 already clean
- Code snippets showing before/after patterns for each category of fix
- Priority summary for execution ordering

## Notes for Optimizer

- Task 0 (helper methods) is a strict prerequisite for all other tasks. Execute it first.
- HIGH priority tasks should be batched by monster type: Dragons (Tasks 1-2), Demons (Tasks 3-8), GrimRip (Task 10). Each batch can be done independently once Task 0 is complete.
- The Implementer should reference WeedMommyBehaviorTests as the model pattern. Show them that file, not just the fix snippets.
- Total estimated effort: Small-Medium (2-3 hours for an experienced Implementer). Most changes are mechanical find-and-replace after Task 0 is done.
