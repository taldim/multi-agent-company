# Visionary Recommendations (Project-Specific) -- Curated Subset

Strategic recommendations from the Visionary specific to this project.
The Manager reads this at the start of every /company-plan session to surface systemic issues.

**Note**: This is a curated subset of ~50 total recommendations, selected to show the breadth of what the Visionary produces. Categories represented: Architecture, Tooling, Process, Tech Debt, Conventions.

---

### 2026-03-26 — 2026-03-25-procedural-map-generation: Sub-Pipeline 3 Review

**Priority**: Critical
**Category**: Architecture
**Recommendation**: MapRealization class is now CRITICAL. Three sub-pipelines completed (167 tests, zero debug iterations) but the system still cannot produce a playable level.
**Rationale**: Elevated from HIGH (SP2 review). Three sub-pipelines with zero usable output.
**Effort**: Medium (1 pipeline phase)
**Impact**: Without this, the entire map generation system produces no playable output.

---

### 2026-03-26 — 2026-03-26-map-generation-integration: Placeholder Sprites Will Not Survive Prefab Serialization

**Priority**: High
**Category**: Tech Debt
**Recommendation**: `MapRealization.CreateColoredSprite()` creates non-persistent `Texture2D` and `Sprite` objects. After domain reload, SpriteRenderers will have null sprites.
**Rationale**: The Implementer's journal explicitly flags this as a known issue that was deferred.
**Effort**: Small
**Impact**: Without this, saved level prefabs have invisible walls when loaded in a fresh editor session.

---

### 2026-03-26 — 2026-03-26-map-generation-integration: Previous HIGH Recommendations Now Addressed

**Priority**: N/A (Status Update)
**Category**: Process
**Recommendation**: Status tracking:
- "MapRealization class is now CRITICAL" -- RESOLVED
- "Wire CoverScatter and ArenaScatter into MapAssembler" -- RESOLVED
- "Add round-trip integration test" -- RESOLVED
- "Replace silent Vector2.zero fallbacks with throws" -- Status unclear
- "Config alignment" -- PARTIALLY RESOLVED (constants migrated, ScriptableObject not created)

---

### 2026-03-26 — 2026-03-26-map-validation-fixes: ObstacleCategory Semantics — Placement Origin vs Collision Behavior

**Priority**: High
**Category**: Architecture
**Recommendation**: Refactor `ObstacleCategory` into two orthogonal dimensions: **CollisionType** (`Hard`, `Soft`) and **PlacementOrigin** (`Edge`, `DeadEndCap`, `TurnFill`, `IntersectionBorder`).
**Rationale**: 5 of 6 test failures in this pipeline were caused by the inability to distinguish turn-fill walls from edge walls.
**Effort**: Small
**Impact**: Eliminates the class of test failures caused by category confusion.

---

### 2026-03-27 — 2026-03-27-map-robustness: ObstacleCategory Zombie — Refactor Left the Old Enum Alive

**Priority**: High
**Category**: Tech Debt
**Recommendation**: The ObstacleCategory refactor added `CollisionType` and `PlacementOrigin` but `ObstacleCategory` was never removed. It survives in 6 files. Every new `WallPlacement` creation now populates three classification fields.
**Rationale**: The Implementer couldn't remove it because `MapRealizationValidatorTests` still referenced it, creating a circular dependency.
**Effort**: Small
**Impact**: Eliminates triple-classification bookkeeping.

---

### 2026-03-27 — 2026-03-27-map-robustness: Previous Recommendations Status Update

**Priority**: N/A (Status Update)
**Category**: Process
**Recommendation**: Status tracking:
- "ObstacleCategory Semantics" -- RESOLVED (CollisionType and PlacementOrigin implemented, but old enum not removed)
- "WallBuildConfig Extraction" -- RESOLVED
- "Map-bounds wall clamping" -- PARTIALLY RESOLVED
- "Ban Index-Based Wall Pairing" -- RESOLVED

---

### 2026-03-27 — 2026-03-27-sound-visual-testing: Massive Code Duplication Across 37 Scenario Classes

**Priority**: High
**Category**: Tech Debt
**Recommendation**: Extract shared helper methods from `SoundScenario` into the base class or a shared utility class. 37 scenario classes with identical utility methods duplicated across files. Create intermediate base classes: `MonsterDeathScenario`, `MonsterSpawnScenario`, `MonsterAttackScenario`.
**Rationale**: 37 classes total approximately 3000 lines. With extracted helpers, approximately 800 lines.
**Effort**: Small
**Impact**: Reduces total scenario code by ~70%.

---

### 2026-03-28 — 2026-03-27-playmode-test-fixes: Camera-Aware Test Positioning Helper

**Priority**: High
**Category**: Tooling
**Recommendation**: Create `PositionForProjectileTest(testPlayer, monster, spawnDistance)` in `PlayModeTestBase` that handles all camera/screen-bounds concerns.
**Rationale**: 12 of 25 failures from camera view narrowing + spawnOffset + MustBeOnScreen interaction.
**Effort**: Small
**Impact**: Eliminates the single largest class of PlayMode test failures.

---

### 2026-03-28 — 2026-03-27-playmode-test-fixes: Expose spawnOffset Programmatically for Tests

**Priority**: High
**Category**: Tooling
**Recommendation**: Add `ProjectileSpawn.GetSpawnOffset(GameObject prefab)` for tests to read spawnOffset from prefabs programmatically.
**Rationale**: Tests currently hardcode per-prefab spawnOffset values that break when designers change them.
**Effort**: Small
**Impact**: Tests auto-adapt when designers change spawnOffset.

---

### 2026-03-28 — 2026-03-27-playmode-test-fixes: SmallMagicDragonTests Chronic Flakiness -- STOP

**Priority**: Critical
**Category**: Architecture
**Recommendation**: STOP patching SmallMagicDragonTests individually. 16 of 18 tests failed with zero code changes. This is the 3rd pipeline with mass failures. Investigate in a dedicated diagnostic pipeline with systematic isolation.
**Rationale**: Three pipelines, three mass failure events. No individual test fix will address this.
**Effort**: Medium
**Impact**: Either fixes 18 tests permanently or identifies need for fundamentally different test architecture.

---

### 2026-03-28 — 2026-03-27-playmode-test-fixes: PlayMode Test Infrastructure Fragility -- Recurring Pattern

**Priority**: High
**Category**: Process
**Recommendation**: Create a "PlayMode Test Hardening" plan that systematically addresses all 4 recurring failure categories: (1) FreezeMonsterBehavior gaps, (2) AI interference, (3) camera/screen bounds, (4) inter-test state bleed. This is infrastructure, not per-test effort.
**Rationale**: 5 pipelines dedicated to fixing PlayMode test infrastructure with the same failure categories.
**Effort**: Large
**Impact**: Reduces PlayMode test maintenance from "multiple debug pipelines per month" to "rare individual failures."

---

### 2026-03-28 — 2026-03-28-player-action-time-windows: Weapon.cs State Accumulation

**Priority**: High
**Category**: Architecture
**Recommendation**: Extract three new state machines (deferred fire, melee commitment, input buffer) from `Weapon.cs` into dedicated internal state holders. See `Documentation/Plans/PlayerActions/WeaponStateExtraction_Plan.md`.
**Rationale**: Weapon.cs grew from ~490 to 581 lines. Three independent state machines share one Update method and one `finishFire()` cleanup.
**Effort**: Small
**Impact**: Isolated state machines are easier to debug, test, and extend.

---

### 2026-03-28 — 2026-03-28-player-action-time-windows: Undocumented Player Action Blocking Matrix

**Priority**: High
**Category**: Conventions
**Recommendation**: Create `Documentation/Technical/PlayerActionStateMachine_Technical.md` documenting the complete blocking matrix between all 5 player actions and their time window phases. See `Documentation/Plans/PlayerActions/PlayerActionStateMachine_Plan.md`.
**Rationale**: Blocking relationships are scattered across 4 files. A designer cannot answer "what happens if I press teleport during melee startup?" without reading all 4 files.
**Effort**: Small
**Impact**: Designers can reason about action interactions without reading code.

---

### 2026-03-28 — 2026-03-28-player-action-time-windows: Debug Visualization Architecture Is Clean

**Priority**: Low (Positive Note)
**Category**: Architecture
**Recommendation**: No action needed. The debug visualization system follows an exemplary architecture: pure static state query classes separated from editor-only MonoBehaviour visualizers.
**Rationale**: Recording positive patterns is as important as flagging problems.
**Effort**: N/A
**Impact**: Reference pattern for future debug systems.

---

### 2026-03-29 -- 2026-03-29-test-infrastructure-cleanup: Pre-Existing Failure Accountability Convention

**Priority**: Medium
**Category**: Conventions
**Recommendation**: Add convention: "No pipeline may close with more pre-existing test failures than the last pipeline." Require failure count comparison and investigation.
**Rationale**: Failure count trajectory has been stable/growing across 4+ pipelines.
**Effort**: Small
**Impact**: Creates accountability preventing silent failure accumulation.

---

### 2026-03-30 -- 2026-03-29-forcestart-migration: WaitForBehavior High-Level Helper

**Priority**: High
**Category**: Tooling
**Recommendation**: Add `WaitForBehavior<T>(monster, timeout)` and `WaitWhileBehavior<T>(monster, duration)` helpers to `MonsterTestBase` that encapsulate the full blocker-clearing loop.
**Rationale**: 18 of 20 fixes in this pipeline required the identical boilerplate pattern.
**Effort**: Small
**Impact**: Eliminates the most common test failure pattern.

---

### 2026-03-30 -- 2026-03-29-forcestart-migration: Condition Callback Purity Convention

**Priority**: High
**Category**: Conventions
**Recommendation**: Document and enforce that condition callbacks registered via `addConditionCallback()` must be pure queries with no side effects. Add to `project-guidelines.md`, `CLAUDE.md`, and `MonsterBehaviourManager.cs` doc comments.
**Rationale**: The GrimRip root cause was a side-effect-laden condition callback causing permanent behavior deadlock. Took extensive debugging across 2 pipeline iterations.
**Effort**: Small
**Impact**: Prevents the most expensive class of bug found in recent pipelines.
