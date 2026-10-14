# Model Development Workflow (Claude Code skills)

A sample human-in-the-loop workflow for building a binary classifier, implemented as Claude Code skills. A navigator skill drives six step skills; after each step an independent subagent validates the decisions before you move on.

Data is **simulated** in steps 2–5. The point is the workflow mechanics: decisions, interim results, cutoffs, and validation.

## Quick start

1. Open Claude Code in this directory (restart the session if skills aren't listed).
2. Run `/top-workflow`.
3. Pick a step from the dashboard, answer its questions, read the gate verdict, repeat.

Progress is saved in `.workflow/state.json`, so you can quit and resume at any time.

## Flow

```
/top-workflow (navigator, reads state file)
 -> user picks step N
 -> step-N skill runs inline (simulation + questions to the user)
 -> step emits a ## DECISIONS JSON block
 -> navigator writes decisions to the state file
 -> navigator spawns quality-gate subagent (clean context, no user interaction)
      input:  step decisions + prior steps' context + artifact paths
      checks: completeness, range, consistency, cross-step coherence, domain plausibility
      output: verdict JSON (pass | warn | fail)
 -> navigator shows the verdict
 -> on fail: user chooses re-run or override (with a reason)
 -> back to dashboard
```

## Steps

| # | Skill | What happens | You decide |
|---|---|---|---|
| 1 | `step-1-problem-framing` | Human input only | target definition, primary metric, minimum acceptable value, FP/FN costs, windows |
| 2 | `step-2-data-audit` | Simulated column profile | missing-value threshold, outlier rule, columns to drop |
| 3 | `step-3-feature-selection` | Simulated IV and correlation screen | IV minimum, correlation cutoff, suspiciously strong features |
| 4 | `step-4-model-training` | Simulated CV of LR / RF / GBM | model family, CV folds, hyperparameter |
| 5 | `step-5-cutoff-selection` | Simulated precision/recall/cost grid | selection rule, probability cutoff |
| 6 | `step-6-signoff` | Model card and open risks | go / conditional / no go, monitoring plan |

A step is available once all earlier steps are done (`pass`, `warn` or `overridden`). Re-running a done step marks all later steps `stale`.

## Layout

```
.claude/
  skills/
    top-workflow/SKILL.md          navigator, owns the state file and the loop
    step-1-problem-framing/SKILL.md
    step-2-data-audit/SKILL.md
    step-3-feature-selection/SKILL.md
    step-4-model-training/SKILL.md
    step-5-cutoff-selection/SKILL.md
    step-6-signoff/SKILL.md
  agents/
    quality-gate.md                read-only validator subagent
.workflow/                         created at runtime
  state.json                       decisions, results, verdicts per step
  artifacts/                       CSVs produced by the simulations
```

## State file

`.workflow/state.json` holds, per step: `status` (`not_started`, `pass`, `warn`, `fail`, `overridden`, `stale`), `decisions`, `interim_results`, `artifacts`, the gate `verdict`, and `override_reason` if overridden. Only the navigator writes it.

## Step contract

Each step skill ends its reply with:

````
## DECISIONS
```json
{ "step": N, "decisions": {...}, "interim_results": {...}, "artifacts": [...] }
```
````

If the block is missing or invalid, the navigator records nothing and offers a re-run.

## Quality gate

`quality-gate` has read-only tools (Read, Glob, Grep) and cannot ask questions. It runs five checks:

- **completeness**: required fields present
- **range**: values inside allowed bounds
- **consistency**: values within the step agree (e.g. cost recomputes from confusion counts)
- **cross_step**: agrees with earlier steps (e.g. selected features survived the data audit)
- **plausibility**: would a practitioner believe it (e.g. AUC above 0.95 suggests leakage)

Verdict is `fail` if any finding is `fail`, `warn` if any is `warn`, otherwise `pass`. The per-step rubric lives in [.claude/agents/quality-gate.md](.claude/agents/quality-gate.md).

## Extending

- **Add a step:** create `.claude/skills/step-N-.../SKILL.md` that ends with the `## DECISIONS` block, add it to the state template in `top-workflow`, and add a rubric entry in `quality-gate.md`.
- **Use real data:** replace the "Simulate" section of a step with code that reads your dataset; keep the decisions schema unchanged so the gate still works.
- **Tighten or loosen checks:** edit the thresholds in `quality-gate.md`.

## Reset

Delete `.workflow/` to start over (the navigator asks for confirmation if you request a reset).
