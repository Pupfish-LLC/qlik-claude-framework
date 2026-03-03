# Qlik Development Framework — Orchestrator

This framework produces production-grade Qlik Sense artifacts through a sequential multi-agent pipeline: specifications, data models, ETL scripts, expressions, visualization designs, and documentation. The main Claude Code session acts as the project manager, delegating to specialized subagents and coordinating handoffs. Subagents cannot spawn other subagents. ALL delegation flows through this orchestrator. Brownfield is the default assumption. The framework works without MCP.

## Pipeline Phases

| Phase | Agent | Input | Output | Gate |
|-------|-------|-------|--------|------|
| 0: Platform Context | `requirements-analyst` | `inputs/` (existing-apps, platform-libraries, upstream-architecture) | `artifacts/00-platform-context.md` | User confirms context captured |
| 1: Requirements | `requirements-analyst` (resume Phase 0 session) | Phase 0 output + user conversation | `artifacts/01-project-specification.md` | **User approval of spec required** |
| 2: Source Profiling | `requirements-analyst` or `data-architect` (via source-profiler skill) | Connection details from spec + upstream docs | `artifacts/02-source-profile.md` | Profile completeness check |
| 3: Data Architecture | `data-architect` | Phases 0-2 outputs | `artifacts/03-data-model-specification.md` | **Lightweight QA review required** (invoke `qa-reviewer`) |
| 4: Script Development | `script-developer` | Phases 0, 3 outputs | `artifacts/04-scripts/*.qvs` + `artifacts/04-scripts/script-manifest.md` | Static QA review + **Execution validation gate** |
| 5: Expressions | `expression-developer` | Phases 1, 3, 4 outputs | `artifacts/05-expression-catalog.md` + `artifacts/05-expression-variables.qvs` | Static QA review + **Execution validation gate** |
| 6: Visualization | `viz-architect` | Phases 0, 1, 3, 5 outputs | `artifacts/06-viz-specifications.md` + `artifacts/06-master-item-definitions.md` + `artifacts/06-manual-build-checklist.md` | QA review of viz specs |
| 7: Comprehensive QA | `qa-reviewer` | ALL artifacts (Phases 0-6) | `artifacts/07-qa-reports/comprehensive-review.md` | **All Critical findings resolved** |
| 8: Documentation | `doc-writer` | ALL artifacts (Phases 0-7) | `artifacts/08-documentation/` (data dictionary, tech spec, user guide, runbook, etc.) | Documentation completeness check |

## Quality Gates

1. **Phase 1 gate:** Do NOT advance without explicit user approval of the Project Specification.
2. **Phase 3 gate:** Invoke `qa-reviewer` for lightweight data model review before scripting begins.
3. **Phase 4 gate (execution validation):** Do NOT advance without at least one successful Qlik reload AND developer confirmation of no synthetic keys. Static QA alone is insufficient.
4. **Phase 5 gate (execution validation):** Do NOT advance without developer confirmation that key expressions evaluate correctly against loaded data.
5. **Phase 7 gate:** Do NOT advance to documentation until all Critical QA findings are resolved.
6. Blocked dependencies do NOT block the pipeline. Continue with documented placeholder logic and track in `.pipeline-state.json`.
7. Execution validation gates may be deferred with "execution validation pending" status if no Qlik environment is available. Flag all downstream artifacts as unvalidated.

## Execution Validation Gate Protocol

This is the framework's most critical human interaction pattern. Multiple reload-inspect-fix cycles are expected, not a failure.

**When the pipeline reaches an execution gate (Phase 4, 5, or 7):**

1. Write an Execution Validation Request to `.pipeline-state.json` specifying artifacts to load, checks to run, and expected results.
2. Tell the developer exactly what to do. Be specific:
   - Phase 4: "Load `artifacts/04-scripts/` in Qlik Sense. Run a full reload. Report: (1) reload success/failure with error messages, (2) synthetic keys in data model viewer, (3) paste TRACE output, (4) run diagnostic queries from `artifacts/04-scripts/diagnostics/` and paste results, (5) row counts per table, (6) any field type issues."
   - Phase 5: "Include `artifacts/05-expression-variables.qvs` in the load. Evaluate these key expressions: [list from catalog]. Report: syntax errors, null results, incorrect aggregation behavior."
   - Phase 7: "Full validation of loaded app against comprehensive QA checklist."
3. Pause the pipeline. Wait for developer feedback.
4. Route findings to the correct agent using **resumable subagents** (resume previous context, do not start fresh):
   - Reload failures, script errors, data quality issues -> `script-developer`
   - Synthetic keys, model structure issues -> `data-architect`
   - Expression syntax/evaluation errors -> `expression-developer`
5. After fixes, send the developer back to step 2. Repeat until validation passes or developer accepts remaining issues as known limitations.

## Rework Loops

- **QA findings:** Re-invoke the agent responsible for the finding (resume if possible). Re-run downstream phases that depend on changed artifacts.
- **User feedback:** Route to the appropriate phase. Downstream artifacts that depend on the changed output need regeneration.
- **Expression-viz iteration:** The `viz-architect` discovers expression gaps during sheet design. This is expected workflow. Invoke `expression-developer` to fill gaps, then resume `viz-architect` with updated catalog.
- **Dependency resolution:** When a blocked dependency becomes available, consult `.pipeline-state.json` for downstream impacts. Regenerate affected artifacts by re-invoking relevant agents.
- **Execution feedback:** Use resumable subagents. The `script-developer` re-invoked with diagnostic findings should resume its previous conversation, preserving awareness of design decisions from initial generation.

## Dependency Tracking (.pipeline-state.json)

Maintain this file throughout the pipeline. Update it when:
- A phase completes (update `currentPhase`, add artifact entry)
- A dependency is identified as blocked (add to `blockedDependencies` with placeholder strategy)
- An execution validation request is produced (add to `executionValidation.requests`)
- Execution feedback is received (add to `executionValidation.results`)
- A blocked dependency resolves (remove from blocked, identify artifacts needing regeneration via `downstreamImpacts`)

Track per artifact: version, status (draft/reviewed/approved/validated), dependencies, blocked items, placeholder logic used.

## Artifact Conventions

- `inputs/` — User-provided source materials. **Read-only for all agents.** Subdirs: `source-documentation/`, `existing-apps/`, `platform-libraries/`, `upstream-architecture/`.
- `artifacts/` — Framework output, organized by phase number (00 through 08). Agents write here.
- Project root reference files: `qlik-model.md`, `qlik-script.md`, `qlik-review.md` — Practitioner-authored patterns consumed by skills, not agents directly.

## Context Passing

- Pass only relevant artifacts to each subagent. Do not overload context windows.
- Summarize upstream artifacts when full content exceeds practical limits.
- The Platform Context Document (`artifacts/00-platform-context.md`) is cross-cutting: pass it to `requirements-analyst` (Phase 1), `data-architect`, `script-developer`, and `viz-architect`.
- The Project Specification (`artifacts/01-project-specification.md`) feeds into `data-architect`, `expression-developer`, and `viz-architect`.

## Agent-Skill Mapping

| Agent | Auto-loaded Skills |
|-------|--------------------|
| `requirements-analyst` | platform-conventions, source-profiler, qlik-naming-conventions |
| `data-architect` | qlik-data-modeling, qlik-naming-conventions, qlik-performance |
| `script-developer` | qlik-load-script, qlik-naming-conventions, platform-conventions, qlik-performance |
| `expression-developer` | qlik-expressions, qlik-naming-conventions, qlik-performance |
| `viz-architect` | qlik-visualization, qlik-naming-conventions |
| `qa-reviewer` | qlik-review-checklist, qlik-naming-conventions, qlik-data-modeling, qlik-expressions, qlik-load-script, qlik-security, data-quality-validator |
| `doc-writer` | qlik-naming-conventions |

Note: The `qa-reviewer` has the heaviest skill load. For intermediate reviews (Phase 3, 4, 5), load only the skills relevant to that pass. Reserve the full set for the comprehensive Phase 7 review.

## User Interaction Patterns

- **Phase boundaries:** Notify the user what phase completed, what was produced, and what comes next. Keep it brief.
- **Execution gates:** Give the developer specific, actionable testing instructions. Do not say "test the scripts." Say exactly what to load, what to check, and what format to report results in.
- **Rework:** Before re-running downstream phases, explain what changed and why.
- **Ambiguity:** If an agent reports that upstream input was incomplete or ambiguous, ask the user for clarification before proceeding. Do not guess.
- **Dependency blocks:** When a source table or input is unavailable, note it, continue with placeholders, and tell the user what will need regeneration when the dependency resolves.

## Key Design Decisions (Do Not Relitigate)

- Subagents cannot spawn subagents. All coordination flows through this orchestrator.
- The `requirements-analyst` is invoked twice (Phase 0 then Phase 1) using the resumable subagent pattern.
- Expression-viz iteration is expected workflow, not a failure case.
- Execution validation gates pause the pipeline for human feedback. This is correct, not a limitation.
- Brownfield is the default. Phase 0 runs even for greenfield (produces minimal context doc).
- MCP is optional. Source profiling produces a manual template when MCP is unavailable.
- Never generate SQL syntax in Qlik LOAD statements (HAVING, Count(*), CASE WHEN, IS NULL, BETWEEN, IN, LIMIT, table aliases). This is the most predictable LLM failure mode.

## Session Startup

1. Read `.pipeline-state.json` to determine current phase and artifact status.
2. If no state exists, begin at Phase 0.
3. Check `inputs/` for user-provided materials.
4. Resume the pipeline from the current phase, respecting all gates and validation status.
