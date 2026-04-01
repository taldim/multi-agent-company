# Pipeline Learnings (Generic)

Cross-project best practices distilled from real pipeline executions.
These ship with the multi-agent-company template and apply to any project.

The Optimizer updates this file with lessons that are universally applicable.
Project-specific lessons go to `Company/project/learnings.md`.

## Changelog Format

```
### {date} — {pipeline-id}: {feature name}

**Context:** ...
**Changes applied:** ...
```

## Proposed Role Changes (Pending Review)

Significant changes the Optimizer flagged for user approval before applying.

```
### {date} — {pipeline-id}
**File**: {role-file}
**Proposed change**: {description}
**Reason**: {what went wrong that motivates this}
**Status**: Pending | Approved | Rejected
```

---

<!-- Generic changelog entries below -->

### 2026-03-24 — no-pipeline: Monster Debug Attempt

**Context:** Manager violated SKILL.md by running tests directly instead of creating a blueprint.

**Changes applied:**
- `company-plan/SKILL.md`: Added Debug Pipeline exception to "3 questions" rule — reduced to "1 scoping question" for Debug Pipelines. Reason: Debug directives are often unambiguous; forcing 3 questions is counterproductive.
- `company-plan/SKILL.md`: Added negative example to Critical Rules section showing what the Manager should NOT do (acquire test infrastructure, run tests directly). Reason: Manager violated Rule 3 directly.
- `company-plan/SKILL.md`: Expanded Debug Pipeline section in Step 4 with detailed step-by-step workflow and blueprint template. Reason: Debug Pipeline instructions were too thin compared to Feature Pipeline.
- `roles/debugger.md`: Added "Step 0: Cluster Failures by Error Pattern" before individual diagnosis. Reason: Systemic issues (like a missing dependency causing mass failures) should be identified before spending time on individual test failures.

**Post-review fixes (user caught leaks):**
- `company-plan/SKILL.md`: Removed project-specific tool names from Critical Rules — replaced with generic "test infrastructure."
- `roles/debugger.md`: Removed project-specific error example from Step 0 — replaced with generic "initialization error or missing dependency."

### 2026-03-24 — self-improvement: Generic vs Project-Specific Enforcement

**Context:** Optimizer's own edits leaked project-specific content (tool names, system names, examples) into generic role files. User caught it during review.

**Changes applied:**
- `roles/optimizer.md`: Added "Generic vs Project-Specific Litmus Test" — a 3-question checklist the Optimizer must run on every sentence before writing it to a role/SKILL file. Includes post-write re-read verification step. Reason: Optimizer repeatedly embedded project-specific examples despite existing guidance.

### 2026-03-24 — no-pipeline: Manager Autonomy & Guidelines Feedback Loop

**Context:** Manager asked an unnecessary scoping question for a Debug Pipeline where the user's intent was unambiguous.

**Changes applied:**
- `company-plan/SKILL.md`: Changed Debug Pipeline exception to "default to 0 questions." Reason: Debug directives are self-scoping.
- `company-plan/SKILL.md`: Added "Autonomy Principle: Decide, Don't Ask" section with litmus test. Reason: Manager had no guidance on WHEN NOT to ask.
- `company-plan/SKILL.md`: Added Step 5.5 "Guidelines Sufficiency Audit" — Manager checks what info was missing from project-guidelines.md and logs it as Guidelines Gaps. Reason: Creates feedback loop so each pipeline improves guidelines.
- `company-plan/SKILL.md`: Added `## Guidelines Gaps` to blueprint template.
- `roles/optimizer.md`: Added "5. Audit Manager Autonomy" to analysis checklist + "Manager Autonomy Audit" table. Reason: Optimizer must check if Manager asked unnecessary questions and backfill project-guidelines.

### 2026-03-24 — self-improvement: Optimizer Self-Audit on Autonomy Changes

**Context:** Optimizer applied autonomy improvements but leaked project-specific examples into generic SKILL.md. Caught during self-audit.

**Changes applied:**
- `company-plan/SKILL.md`: Replaced project-specific examples with generic placeholders.

**Lesson for Optimizer:** The litmus test catches leaks — but only if applied in real-time while writing, not just as a retrospective. Apply the test per-sentence AS you write.

### 2026-03-24 — no-pipeline: Pipeline Journal System

**Context:** Background agents complete work but only return concise summaries. The Optimizer at Phase 6 was effectively blind — no decision trails, problem reports, or root cause blame from each role.

**Changes applied:**
- `company-execute/SKILL.md`: Added "Pipeline Journal" section — Orchestrator creates `{id}_journal/` folder, passes path to every role. Added journal folder creation to Blueprint Mode and Phase 0. Added Orchestrator's own journal entry for Phase 4. Added Rule #12 requiring journal entries. Added journal path + file name to all role dispatch prompts.
- `roles/documenter.md`: Added "Journal Entry (MANDATORY)" section with template.
- `roles/tester.md`: Added "Journal Entry (MANDATORY)" section with template.
- `roles/implementer.md`: Added "Journal Entry (MANDATORY)" section with template.
- `roles/debugger.md`: Added "Journal Entry (MANDATORY)" section with template — includes "Which Role Caused This" field.
- `roles/visionary.md`: Added "Journal Entry (MANDATORY)" section with template.
- `roles/optimizer.md`: Added "Read the Pipeline Journal (PRIMARY SOURCE)" as first analysis step.

### 2026-03-24 -- 2026-03-24-monster-playmode-debug: Debug Never Modify Design Parameters

**Context:** Debugger agent modified production prefab parameters to make tests pass, causing regressions (5 to 12 failures).

**Changes applied:**
- `roles/debugger.md`: Added Rule 8: "NEVER modify design parameters to make tests pass" — change test assumptions, not production assets.
- `roles/debugger.md`: Added Rule 9: "Verify your fix does not break other tests."
- `roles/debugger.md`: Added "Stale test" as a root cause category — test assumptions outdated because design changed.
- `roles/debugger.md`: Added "Protected Production Assets" section.
- `roles/debugger.md`: Added "Escalation Triggers" section.
- `roles/debugger.md`: Added "Production assets modified" field to Report Format and Journal Entry template.

### 2026-03-24 — Cluster-First Debug Escalation Strategy

**Context:** Two parallel Debugger agents handling 3 clusters each. Agent 1 succeeded. Agent 2 failed — wrong root cause applied in batch caused cascading regressions.

**Changes applied:**
- `company-execute/SKILL.md`: Rewrote Debug Pipeline Phase 2 from flat "spawn Debugger, retry 3 times" to **escalating strategy**: Iterations 1-2 use cluster mode; Iteration 3+ falls back to single-test mode. Explicit switch triggers: cluster fix caused regressions, same cluster failed 2 iterations, or Debugger modified production assets.
- `roles/debugger.md`: Added Rule 10: "If your cluster fix causes regressions, stop."

### 2026-03-25 -- 2026-03-24-monster-test-debug-v2: Integration Test Setup Rules

**Context:** 44 initial failures, 34 fixed, dominant root cause: incomplete copies of helpers duplicated across test files.

**Changes applied:**
- `roles/tester.md`: Added "Integration Test Setup Rules" section — 5 rules: (1) Never hardcode configurable values, (2) Reuse existing test helpers completely, (3) Verify setup preconditions match runtime state, (4) Choose the right test subject variant, (5) Isolate from background systems.
- `roles/debugger.md`: Added Rule 11 "Apply fixes consistently across the module" — scan sibling files for same vulnerability.
- `roles/debugger.md`: Added Rule 12 "Clean up diagnostic artifacts."
- `company-execute/SKILL.md`: Added batch-fix guidance — when same root cause appears across multiple files, batch into single Debugger dispatch.

### 2026-03-25 — 2026-03-25-inter-test-contamination-fix: Base Class Changes Require Impact Analysis

**Context:** Migrated 29 test files. Enhanced base class cleanup caused 9 regressions in a timing-sensitive subclass.

**Changes applied:**
- `roles/tester.md`: Added Rule 6: "Understand what the base class does before removing setup code."
- `roles/tester.md`: Added Rule 7: "Anticipate side effects of enhanced base class cleanup."
- `roles/debugger.md`: Added "Regression Triage for Base Class Changes" section.
- `roles/implementer.md`: Added "Shared Infrastructure Changes" subsection.
- `company-execute/SKILL.md`: Added Rule 10: "Parallel migration agents must run a single verification pass after ALL complete."
- `company-execute/SKILL.md`: Added Rule 11: "Flag high-risk migration targets."

### 2026-03-26 -- 2026-03-25-procedural-map-generation: Documenter Must Read Actual Code in Iterative Sub-Pipelines

**Context:** Sub-Pipeline 2 completed with 0 failures. Sub-Pipeline 1 had 16 failures. Difference: SP2's Documenter read actual SP1 code before writing updated docs.

**Changes applied:**
- `roles/documenter.md`: Added Rule 6: "In iterative sub-pipelines, read actual code before updating docs."

### 2026-03-26 -- 2026-03-25-procedural-map-generation: TDD Stub Creation Communication

**Context:** SP1's 16 failures were caused by divergent interpretations between Tester stubs and Implementer code.

**Changes applied:**
- `roles/tester.md`: Added "TDD Stub Creation" section — documents the pattern and the responsibility to communicate interpretation assumptions in the journal.

### 2026-03-26 -- 2026-03-25-procedural-map-generation: Pure-Static-Method Architecture Validation

**Context:** Three consecutive sub-pipelines with zero Debugger dispatches.

**Pattern validation:** Confirms the pure-static-method + unit-test-only architecture as the optimal pattern for greenfield algorithmic modules.

### 2026-03-26 -- 2026-03-26-arena-spread-tuning: Thresholds Must Be Algorithm-Derived

**Context:** 3 of 4 test failures were preventable. Tester used 60% coverage threshold from test spec without verifying the algorithm could guarantee it. Actual worst-case was 59.9%.

**Changes applied:**
- `roles/tester.md`: Added Rule 8: "Derive thresholds from algorithm guarantees, not round numbers."
- `roles/documenter.md`: Added Rule 7: "Test spec thresholds must be derivable from the algorithm."
- `roles/implementer.md`: Added "Downstream Consumer Audit" section — audit downstream systems when changing algorithm behavior.

### 2026-03-26 -- 2026-03-26-map-generation-integration: TDD Stubs Should Flag API Deviations

**Context:** Tester created a struct instead of the documented tuple return type but described it as "per the task spec" instead of flagging the deviation.

**Changes applied:**
- `roles/tester.md`: Added bullet to TDD Stub Creation: "Flag API deviations from documentation."

### 2026-03-26 -- 2026-03-26-map-generation-integration: Four Consecutive Zero-Debug Pipelines

**Pattern validation:** Four consecutive pipelines with zero Debugger dispatches confirms pure-static-method + unit-test-only + iterative-doc-sync as the optimal pattern for greenfield algorithmic modules.

### 2026-03-26 -- 2026-03-26-map-wall-tightness: Validate Assertions Against Actual Data

**Context:** 3 of 4 failures were Tester bugs (untested tolerances, unreliable reverse-lookup validation). Debugger's test rewrite was also flawed, requiring second dispatch.

**Changes applied:**
- `roles/tester.md`: Added Rule 9: "Validate assertions against actual data before submitting."
- `roles/implementer.md`: Added "Spatial Region Ownership" subsection — when adding placement logic, suppress existing placement in the same spatial region.
- `roles/debugger.md`: Added Rule 13: "When rewriting a test, re-read the test documentation first."

### 2026-03-26 -- 2026-03-26-map-validation-fixes: New Categories Need Distinct Identifiers

**Context:** 5 of 6 test failures had the same root cause: new items lacked a distinct category, making them indistinguishable from existing items.

**Changes applied:**
- `roles/implementer.md`: Added "New Placement Origins Need Distinct Categories."
- `roles/tester.md`: Added Rule 10: "Identify items by category, not by list structure" — never use index parity or positional heuristics when categories are available.
- `roles/tester.md`: Added Rule 11: "Use 3+ seeds for different-seeds-produce-different-results tests."
- `roles/documenter.md`: Added Rule 8: "Specify how tests should identify items in heterogeneous outputs."

### 2026-03-27 -- 2026-03-27-map-robustness: Sequential Constraints and Filter Exemptions

**Context:** 1 Tester bug (missing filter exemption), 1 Implementer bug (sequential constraint violation), 1 Documenter inconsistency (enum value name disagreement).

**Changes applied:**
- `roles/implementer.md`: Added "Sequential Constraint Validation" — when applying post-generation transforms, re-check upstream constraints.
- `roles/tester.md`: Added Rule 12: "Mirror production filter exemptions in integration tests."
- `roles/documenter.md`: Added Rule 9: "Design doc and test spec must agree on enum values and API names."

### 2026-03-27 -- 2026-03-27-sound-visual-testing: Sub-Manager Code Writing and Shared Files

**Context:** 4 Sub-Managers wrote code directly instead of spawning workers. 3 parallel Sub-Managers modified the same factory file — last writer won by luck.

**Changes applied:**
- `roles/sub-manager.md`: Added "When to Write Code Directly vs. Spawn Workers" section with decision framework.
- `roles/sub-manager.md`: Added "Shared File Conflicts in Parallel Waves" section — read before modifying, preserve existing entries, use additive edits.

**Proposed changes (pending user review):**
- `company-execute/SKILL.md`: Add rule to identify shared files before launching parallel waves. — Status: Pending
- `company-expand/SKILL.md`: Add "Shared Files Across Parallel Sub-Plans" to blueprint template. — Status: Pending

### 2026-03-27 — Full Regression Rule for Large Pipelines

**Context:** A pipeline completed 160/160 scoped tests but never ran the full regression suite. Large enough that cross-module regressions could go undetected.

**Changes applied:**
- `company-expand/SKILL.md`: Added "Full Regression" step to blueprint Execution Order template.
- `company-plan/SKILL.md`: Added "Scale Check: When to Recommend `/company-expand`" — when 3+ sub-pipelines, 3+ modules, or 5+ new files.

### 2026-03-28 -- 2026-03-27-playmode-test-fixes: Spawn/Offset Geometry in Distance Fixes

**Context:** Debugger reduced spawn distances without checking weapon offset values, causing 5+ tests to need Manager iteration fixes. Manager applied direct code edits during verification.

**Changes applied:**
- `roles/debugger.md`: Added Step 3.5 "Account for Spawn/Offset Geometry in Distance Fixes."
- `roles/debugger.md`: Added "Verification-Phase Quick Fixes" section.
- `roles/tester.md`: Added Integration Test Writing Rule 6 "Use APIs That Report Success/Failure."
- `company-execute/SKILL.md`: Added "Flaky vs Real Failures" classification guidance.
- `company-execute/SKILL.md`: Added "Manager Quick-Fix During Verification" guidance.

### 2026-03-28 -- 2026-03-28-player-action-time-windows: Verify Enum/API Names and Thread Safety

**Context:** Sub-Manager used a non-existent enum value (trivial compilation fix). Another Sub-Manager added a property that called a main-thread-only API from a background thread.

**Changes applied:**
- `roles/sub-manager.md`: Added Rule 9: "Verify enum/API names against actual code" — read the source file to confirm values exist.
- `roles/sub-manager.md`: Added Rule 10: "Consider thread safety for properties on shared objects."
- `roles/tester.md`: Added Rule 13: "Test assertions must align with the stated design of the system under test."
- `roles/implementer.md`: Added "Thread Safety for Shared Properties" subsection.

### 2026-03-26 -- 2026-03-26-map-wall-tightness: Test Validation Strategies

**Context:** Debugger rewrote a test with a new flawed validation strategy without checking the test spec. Required second Debugger dispatch.

*(Role changes already captured in the "Validate Assertions Against Actual Data" entry above.)*

### 2026-03-25 -- 2026-03-25-procedural-map-generation: Visionary Recommendations Should Influence Manager Scoping

**Context:** SP2 Visionary identified 3 HIGH-priority gaps. SP3 Manager scoped work without checking Visionary recommendations.

**Changes applied:**
- `recommendations.md` (generic): Added recommendation that Managers should read `recommendations.md` before scoping sub-pipelines. *(The actual entry was project-specific, but the PRINCIPLE is generic.)*

### 2026-03-25 -- 2026-03-25-procedural-map-generation: Algorithm-Without-Integration Pipeline Pattern

**Context:** Second consecutive pipeline where core algorithms are fully tested but their integration into the runtime pipeline is silently dropped.

**Lesson:** Blueprint tasks should distinguish "algorithm" tasks from "integration" tasks. The Manager should verify both categories are dispatched. Tests passing on the algorithm API does not prove integration completeness.

### 2026-03-30 -- 2026-03-29-forcestart-migration: Root Cause Hypothesis Verification

**Context:** Blueprint hypothesized one root cause; Implementer correctly identified the actual cause, saving a wasted iteration.

**Changes applied:**
- `roles/implementer.md`: Added rule 4 "Verify the blueprint's root cause hypothesis" to Read Before Writing.
- `roles/debugger.md`: Added Step 3.6 "Check for Side Effects in Condition Callbacks."
- `roles/debugger.md`: Added "Inter-Test Contamination (Pass Individually, Fail in Suite)" section.
- `roles/tester.md`: Added bullets to Integration Test Writing Rule 3: check distance ranges for zero-width windows, check speed for stationary items.

### 2026-03-30 -- 2026-03-30-company-repo-separation: Multi-Agent Company Repo Separation

**Context:** Expand pipeline that separated the generic multi-agent Company framework from project-specific content. 8 sub-plans across 4 waves, all completed successfully. Zero debug iterations. SP-D and SP-E hit background agent write permission issues.

**Changes applied:**
- `roles/sub-manager.md`: Fixed "EditMode"/"PlayMode" terminology leaks in journal template — replaced with "Unit tests"/"Integration tests" for framework portability.
- `roles/tester.md`: Fixed "EditMode"/"PlayMode" terminology leaks in base class selection framework — replaced with "unit test base class"/"integration test base class".
- `roles/sub-manager.md`: Added "Write permission gotcha" to "When to Write Code Directly" section — background workers may lack write permissions for new directories; Sub-Manager must write files itself using worker output when this happens.
- `company-execute/SKILL.md`: Fixed "prefabs, ScriptableObjects" leak — replaced with generic "configuration files, designer-tuned data".

**Lesson for meta-pipelines:** When a pipeline restructures the pipeline infrastructure itself, the agents performing the work ARE the system being modified. This creates unique risks: (1) agents may read stale versions of files they're actively editing, (2) path references become invalid mid-pipeline as files are moved, and (3) the litmus test for "generic vs project-specific" requires extra vigilance because the agent is simultaneously an instance of the framework AND a user of it. The mitigation is strict wave ordering — audit/split phases first, creation phases second, wiring/verification phases last.

### 2026-03-31 -- 2026-03-31-sound-auto-runner: Clean Pipeline — Guidelines Gap Backfill

**Context:** Zero-debug pipeline (2 roles, 0 failures). Manager identified 2 Guidelines Gaps in the blueprint: no SoundTesting framework context in project-guidelines, no mention of ExpectedSoundEvents metadata pattern.

**Changes applied:**
- `Company/project/project-guidelines.md`: Added "Sound Testing Framework" section — key files, cross-product expected-sounds pattern, auto-run mode, safe coroutine iteration pattern (`MoveNext()` in `try/catch`), `SoundRegistry.OnSoundPlayed` event hook. Reason: Manager flagged these as Guidelines Gaps; backfilling prevents future pipelines from needing to re-discover this context.

**No generic role file changes.** Pipeline was clean — no failures, no role confusion, no process waste warranting new rules.

### 2026-03-31 -- 2026-03-31-sound-scenario-fixes: Root Cause Hypothesis Verification Pays Off (Again)

**Context:** Zero-debug pipeline. Blueprint's root cause hypothesis was partially wrong (assumed multi-identity metadata would work, but the validation system uses a strict cross-product). Implementer caught this by reading the actual validation code before implementing, exactly as the "Verify the blueprint's root cause hypothesis" rule in implementer.md prescribes.

**Pattern validation:** This is the second pipeline where the implementer.md Rule 4 ("Verify the blueprint's root cause hypothesis") prevented a wasted debug iteration. The rule was added after `2026-03-29-forcestart-migration` and has now proven its value in a different context (test tooling metadata vs. runtime behavior). Confirms the rule is broadly applicable.

**No generic role file changes.** Pipeline was clean. Existing rules worked as intended.

### 2026-03-31 -- 2026-03-31-sound-scenario-not-triggered: Clean Pipeline — Guidelines Gap Backfill

**Context:** Zero-debug pipeline (2 roles, 0 failures). Debugger identified movement gate as root cause for ForceAttack not firing in sound scenarios. Implementer wired 12 sound events across 3 monster base classes using behaviorStartSound pattern + direct SoundRegistry.Play() calls. Blueprint flagged 3 Guidelines Gaps.

**Changes applied:**
- `Company/project/project-guidelines.md`: Added "ForceAttack Requires Movement Gate Clearing" section — documents the movement gate gotcha for any code that creates ForceFireBehaviour outside PlayMode tests. Reason: Debugger identified this as a general pattern; SoundTestInitializer was the second independent codebase to hit it (after the PlayMode test infrastructure).
- `Company/project/project-guidelines.md`: Added "behaviorStartSound Pattern" section — documents how to wire walk/behavior-start sounds on monster behaviors, lists which base classes have which sounds wired. Reason: Blueprint flagged as Guidelines Gap; prevents future agents from re-discovering the pattern.
- `Company/project/project-guidelines.md`: Added "Counter Cooldown Persistence in Sound Scenarios" section — documents counter cooldown persistence and Counter_Fail dual trigger points. Reason: Blueprint flagged as Guidelines Gap.

**No generic role file changes.** Both workers followed their role definitions correctly. Existing rules were sufficient.
