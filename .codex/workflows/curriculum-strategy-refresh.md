# Curriculum Strategy Refresh Workflow

Use this workflow when you want to refresh the macro curriculum plan before file-level rewrite work continues.

This workflow is for strategy maintenance, not lesson rewriting.

## Purpose

Run a director pass over the curriculum's planning surface, then require an assistant-director review before the refreshed plan becomes active.

The foreground agent for this workflow should manage the interaction and dispatch behavior, not replace the macro specialists.

- The foreground agent should load the five target files, select the right moment to dispatch, and keep the user informed.
- The foreground agent should dispatch the director for the planning pass and the assistant director for the review pass.
- The foreground agent should summarize outputs, maintain workflow state, and decide whether the approved result should become the active plan.
- The foreground agent should not perform the director's strategy rewrite work or the assistant director's review work itself when those specialist agents are available.
- The foreground agent may run validation and make coordinator-only state updates after the specialist passes complete.

The target files for this workflow are:

- `CURRICULUM_MAP.md`
- `CURRICULUM_STRATEGY.md`
- `LEARNING_PATHS.md`
- `README.md`
- `TODO.md`

## Roles

- `.agents/curriculum-director.agent` owns the planning pass and may update the target files above.
- `.agents/curriculum-assistant-director.agent` owns the read-only review pass and must verify the updated strategy before execution resumes.
- The supervisor or coordinator runs this workflow, summarizes the results, and decides whether the approved `TODO.md` should become the new execution plan.

## Director Pass

Dispatch `.agents/curriculum-director.agent` with strict scope limited to the five target files above.

The director should:

1. Review `CURRICULUM_STRATEGY.md` for purpose, learner assumptions, scope tiers, priority clusters, and milestone outcomes.
2. Review `LEARNING_PATHS.md` for sequencing quality and path clarity.
3. Review `CURRICULUM_MAP.md` for stage-to-track mapping and structural completeness.
4. Review `README.md` for public-facing curriculum identity and entry-point guidance.
5. Review `TODO.md` for prioritization quality, metadata quality, and alignment with the approved strategy artifacts.
6. Update any of those files when the improvements are justified by the repo and the automation-first learner goal.
7. Keep edits compact and strategy-focused. Do not rewrite lesson content through this workflow.

The director should return:

1. A short strategy summary.
2. The files changed.
3. Priority changes made to `TODO.md`.
4. Assumptions or risks the assistant director should inspect closely.

## Assistant Director Review Pass

After the director finishes, dispatch `.agents/curriculum-assistant-director.agent` against the same five files and any explicit move plan.

The assistant director should review the updated plan as one bundle and return:

1. `PASS` or `NEEDS_FIX`.
2. Blocking findings first.
3. Strategy artifact findings.
4. Unsupported or weak repo-evidence claims.
5. Sequencing or prerequisite problems.
6. Path, naming, or structural churn risks.
7. `TODO.md` metadata or prioritization problems.
8. Residual risks.

If the review returns `NEEDS_FIX`, send only those findings back to the director and rerun the same review workflow.

## Approval Rule

Do not treat the refreshed strategy as active until the assistant director returns `PASS`.

After approval:

1. Treat `CURRICULUM_STRATEGY.md`, `LEARNING_PATHS.md`, `CURRICULUM_MAP.md`, and `README.md` as the approved macro surface.
2. Treat the updated `TODO.md` as the approved execution queue.
3. Resume or start the file-level workflow in `.codex/workflows/curriculum-audit-rewrite.md`.

## Context Hygiene

Keep this workflow narrow.

- Load only the five strategy files, the two macro agent contracts, and any move plan under review.
- Do not pull lesson content unless a strategic claim cannot be verified without one nearby example.
- Summarize director output before handing it to the assistant director when the pass is verbose.
- Discard old strategy drafts after a new assistant-director-approved version exists.

## Validation

After an approved pass:

1. Verify the touched strategy files have no markdown or agent-file errors.
2. Confirm `TODO.md` still preserves the expected operational fields and status values.
3. Confirm the updated strategy still points the execution workflow toward a bounded next batch rather than a broad backlog.

## Suggested Dispatch Prompt For Director

```text
Refresh the curriculum strategy for this repository.
Review and update only these files:
- CURRICULUM_MAP.md
- CURRICULUM_STRATEGY.md
- LEARNING_PATHS.md
- README.md
- TODO.md

Optimize for the automation-first learner.
Keep the output compact and strategy-focused.
Do not rewrite lesson content.
Return the strategy summary, files changed, TODO priority changes, and risks for assistant-director review.
```

## Suggested Dispatch Prompt For Assistant Director

```text
Review the refreshed curriculum strategy bundle for this repository.
Review these files together:
- CURRICULUM_MAP.md
- CURRICULUM_STRATEGY.md
- LEARNING_PATHS.md
- README.md
- TODO.md

Return PASS or NEEDS_FIX.
Check repo evidence, sequencing quality, path clarity, naming, structural churn, and TODO prioritization.
Blocking findings first.
```
