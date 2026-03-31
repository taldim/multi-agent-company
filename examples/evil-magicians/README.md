# Example: Evil Magicians

## What is Evil Magicians?

Evil Magicians is a Unity 2D multiplayer game -- 4-player couch co-op with magic-based combat, AI-controlled monsters, and clan-based PvP. It is built with Unity 6, C#, and a MonoBehaviour architecture.

## How the Company Has Been Used

The Company multi-agent system has been used on Evil Magicians for approximately **22 pipelines over 1 week** (March 24-30, 2026). These include:

- **Feature pipelines**: Procedural map generation (3 sub-pipelines), player action time windows (6 sub-plans), debug visualization, sound testing framework, arena traversal
- **Debug pipelines**: Monster PlayMode test debug (2 rounds), PlayMode test fixes, inter-test contamination fix, corridor path fixes
- **Infrastructure pipelines**: Test camera infrastructure, test infrastructure cleanup, ForceStart migration

## What This Example Demonstrates

This curated subset shows what a mature, battle-tested Company configuration looks like after extensive real-world use:

| File | What It Shows |
|------|---------------|
| `project-guidelines.md` | A comprehensive project-guidelines file that has grown organically through 22 pipelines. Shows error handling conventions, test infrastructure patterns, common gotchas, and architecture patterns -- all learned through real pipeline failures. |
| `learnings.md` | Project-specific lessons added by the Optimizer after each pipeline. Shows the feedback loop between pipeline execution and knowledge accumulation. |
| `recommendations.md` | Curated Visionary recommendations (15 of ~50 total). Shows the breadth of strategic analysis: architecture, tooling, process, tech debt, and conventions. |
| `pipelines/2026-03-24-monster-test-debug-v2.md` | A Debug pipeline with 44 initial failures. Shows cluster-first diagnosis, root cause analysis, and the Optimizer's post-mortem. |
| `pipelines/2026-03-28-player-action-time-windows.md` | An Expand (Feature) pipeline with 6 sub-plans. Shows the full Doc-Test-Code lifecycle, dormant-by-default design, and Sub-Manager direct-write pattern. |
| `pipelines/*_journal/` | Representative journal entries from Debugger, Documenter, Visionary, Optimizer, and Sub-Manager roles. Shows how each role communicates findings for the Optimizer to analyze. |

## Important Note

This is a **curated subset**. The actual project has:
- ~50 Visionary recommendations (15 shown here)
- 22 pipeline state files (2 shown here)
- ~100 journal entries across all pipelines (10 shown here)
- A 340-line project-guidelines file that has been refined by every pipeline
- Extensive documentation (GDD, Technical, Testing) that the Company maintains

The purpose of this example is to show the **pattern and maturity level**, not every detail. Use it as a reference for what your own Company configuration can grow into.
