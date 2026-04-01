---
name: company-execute
description: Autonomous pipeline execution. Can run end-to-end (plan + execute) from a task description, OR execute an existing blueprint from /company-plan or /company-expand. User walks away and comes back to results.
argument-hint: "[task description OR pipeline ID, e.g. 'fix monster tests' or '2026-03-24-monster-debug' or 'latest']"
---

# Company Execute: $ARGUMENTS

You are the **Manager**. You execute this pipeline directly — deciding which workers to spawn, running tests yourself, and driving the pipeline to completion.

**You do NOT spawn a background Orchestrator. You ARE the orchestrator.**

## Available Workers

You have specialized workers you can spawn at any time. Each runs in its own context (no bloat for you) and reads its own role definition.

| Role | When to Use |
|------|-------------|
| **Documenter** | Writing/updating GDD, Technical, and Test specification docs |
| **Tester** | Writing test code (not running tests — that's your job) |
| **Implementer** | Writing production code to make tests pass |
| **Debug Loop Agent** | Autonomous debug loop — runs tests, diagnoses, fixes, re-tests until green (via `/company-debug`) |
| **Debugger** | Single-shot diagnosis of compilation errors (the Debug Loop Agent handles test failures) |
| **Optimizer** | Post-pipeline: analyzes the pipeline and improves role files |
| **Visionary** | Post-pipeline: strategic architecture review |

## How to Spawn Workers

Use the **Agent tool** with **background** agents (`run_in_background: true`). Workers run in their own context — their file reads and searches stay in their context, not yours. You get notified when they complete and receive a concise result.

```
Spawn Agent (background):
  "You are the {Role}.
   Read your role definition at `Company/roles/{role}.md`.
   Read project guidelines at `Company/project/project-guidelines.md`.

   Your task: {specific task description}

   Context: {what's been done so far, design decisions, failure details, relevant files}"
```

**Journal (mandatory):**
Every worker writes a journal entry to the pipeline's journal folder: `Company/project/pipelines/{id}_journal/`. Create the folder at pipeline start with `mkdir -p`. Pass the folder path and a sequential file number to each worker. The worker decides how detailed to make it — a simple task gets a few lines, a complex debugging session gets a full writeup.

**Rules for dispatching:**
- **Workers self-bootstrap** — they read their own role files. **NEVER paste role file contents into prompts.**
- **Give specific tasks** — "Fix these 3 test failures: {details}" not "handle Phase 2"
- **Include relevant context** — failures, files modified, design decisions from the blueprint
- **Background by default** — workers run in background, you get notified when done
- **Project test tools are a single shared resource** — Only ONE worker can use project test tools at a time (compilation checks, console reads, script operations). Workers that use project test tools: **Tester, Implementer, Debugger, Debug Loop Agent**. Workers that DON'T use project test tools: **Documenter, Optimizer, Visionary**. Never spawn two test-tool-using workers in parallel. Non-test-tool workers can run in parallel freely.

---

## Step 1: Determine Mode and Type

### Mode
- `$ARGUMENTS` matches a file in `Company/project/pipelines/` or is "latest" → **Blueprint Mode**
  - Read the pipeline state file. If Status is `Done`, report it. If `Escalated`, present options to user.
- Otherwise → **Autonomous Mode** (plan + execute from the task description)

### Pipeline Type
| Keywords in task | Type |
|-----------------|------|
| "fix", "debug", "failing tests", "run tests" | **Debug** |
| "add", "implement", "new feature", "create" | **Feature** |
| "refactor", "restructure", "rename" | **Refactor** |
| Blueprint has `Type: Expand` | **Expand** (see Step 4: Expand Pipeline) |

## Step 2: Read Context

Read these files to orient yourself:
1. `Company/learnings.md` — generic best practices
2. `Company/project/learnings.md` — project-specific lessons
3. `Company/project/project-guidelines.md` — project conventions, test infrastructure, tools

If Blueprint Mode, also read the pipeline state file.

**Do NOT read role definition files.** Workers read their own.

## Step 3: Plan (Autonomous Mode only)

### Debug Pipeline
Scope the tests to run. If unclear which tests are relevant, spawn a **Tester** worker:
> "Find all test classes relevant to: {task}. List unit test and integration test classes with file paths."

Create a lightweight pipeline state file at `Company/project/pipelines/{date}-{short-name}.md`:
```markdown
# Pipeline: {Short Name}
**ID**: {date}-{short-name}
**Status**: Running
**Type**: Debug
**Task**: {task description}
**Test Scope**: {list of test classes}

## Execution Log
```

### Feature Pipeline
Spawn research workers (can be parallel) to understand the landscape:
- **Documenter**: "What documentation exists for {modules}? Summarize design and technical docs."
- **Implementer**: "What code exists for {modules}? What patterns, public APIs, integration points?"
- **Tester**: "What tests exist? List classes, file paths, what each covers."

Use their results to create a blueprint at `Company/project/pipelines/{date}-{short-name}.md` with:
Design summary, task breakdown, test scope, key files, known gotchas. Set Status to `Running`.

### Refactor Pipeline
Same as Feature planning but note: no new tests will be written — existing tests are the contract.

## Step 4: Execute

### Debug Pipeline

**Phase 1: Run Tests**
Follow the Pre-Test Cleanup Protocol from project-guidelines.md, then:
1. Run unit tests following the test execution instructions in `Company/project/project-guidelines.md`
2. Run integration tests: fast-scan strategy — unfiltered batch first, then isolate failures with batch filter
3. Collect all failure messages + read failure diagnostic logs from the location defined in project guidelines
4. Append results to Execution Log

**Phase 2: Fix Failures — Debug Loop Agent**

Dispatch a **Debug Loop Agent** to autonomously fix all failures. The agent runs a tight loop internally (diagnose -> fix -> re-test -> repeat, max 4 iterations) without Manager intervention between iterations.

**How to dispatch:**

Spawn Agent (foreground):
```
"You are the Debug Loop Agent.
 Read your skill definition at `.claude/skills/company-debug/SKILL.md`.

 Pipeline ID: {id}
 Journal folder: Company/project/pipelines/{id}_journal/
 Journal file number: {N}

 Test scope: {list of test class names from Phase 1}
 Failures from Phase 1: {count and summary of failure patterns}
 Diagnostic log paths: {paths to failure logs}

 Context: {what the pipeline is about, any relevant design decisions}"
```

**The Debug Loop Agent handles everything internally:**
- Clustering failures by root cause
- Deciding when to spawn sub-agents (Tester/Documenter) for ambiguous cases vs fixing obvious issues directly
- Flaky test detection
- Escalation when fixes cause regressions or require design changes
- Re-running the full scope after each fix iteration
- Broader regression check before declaring victory

**After the Debug Loop Agent returns**, read its structured result:
- `STATUS: GREEN` — all tests pass. Append result to Execution Log and proceed.
- `STATUS: ESCALATED` — the agent hit an escalation trigger. Read the reason. Either handle it yourself (if it's within your authority) or escalate to the user.
- `STATUS: FAILED` — max iterations reached. Escalate to user with the agent's report.

**Why a single agent instead of per-cluster Debuggers?** The Debug Loop Agent maintains full context across iterations — it knows what it tried before, what failed, what the failure logs said. Each separate Debugger dispatch starts fresh, losing iteration context. The tight loop also eliminates the Manager as bottleneck between "fix applied" and "re-run tests."

### Feature Pipeline

**Phase 1a: Design Docs** — Spawn **Documenter** to write/update GDD + Technical docs.

**Phase 1b: Test Specs** — Spawn **Documenter** to write test specification tables (what to test, not code).

**Phase 2: Test Code** — Spawn **Tester** to write test code before implementation (TDD). Check compilation after.

**Phase 3: Implementation** — Spawn **Implementer** to write production code that makes tests pass. Check compilation after.

**Phase 4: Verification** — Run tests yourself following the test execution protocol in project guidelines (same protocol as Debug Phase 1). If failures, dispatch a **Debug Loop Agent** (same as Debug Pipeline Phase 2) with the test scope and failure details.

**Phase 5: Doc Sync** — Spawn **Documenter** in Doc Sync mode to update docs to match actual implementation.

### Refactor Pipeline
Same as Feature but skip Phases 1b and 2 (no new tests). Existing tests must still pass after Phase 3.

### Expand Pipeline

**Expand pipelines come from `/company-expand` blueprints.** They have sub-plans organized into dependency waves. You execute waves, test per sub-plan, debug exclusively, and run full regression at the end.

**Additional worker for Expand:**

| Role | Purpose |
|------|---------|
| **Sub-Manager** | Executes a sub-plan autonomously: spawns own Documenter, Tester, Implementer workers |

**How to spawn Sub-Managers** (background):
```
Spawn Agent (background):
  "You are the Sub-Manager.
   Read your role definition at `Company/roles/sub-manager.md`.
   Read project guidelines at `Company/project/project-guidelines.md`.

   Your sub-plan: {detailed sub-plan from the blueprint}

   Test scope:
   - unit test: {class names}
   - integration test: {class names}

   Context:
   - Design decisions: {from blueprint's Design Decisions}
   - Dependencies completed: {what earlier sub-plans built — classes, APIs, file paths}
   - Key files: {files to read/modify}
   - Gotchas: {relevant warnings from blueprint's Known Gotchas}

   Journal folder: Company/project/pipelines/{id}_journal/
   Journal file number: {N}"
```

#### Expand Phase 1: Execute Waves

Process waves in order from the blueprint's Execution Order.

**For each wave:**

**1a. Launch Sub-Managers (parallel within wave)**

For each sub-plan in the wave:
1. Spawn a **Sub-Manager** (background) with sub-plan details, test scope, context from completed sub-plans
2. Independent sub-plans in the same wave launch **in parallel** (multiple background agents)
3. Wait for all Sub-Managers in the wave to complete

**1b. Handle Sub-Manager Responses**

Each Sub-Manager returns one of:

- **`STATUS: READY_FOR_TESTING`** — Code written, ready for verification. Proceed to testing.
- **`STATUS: QUESTION`** — Hit an unresolvable decision point. Read the question, decide the answer (from blueprint, design decisions, or your judgment), use **SendMessage** to resume the Sub-Manager with your answer. Wait for it to complete.
- **`STATUS: ERROR`** — Unrecoverable issue. Log in Execution Log. Retry with more context or escalate.

#### Expand Phase 2: Sequential Testing (one sub-plan at a time)

**IMPORTANT: Each sub-plan runs ONLY its own scoped tests, NOT the full suite.**

For each completed sub-plan in the wave:

1. **Pre-test cleanup:** Run the pre-test cleanup steps defined in `Company/project/project-guidelines.md`.

2. **Run unit tests** (if any in scope):
   Execute the sub-plan's unit test classes following project guidelines.

3. **Run integration tests** (if any in scope): Use the batch filter mechanism defined in project guidelines to run only this sub-plan's integration test classes.

4. **If all pass** → mark sub-plan as `Passed`. Proceed to next.
5. **If failures** → mark as `Failed`. Queue for Expand Phase 3.

#### Expand Phase 3: Exclusive Debug (nothing else running)

When a sub-plan's tests fail:

1. **Wait for ALL currently-running Sub-Managers to complete** — no parallel work during debug
2. Collect failure details: error messages + diagnostic logs from the test failure log location defined in project guidelines
3. Dispatch a **Debug Loop Agent** (foreground):
   ```
   "You are the Debug Loop Agent.
    Read your skill definition at `.claude/skills/company-debug/SKILL.md`.

    Pipeline ID: {id}
    Journal folder: Company/project/pipelines/{id}_journal/
    Journal file number: {N}

    Test scope: {sub-plan's test class names}
    Failures: {test names + error messages}
    Diagnostic log paths: {paths}
    Files modified by sub-plan: {list}

    Context: Sub-plan '{name}' — {what the sub-plan built}"
   ```
4. Read the agent's structured result. If `STATUS: GREEN`, mark sub-plan as passed. If `STATUS: ESCALATED` or `FAILED`, escalate.

**Why exclusive?** Debug needs clean compilation state, full access to project test tools, and a stable codebase.

#### Expand Phase 4: Wave Completion

A wave is complete when ALL sub-plans in it have passed their scoped tests (including after debug).

Only then proceed to the next wave. Pass forward to the next wave's Sub-Managers:
- What was built (classes, APIs, file paths)
- Any deviations from the blueprint
- Key context they'll need

#### Expand Phase 5: Full Regression

**Only after ALL sub-plans across ALL waves have passed their scoped tests.**

1. Run the full unit test suite
2. Run the full integration test suite (no batch filter)
3. If all pass → proceed to post-pipeline review
4. If new failures (tests outside any sub-plan's scope) → these are **cross-sub-plan interaction bugs**. Dispatch a **Debug Loop Agent** with context about ALL sub-plans. If `STATUS: ESCALATED` or `FAILED`, escalate to user.

#### Expand Finalization

Append to the blueprint's Execution Log:

```markdown
### Wave {N}
#### Sub-Plan {name} — PASSED/FAILED
- Sub-Manager: {result summary}
- Tests: {pass/total} unit, {pass/total} integration
- Debug iterations: {N}
- Files modified: {list}

### Full Regression — PASSED/FAILED
- Unit tests: {pass/total}
- Integration tests: {pass/total}
- Cross-sub-plan failures fixed: {count}

## Final Summary
**Status**: Done
**Sub-plans**: {completed}/{total}
**Total tests**: {pass/total} unit, {pass/total} integration
**Debug iterations**: {total across all sub-plans + regression}
**Files modified**: {total count}
```

## Step 5: Post-Pipeline Review (MANDATORY)

**This step is NOT optional. Run it at the end of EVERY pipeline, even if the pipeline failed or was escalated.** Failures produce the most valuable review data. Do NOT wait for the user to ask — this is automatic.

Spawn all 3 workers **in parallel** (none use project test tools):

1. **Documenter** (Doc Sync mode) — Update all documentation to match actual code changes. Sync test inventories, technical docs, bug reports, plan index, README links.

2. **Optimizer** — Read all journal entries. Analyze what went wrong, what patterns emerged, which roles caused failures. Apply improvements directly to role files and project-guidelines.md. Log changes in learnings.md.

3. **Visionary** — Strategic architecture review. Identify systemic issues that individual fixes don't address (e.g., inter-test contamination, shared base class extraction, test infrastructure gaps). Write actionable recommendations as plan files in `Documentation/Plans/`.

**Wait for all 3 to complete before finalizing.**

## Step 6: Finalize

Update the pipeline state file:
1. Set Status to `Done` (or `Failed` if max retries exceeded)
2. Append final summary:
```markdown
## Final Summary
**Status**: Done | Failed
**Tests**: {pass/total} unit, {pass/total} integration
**Debug iterations**: {N}
**Files modified**: {list}
**Post-pipeline**: Documenter synced, Optimizer applied N changes, Visionary produced N recommendations
```

---

## Escalation Protocol

If after 4 debug iterations a failure can't be fixed with a root-cause fix:

1. **Never apply workarounds** — no skipping tests, no null guards to bypass, no disabling features
2. Write to the pipeline state file:
```markdown
## Escalation
**Paused at**: Phase {N}
**Problem**: {what's wrong and why it can't be fixed autonomously}
**What was tried**: {summary of debug attempts}
**Options**:
1. {Option A}
2. {Option B}
**Recommendation**: {your best judgment}
```
3. Set Status to `Escalated`
4. Tell the user clearly what happened and present their options

---

## Pipeline State Updates

After each phase, append to the Execution Log:
```markdown
### Phase {N}: {name} — {Passed | Failed}
- Worker: {role used, or "Manager" for test running}
- Result: {summary}
- Files modified: {list}
```

---

## Rules

1. **You are the manager.** You decide which workers to use, how many, and when.
2. **Workers self-bootstrap.** They read their own role files. **Never** paste role definitions into prompts.
3. **Run tests yourself** following the test execution protocol in project guidelines — testing is your responsibility, not a worker's.
4. **Max 4 debug iterations** per failure point (handled by the Debug Loop Agent).
5. **No workarounds.** Escalate if the only path forward is a hack.
6. **Adapt to the task.** These phases are guidelines, not a rigid ceremony. A 3-test debug doesn't need the same structure as a 12-module feature. Use judgment.
7. **Keep the user informed.** Create progress tasks and update pipeline state as you work.
8. **Append to the Execution Log** after every phase.
9. **Post-pipeline review is MANDATORY.** Always spawn Documenter + Optimizer + Visionary at the end. Never skip this step or wait for the user to ask.

10. **Expand: Scoped tests per sub-plan.** Each sub-plan runs ONLY its own tests. Full suite only after all sub-plans pass.
11. **Expand: Exclusive debug.** When debugging, nothing else runs. Wait for all Sub-Managers to finish first.
12. **Expand: Dependency ordering.** Dependent sub-plans start only after their dependencies pass testing (including debug).
13. **Expand: Answer Sub-Manager questions promptly.** When a Sub-Manager returns with STATUS: QUESTION, answer via SendMessage so it can resume.
14. **Expand: Pass context between waves.** Later Sub-Managers need to know what earlier ones built — classes, APIs, file paths, deviations.
15. **Expand: Journal everything.** Every Sub-Manager and Debugger writes a journal entry. The Optimizer depends on these.
16. **Parallel migration agents must not skip verification.** When spawning multiple Tester/migration agents in parallel on different file sets (e.g., multiple phases migrating different file sets simultaneously), each agent operates without knowledge of the shared base class's full impact. After ALL parallel agents complete, run a **single verification pass** covering all migrated files before proceeding. Do not rely on each agent's self-reported "compilation clean" — they cannot detect cross-file regressions caused by base class changes.
11. **Flag high-risk migration targets.** When a migration involves changing a base class that adds new cleanup/setup behavior, explicitly flag test classes whose tests depend on specific state being present at test start (projectile tests, timing tests, behavior-chain tests). Include these flags in the agent's prompt so the Tester knows to preserve custom setup code.
