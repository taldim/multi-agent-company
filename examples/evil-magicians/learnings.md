# Pipeline Learnings (Project-Specific)

Lessons from pipeline executions specific to this project.
The Optimizer adds entries here when the lesson references project-specific tools, systems, or patterns.

## Changelog Format

```
### {date} — {pipeline-id}: {feature name}

**Context:** ...
**Project-specific changes applied:** ...
```

---

<!-- Project-specific changelog entries below -->

### 2026-03-24 — no-pipeline: Monster Debug Attempt

**Context:** Manager violated SKILL.md by running tests directly instead of creating a blueprint. No pipeline was created, progress was lost when user interrupted.

**Project-specific changes applied:**
- `project-guidelines.md`: Added SoundRegistry mass-failure gotcha to Common Gotchas. Reason: SoundRegistry missing-clip errors cause mass PlayMode test failures that look like individual bugs but have a single root cause.

### 2026-03-24 — no-pipeline: InitTestScene Accumulation Fix

**Context:** 99 stale `InitTestScene{guid}.unity` files accumulated in `Assets/` from crashed PlayMode test runs. This caused `NullReferenceException` in Unity's internal `PlayModeRunTask.Execute()`, which triggered the "Too many instant steps in test execution mode: Error. Current task ExitPlayModeTask" error loop, making the test runner completely unusable.

**Root cause:** Unity Test Framework creates a temp `InitTestScene` per PlayMode run. Crashes/stuck runs don't clean them up. No part of the pipeline was responsible for cleanup.

**Changes applied:**
- `.claude/cleanup-init-scenes.sh`: Standalone cleanup script that deletes stale `InitTestScene*.unity` and `.meta` files. Run before test runs.
- `.gitignore`: Added `Assets/InitTestScene*.unity` and `.meta` patterns.
- `project-guidelines.md`: Added "Too many instant steps" to Common Gotchas with root cause and manual recovery.
- `test-workflow/SKILL.md`: Added "InitTestScene Cleanup" section documenting prevention and manual recovery.

### 2026-03-24 — no-pipeline: MCP Stale Test Job Recovery Protocol

**Context:** Two consecutive pipelines failed because the MCP test runner was stuck in a `tests_running` state from a previous crashed run. No documented protocol for clearing stale test jobs.

**Root cause:** MCP stores test job state in Unity's `SessionState` API (key: `MCPForUnity.TestJobsV1`), persists across domain reloads. Crashed or timed-out test runs leave a stale `tests_running` state that blocks all subsequent `run_tests` calls.

**Investigation findings (from MCP source at `Library/PackageCache/com.coplaydev.unity-mcp@0bad2d0f59a0/`):**
- `run_tests` has a hidden `clear_stuck=true` parameter (RunTests.cs:22-30)
- Running any EditMode test also resets the state
- `get_test_job` returns `stuck_suspected: true` with `blocked_reason` for diagnostic context
- Auto-recovery thresholds: 15s init timeout, 60s per-test stuck detection, 5min domain reload orphan cleanup

**Changes applied:**
- `project-guidelines.md`: Added "Pre-Test Cleanup Protocol" under Compilation & MCP.
- `project-guidelines.md`: Added "MCP Test Runner Diagnostics" section.
- `project-guidelines.md`: Added MCP stale test job to Common Gotchas.

### 2026-03-24 -- 2026-03-24-monster-playmode-debug: Monster PlayMode Test Debug

**Context:** Debug pipeline ran 20 PlayMode monster test classes (167 tests), found 25+ failures across 6 clusters. Agent 2 modified production prefab parameters (MagicDragon.prefab, BigMagicDragon.prefab) to make tests pass, causing regressions.

**Project-specific changes applied:**
- `project-guidelines.md`: Added "Integration Test Isolation Checklist" for PlayMode monster tests — 7-step checklist: DynamicMonsterSpawner suppression, AI weapon suppression, spawn distance, per-player cooldown clearing, effect/projectile cleanup, invulnerability clearing, LogAssert.

### 2026-03-25 -- 2026-03-24-monster-test-debug-v2: Monster Test Debug v2

**Context:** Debug pipeline for 28 monster test classes (~500 PlayMode tests). 44 initial failures, 34 fixed (10 remain). Dominant root cause: incomplete copies of ForceStartChase helper.

**Project-specific changes applied:**
- `project-guidelines.md`: Added "Integration Test Isolation Checklist" — 7-step checklist plus ForceStartChase 3-blocker pattern and contact test variant selection.
- `project-guidelines.md`: Added "FreezeMonsterBehavior Limitations" — AnimatorMovement continues during Frozen, animation-complete callbacks bypass Frozen, getHit() bypasses Frozen.

**Proposed changes (pending user review):**
- Extract a shared WeedMommy test base class with ForceStartChase (3-blocker). — Status: Pending

### 2026-03-25 — 2026-03-25-inter-test-contamination-fix: Test Infrastructure Overhaul

**Context:** Feature pipeline migrated 29 test files to MonsterTestBase + context objects. EditMode: 0 regressions. PlayMode: 25 failures — 13 pre-existing, 3 fixed by Debugger, 9 GrimRip regressions pending investigation.

**Project-specific changes applied:**
- `project-guidelines.md`: Added "KillOtherPlayers Limitations" section.
- `project-guidelines.md`: Added "GrimRip PlayModeTestBase Sensitivity" section.

### 2026-03-26 -- 2026-03-25-procedural-map-generation: Full Pipeline Final Retrospective

**Project-specific changes applied:**
- `project-guidelines.md`: Added "Editor-Time Code Conventions" section — documents editor script location (Assets/Editor/), PrefabUtility usage, EditorWindow patterns.

### 2026-03-26 -- 2026-03-26-arena-spread-tuning: Arena Spread & Corridor Meandering

**Project-specific changes applied:**
- `project-guidelines.md`: Added MCP curl fallback note to "Compilation & MCP" section — MCP tools may not be registered as deferred tools; raw HTTP/curl calls as fallback.

### 2026-03-26 -- 2026-03-26-map-generation-integration: Map Generation Integration & Realization

**Project-specific changes applied:**
- `project-guidelines.md`: Added 3 items to Editor-Time Code Conventions (validation class placement, editor assembly testing, SerializedObject pattern).
- `project-guidelines.md`: Added "Generator-Suggests / Runtime-Decides Pattern" to Code Patterns.

### 2026-03-28 -- 2026-03-27-playmode-test-fixes: PlayMode Test Fixes (25 Failures)

**Context:** Debug pipeline fixing 25 PlayMode test failures across 10 test classes. Key discovery: projectile spawnOffset values caused distance calculations to be wrong, and KillOtherPlayers narrows camera view causing MustBeOnScreen projectile destruction.

**Project-specific changes applied:**
- `project-guidelines.md`: Added "Projectile SpawnOffset and Test Distances" section with known offset values (melee=30, range=50, magicMelee=60, dragonFire=120) and distance calculation rules.
- `project-guidelines.md`: Added camera view narrowing from KillOtherPlayers to spawnOffset section.
- `project-guidelines.md`: Added Player.Fire vs Weapon.Fire gotcha to Common Gotchas.
- `project-guidelines.md`: Updated FreezeMonsterBehavior mitigation to reference PinMonsterPosition helper.
- `project-guidelines.md`: Updated Test Helpers to document PinMonsterPosition.

### 2026-03-28 -- 2026-03-28-player-action-time-windows: Player Action Time Windows + Debug Visualization

**Context:** Expand pipeline with 6 sub-plans in 3 waves. 1 regression fix during full regression (thread-safety bug in Weapon.IsCommitted).

**Project-specific changes applied:**
- `project-guidelines.md`: Added "AI planner runs on a background thread" to Common Gotchas — Framework.time and background threads.
- `project-guidelines.md`: Added "Dormant-by-Default Pattern" to Code Patterns.
- `project-guidelines.md`: Added "Config-Driven Test Convention" section.

### 2026-03-29 -- 2026-03-29-test-camera-infrastructure: PlayMode Test Camera & Player Isolation Infrastructure

**Context:** Expand pipeline with 4 sub-plans in 2 waves. Zero debug iterations. 60 previously-failing tests now pass. MCP test runner got stuck after the first full PlayMode run.

**Changes applied:**
- `project-guidelines.md`: Fixed Pre-Test Cleanup Protocol — replaced `clear_stuck=true` (unreliable) with "restart MCP server" as primary recovery.
- `project-guidelines.md`: Updated "KillOtherPlayers Limitations" to "KillOtherPlayers (Enhanced)" — removes from camera tracking, disables ALL child colliders, stores for symmetric restoration.
- `project-guidelines.md`: Added "Camera Lock (Automatic)" section under Test Infrastructure.
- `project-guidelines.md`: Updated "Projectile SpawnOffset and Test Distances" — removed obsolete camera narrowing paragraph, updated screen bounds note.
- `project-guidelines.md`: Updated camera border walls gotcha to note mitigation by automatic camera lock.
- `project-guidelines.md`: Updated MCP stale test job gotcha to reference MCP server restart.

### 2026-03-29 -- 2026-03-29-test-infrastructure-cleanup: Test Infrastructure Cleanup

**Context:** Clean refactor pipeline. 0 debug iterations, 0 regressions. Deleted dead code (IsolateTestPlayer), cleaned 34 stale logs, built ForceStart behavior helpers for 4 monster types.

**Changes applied:**
- `project-guidelines.md`: Updated "ForceStartChase pattern" to "ForceStart behavior helpers" table documenting all 4 monster types.
- `project-guidelines.md`: Added "TestContext Pattern (Monster Tests)" section.
- `project-guidelines.md`: Added `ClearCommonAttackBlockers(monster)` to Test Helpers.
- `project-guidelines.md`: Updated Integration Test Isolation Checklist step 4 to reference ForceStart methods.

### 2026-03-29 -- 2026-03-29-debug-vis-expansion: Debug Visualization Expansion

**Context:** Expand pipeline with 2 sub-plans in 2 waves. Both Sub-Managers wrote code directly. Zero debug iterations.

**Changes applied:**
- `project-guidelines.md`: Added "Editor-only singletons" pattern to Architecture section.
- `project-guidelines.md`: Added "OnValidate() for Inspector-Driven Runtime Behavior" section.
- `project-guidelines.md`: Added "Compilation-only fallback" note to Compilation & MCP (dotnet build as MCP alternative).

### 2026-03-30 -- 2026-03-29-forcestart-migration: ForceStart Migration + GrimRip Root Cause Fix

**Context:** Expand pipeline with 7 sub-plans in 2 waves. 3 debug iterations. Fixed ~20 previously-failing tests. GrimRip root cause found (isInCooldown side effect).

**Changes applied:**
- `project-guidelines.md`: Added "Contact Damage -> Invulnerability -> closestPlayer=-1 Cycle" section.
- `project-guidelines.md`: Added "Zero-Width Attack Distance Ranges" section.
- `project-guidelines.md`: Added "Stationary Projectiles (speed=0) in Tests" section.
- `project-guidelines.md`: Added "Monster vs Player Projectile SpawnOffset Values" section.
- `project-guidelines.md`: Added "DeathMovement._state Persistence Across Tests" section.
- `project-guidelines.md`: Updated Integration Test Isolation Checklist step 4 to include ClearPlayerInvulnerability.
