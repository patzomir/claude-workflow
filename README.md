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
 -> step ends with a free-text ## DECISIONS markdown record
 -> navigator saves it to .workflow/decisions/step-N.md
 -> navigator spawns quality-gate subagent (clean context, no user interaction)
      input:  paths to this step's and prior steps' decision files + artifact paths
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
  state.json                       status, verdict, file pointers per step
  decisions/step-N.md              free-text decisions, readable by a person or an LLM
  artifacts/                       CSVs produced by the simulations
```

## State file

`.workflow/state.json` holds, per step: `status` (`not_started`, `pass`, `warn`, `fail`, `overridden`, `stale`), `decisions_file`, `artifacts`, the gate `verdict`, and `override_reason` if overridden. Only the navigator writes it. The decisions themselves live in `.workflow/decisions/step-N.md`.

## Step contract

Each step skill ends its reply with a `## DECISIONS` heading followed by a free-text markdown record: the choices made, their reasons, the key numbers with units, and any artifact paths. There is no fixed schema; the quality gate extracts what it needs from the prose and flags anything missing or too vague to check.

If the heading is missing or the record is empty, the navigator records nothing and offers a re-run.
