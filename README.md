# Multi-Agent Company

A structured multi-agent pipeline framework for [Claude Code](https://claude.ai/code).
Specialized AI roles collaborate through documentation-first workflows to plan, implement,
test, and review software -- with a self-improving learning system that gets better after every run.

## Why This Exists

AI coding assistants are powerful, but without structure they operate like a brilliant developer who never reads the existing docs, doesn't follow the team's conventions, and forgets everything between sessions. The result: inconsistent code, tests that don't catch real bugs, and the same mistakes repeated across every conversation.

The Multi-Agent Company gives Claude Code a team of specialized roles that follow a **Doc-Test-Code** lifecycle. Instead of one agent doing everything, a Documenter designs the feature, a Tester writes tests from the design, an Implementer writes code to pass the tests, and a Debugger fixes what breaks. Each role has a focused definition with rules distilled from real pipeline failures.

The key differentiator is the **learning loop**. After every pipeline, an Optimizer analyzes what went wrong, traces failures back to the responsible role, and edits that role's definition to prevent recurrence. A Visionary reviews the architecture and spots systemic patterns. Generic lessons go into shared role files that ship with the template. Project-specific lessons stay in your project's own learnings file. The result: increasingly effective autonomous development over time -- the system literally rewrites itself to avoid past mistakes.

## What You Get

### 8 Specialized Roles

| Role | Responsibility |
|------|---------------|
| **Documenter** | Writes design docs, technical docs, and test specifications before code exists |
| **Tester** | Writes test code from test specifications -- pessimistic tests that fail unless everything works |
| **Implementer** | Writes production code to make tests pass, following existing patterns and conventions |
| **Debugger** | Diagnoses and fixes failures by cross-referencing tests, docs, and code |
| **Task Manager** | Scoped planner -- decomposes a module's work into phases for a parent manager |
| **Sub-Manager** | Executes a sub-plan autonomously -- spawns own workers, manages own phases |
| **Optimizer** | Post-pipeline analyst -- traces failures to their source role, edits role files to prevent recurrence |
| **Visionary** | Strategic reviewer -- spots architectural patterns, recommends systemic improvements |

### 6 Pipeline Commands

| Command | Purpose |
|---------|---------|
| `/company-startup` | First-time setup -- scans your project and generates configuration |
| `/company-integrate` | Bring an existing feature into the Doc-Test-Code lifecycle |
| `/company-plan` | Interactive planning session -- research, design questions, produces a blueprint |
| `/company-execute` | Autonomous pipeline execution -- runs a blueprint or plans and executes from a description |
| `/company-expand` | Plan large, multi-module missions with dependency waves and sub-plans |
| `/company-fast` | Quick single-pass for small tasks -- no blueprint, no journal, no post-review |

### The Learning Loop

Every pipeline produces journal entries from each role that participated. After execution completes, the Optimizer reads every journal entry and the execution log, then:

1. **Traces failures to their source.** If a test broke because the Implementer missed an edge case, the Optimizer adds a rule to `implementer.md`. If the Tester wrote a flaky assertion, the rule goes into `tester.md`.
2. **Separates generic from project-specific.** A lesson like "derive test thresholds from algorithm guarantees, not round numbers" goes into the shared Tester role file (it applies to any project). A lesson like "clear player invulnerability in WaitUntil loops" goes into your project's guidelines (it is codebase-specific).
3. **Edits role files directly.** Small improvements (a new rule, a warning, a gotcha) are applied immediately. Significant changes (restructuring a role, changing phase order) are flagged for user review.

The Visionary operates in parallel, looking for patterns across pipelines: recurring failure classes, growing modules that need splitting, algorithms delivered without integration, test infrastructure gaps. Recommendations are prioritized and actionable.

The net effect: each pipeline makes the next one better. Early pipelines may need multiple debug iterations. After 10-20 pipelines, the role files contain enough hard-won rules that zero-debug runs become common.

### Documentation-First Philosophy

The Company enforces a strict **Doc-Test-Code** lifecycle:

1. **Document the design first.** Before any code exists, a Documenter writes what the feature does, its edge cases, and its integration points. This forces design thinking before implementation.
2. **Write tests from the documentation.** A Tester reads the design doc and writes test code that verifies every documented behavior. The tests will not compile yet -- that is expected.
3. **Implement to make tests pass.** An Implementer writes production code guided by the technical doc and the failing tests. When all tests pass, the feature is complete.
4. **Sync documentation after.** A final Documenter pass updates docs to match what was actually built, catching any deviations between plan and reality.

This workflow prevents the most common AI coding failure mode: writing code that "works" but has no specification, no tests, and no documentation -- making it impossible to verify correctness or safely modify later.

## Getting Started

### 1. Set Up the Template

Copy the Company infrastructure into your project:

```bash
# Clone the template repo
git clone https://github.com/yourorg/multi-agent-company.git

# Copy the Company directory and skills into your project
cp -r multi-agent-company/Company/ /path/to/your/project/Company/
cp -r multi-agent-company/.claude/skills/company-*/ /path/to/your/project/.claude/skills/
```

Or fork this repo as your project's starting point.

### 2. Configure for Your Project

```
/company-startup [your project name]
```

The startup wizard:
- Scans your existing codebase and documentation
- Identifies your language, framework, test infrastructure, and build system
- Asks targeted questions about conventions that could not be inferred from files alone
- Generates `Company/project/project-guidelines.md` -- the single source of truth for all agents
- Creates the folder structure for project-specific learnings, recommendations, and pipelines

Review the generated `project-guidelines.md` carefully. Every agent reads it. If something is wrong, every agent will make the same mistake.

### 3. Integrate Existing Features

```
/company-integrate [feature name]
```

For features built before the Company was set up. This workflow:
- Reads and understands the existing code thoroughly
- Writes design and technical documentation that describes what the code *actually does*
- Asks you to review the documentation and resolve ambiguities
- Writes test specifications and test code based on the approved documentation
- Runs the tests against the existing code
- Fixes only what the tests reveal as broken

### 4. Plan New Work

```
/company-plan [task description]
```

An interactive planning session where the TopManager:
- Spawns research agents (Documenter, Implementer, Tester) to understand the codebase
- Asks you substantive design questions informed by the research findings
- Decomposes the task into phases with clear dependencies
- Produces a blueprint saved to `Company/project/pipelines/`

### 5. Execute

```
/company-execute [pipeline-id or task description]
```

Autonomous execution. The Manager reads the blueprint (or plans from scratch), spawns workers for each phase, runs tests, dispatches Debuggers for failures, and drives the pipeline to completion. Afterwards, the Optimizer and Visionary automatically review the pipeline and improve the system.

You walk away and come back to results.

## Directory Structure

```
Company/
  roles/                    # 8 generic role definitions (improved by Optimizer after each pipeline)
    debugger.md
    documenter.md
    implementer.md
    optimizer.md
    sub-manager.md
    task-manager.md
    tester.md
    visionary.md
  learnings.md              # Generic pipeline best practices (ships with template, grows over time)
  recommendations.md        # Generic strategic patterns (ships with template, grows over time)
  project/                  # YOUR PROJECT-SPECIFIC content (generated by /company-startup)
    project-guidelines.md   # The key file -- project conventions, tools, patterns, gotchas
    learnings.md            # Your project's pipeline lessons (added by Optimizer)
    recommendations.md      # Your project's strategic insights (added by Visionary)
    pipelines/              # Pipeline state files and journals from your runs

.claude/skills/
  company-startup/          # First-time setup wizard
  company-integrate/        # Feature integration into Doc-Test-Code lifecycle
  company-plan/             # Interactive planning
  company-execute/          # Autonomous execution
  company-expand/           # Large mission planning with sub-plans
  company-fast/             # Quick single-pass execution
```

## How It Works

### Pipeline Lifecycle

A typical feature pipeline flows like this:

1. **User invokes** `/company-plan` with a task description
2. **TopManager reads** project guidelines, learnings, and recommendations
3. **Research agents** (Documenter, Tester, Implementer) analyze the codebase in parallel
4. **TopManager asks** the user design questions informed by agent findings
5. **Blueprint produced** and saved to `Company/project/pipelines/`
6. **User runs** `/company-execute [pipeline-id]` to launch the pipeline
7. **Phase 1**: Documenter writes design docs and test specifications
8. **Phase 2**: Tester writes test code (TDD -- tests before implementation)
9. **Phase 3**: Implementer writes production code to make tests pass
10. **Phase 4**: Manager runs tests; Debugger fixes any failures (max 3 iterations)
11. **Phase 5**: Documenter syncs documentation with actual implementation
12. **Phase 6**: Optimizer + Visionary review the pipeline and improve the system

Each worker writes a journal entry describing what it did, decisions made, and problems encountered. The Optimizer reads these journals to trace failures back to their source.

### The Role System

Roles are **generic and reusable** across any project. They contain rules like "never hardcode configurable values in tests" and "trace failures to the responsible role" -- lessons that apply regardless of the codebase.

**Project-specific knowledge** lives in `Company/project/project-guidelines.md`, not in the role files. This file contains your project's error handling conventions, test infrastructure details, compilation commands, architecture patterns, and hard-won gotchas.

The Optimizer improves role files after every pipeline, but only with generic lessons. Project-specific lessons go to `Company/project/learnings.md`. This separation ensures the roles remain portable -- you can copy the `Company/roles/` directory into a new project and all the accumulated wisdom carries over.

### Three Pipeline Tiers

| Tier | Command | Use When |
|------|---------|----------|
| **Fast** | `/company-fast` | Small, quick tasks (bug fix, doc update, small refactor). No blueprint, no journal, no post-review. |
| **Standard** | `/company-plan` + `/company-execute` | Normal features. Full Doc-Test-Code lifecycle with blueprint and post-pipeline review. |
| **Expand** | `/company-expand` + `/company-execute` | Large multi-module work. Decomposes into sub-plans with dependency waves. Includes full regression testing. |

#### When to Use Expand

If a task has 3+ independent sub-pipelines, touches 3+ modules, or requires 5+ new files, consider `/company-expand`. It adds:
- **Sub-Managers** that execute sub-plans autonomously with their own workers
- **Dependency waves** -- independent sub-plans run in parallel, dependent ones wait
- **Full regression testing** after all sub-plans complete, catching cross-module interaction bugs

## Examples

See `examples/evil-magicians/` for a real-world configuration from a Unity game project that has run 20+ pipelines. It includes:
- A battle-tested `project-guidelines.md` with 15+ sections accumulated over 22 pipelines
- Project-specific learnings showing the Optimizer's feedback loop in action
- Curated Visionary recommendations covering architecture, process, tooling, and conventions
- Two complete pipeline examples (Feature and Debug) with journal entries from multiple roles

## Philosophy

### Pessimistic Testing

Tests must fail unless everything works. A passing test that does not catch a real bug is worse than no test at all. The Tester role enforces: assert preconditions before the action, force the scenario so it must trigger, no conditional assertions, assert exact values when known, verify intermediate steps in behavior tests, and assert what should not happen.

### No Silent Failures

Errors must be visible. No swallowed exceptions, no defensive null checks that hide bugs, no fallbacks that mask missing configuration. If something is wrong, it must crash loudly so it gets fixed. The Implementer role enforces this as a hard rule -- fallbacks that look "safe" create bugs that are impossible to debug because the system silently degrades.

### Self-Improvement

Every pipeline makes the system better. The Optimizer traces failures to their source role and adds rules to prevent recurrence. The Visionary spots patterns across pipelines and recommends architectural improvements. Generic lessons go into the shared role files (portable to any project); project-specific lessons stay in your project's learnings. After 10-20 pipelines, the role files contain enough rules that common failure modes are eliminated before they occur.

### Root Cause Analysis

When something breaks, fix the actual problem -- not the symptom. The Debugger role cross-references three sources (test code, test documentation, design documentation) to determine whether a failure is a code bug, a test bug, a stale test, or a documentation gap. Quick patches that hide symptoms are explicitly prohibited.

## Requirements

- [Claude Code](https://claude.ai/code) (CLI or IDE integration)
- A codebase with a test framework (the Company is language-agnostic but needs a way to verify correctness)

## License

MIT
