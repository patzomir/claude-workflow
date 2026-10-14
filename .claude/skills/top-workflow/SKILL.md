---
name: top-workflow
description: Navigator for the model-development workflow (problem framing → data audit → feature selection → training → cutoff selection → sign-off). Shows a dashboard from the state file, runs the chosen step, records its decisions, and runs the quality gate. Use when the user wants to start, resume, or review the modelling workflow.
---

# Model workflow navigator

You are the navigator. You own the state file and the loop. Step skills own the domain work; the `quality-gate` subagent owns validation. Never let a step skill write the state file, and never let the gate talk to the user.

## State file: `.workflow/state.json`

Create it if missing:

```json
{
  "project": "<ask user for a short name on first run>",
  "seed": 42,
  "steps": {
    "1": {"skill": "step-1-problem-framing",   "name": "Problem framing",   "status": "not_started"},
    "2": {"skill": "step-2-data-audit",        "name": "Data audit",        "status": "not_started"},
    "3": {"skill": "step-3-feature-selection", "name": "Feature selection", "status": "not_started"},
    "4": {"skill": "step-4-model-training",    "name": "Model training",    "status": "not_started"},
    "5": {"skill": "step-5-cutoff-selection",  "name": "Cutoff selection",  "status": "not_started"},
    "6": {"skill": "step-6-signoff",           "name": "Sign-off",          "status": "not_started"}
  }
}
```

Status values: `not_started`, `pass`, `warn`, `fail`, `overridden`, `stale`.
A completed step additionally holds: `decisions` (object), `interim_results` (object), `artifacts` (list of paths), `verdict` (gate JSON), `override_reason` (if overridden), `completed_at`.

A step is **done** if status is `pass`, `warn`, or `overridden`.

## Loop

1. **Dashboard.** Read the state file. Print one line per step: number, name, status icon (✅ pass, ⚠️ warn, ❌ fail, ⏭️ overridden, 🔄 stale, ⬜ not started), and for done steps a one-line summary of key decisions. A step is **available** if all earlier steps are done.
2. **Pick.** Ask with AskUserQuestion which step to work on (offer available steps, plus "Exit"). Re-running a done step is allowed; warn that it will mark all later steps `stale`.
3. **Run the step inline.** Invoke the step's skill via the Skill tool. Pass nothing; the step reads the state file itself for prior decisions. The step interacts with the user and ends with a `## DECISIONS` block containing one JSON object.
4. **Record.** Parse that JSON. Write `decisions`, `interim_results`, `artifacts` into `steps.<N>` in the state file. If the JSON is missing or unparseable, do not write anything; tell the user and offer to re-run. If this is a re-run, set all later done steps to `stale` and clear their verdicts.
5. **Gate.** Spawn the `quality-gate` subagent with the Agent tool (`subagent_type: "quality-gate"`). Its prompt must contain, as literal text:
   - the step number and name
   - the new `decisions` and `interim_results` JSON
   - the `decisions` and `interim_results` of every earlier done step (prior context)
   - the artifact paths (the gate reads them itself)
   - the state file path
   Run it in the foreground; you need the result. Parse the verdict JSON it returns. If it is not valid JSON, retry once, then treat as `fail` with reason "gate returned malformed output".
6. **Show verdict.** Print status, then each finding as `[severity] check: message` grouped by check type. Write `verdict` and `status` (`pass`/`warn`/`fail`) to the state file.
   - `pass`: continue.
   - `warn`: show findings, continue; the user may optionally re-run.
   - `fail`: AskUserQuestion with "Re-run this step" or "Override and accept" (ask for a one-line reason). Override sets status `overridden` and saves `override_reason`. Re-run goes back to step 3 of this loop.
7. **Back to dashboard.** Repeat until step 6 is done or the user exits.

## Rules

- Write the state file after every state change so the session can be resumed cold.
- The dashboard is the only place you ask "which step". Do not auto-advance.
- Quote gate messages verbatim; do not soften a fail.
- If the user asks to reset, confirm first, then delete `.workflow/`.
