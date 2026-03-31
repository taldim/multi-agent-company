# Journal: Documenter -- WeedMommy Chase/Charge Technical Documentation

## What I Did
- Read all WeedMommy production code (weedMommyBase.cs, weedMommy.cs, BigWeedMommy.cs, ChargePlayerBehaviour.cs, MonsterBehaviourManager.cs)
- Read all related test files (WeedMommyBehaviorTests, WeedMommyChargeTriggerTests, WeedMommyChaseWalkTests, WeedMommyChargeCooldownTests, WeedMommyGateChargeTests, WeedMommyDebuffTests, SmallWeedMommyTests, WeedMommyCooldownTests, WeedMommyHandsAttackTests)
- Read previous debugger journal entries (07, 08) to understand the 7 failing tests and root causes already identified
- Created Documentation/Technical/WeedMommy_ChaseCharge_Technical.md covering all 6 requested areas plus a data flow summary
- Updated Documentation/README.md with the new doc link

## Decisions Made
- Documented the per-player cooldown system in detail because it was identified in debugger journals 07/08 as the PRIMARY root cause of test failures (ChaseWalkBehaviour.targetSelectedCallback setting cooldowns during init)
- Included the "Ambiguities and Potential Test-Breaking Conditions" section (Section 8) specifically to aid debug tasks -- flags 6 known gotchas that can cause test failures
- Used the format of existing technical docs (AnimatorMovement_Technical.md) as the template
- Did NOT hardcode prefab values -- all configurable values (speeds, distances, cooldowns) reference the serialized field names and note they are configurable on prefab

## Problems Encountered
- The updateWhenChasing parameter in ChargePlayerTrigger has misleading naming: true means "skip updates when chasing" (the opposite of what the name suggests). Documented this prominently in Section 4.2.
- ChaseWalkBehaviour and ChargePlayerBehaviour share the same priority value (48) but are in different tiers (normal vs interrupt). Documented clearly in the priority table.

## Assumptions
- The 7 failing chase tests mentioned in the task brief correspond to the 7 tests documented in debugger journal 08
- The debugger has already applied fixes to these 7 tests. The documentation serves as reference for verifying those fixes and for any further debug work needed.

## Files Modified
- Documentation/Technical/WeedMommy_ChaseCharge_Technical.md (NEW -- 9 sections, ~280 lines)
- Documentation/README.md (added link to new doc in Technical Architecture table)

## Notes for Optimizer
- The stopChaseButFinishAnimation() call on line 275 of weedMommyBase is called unconditionally in onHitOther, even for hands attack contact. This sets flags on the charge behavior even when it's not active. The flags are cleaned up by _onNotSelected() every frame, so it's harmless, but it's a code smell worth noting.
- The per-player cooldown being set by ChaseWalkBehaviour on every frame during target selection (not just on transition) is the single biggest source of test fragility for the WeedMommy charge system. Every test that uses ForceStartChase must clear this cooldown.
