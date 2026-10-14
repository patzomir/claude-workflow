---
name: step-1-problem-framing
description: Step 1 of the model workflow. Elicits the business objective, target definition, success metric and error costs from the user. Invoked by top-workflow; not for standalone use.
---

# Step 1: Problem framing

Human-driven step; no simulation. Read `.workflow/state.json` for the project name only.

## Do

Use AskUserQuestion (batch up to 4 questions per call; use free-text "Other" when the choices don't fit) to establish:

1. **Target definition**: what event is the positive class, in one sentence (e.g. "customer cancels within 90 days").
2. **Expected positive rate**: rough prior (offer ranges: <1%, 1–5%, 5–20%, >20%; convert to a number).
3. **Primary metric**: PR-AUC / F1 / recall at fixed precision / expected cost. Explain in one line why accuracy is a trap if the rate is low.
4. **Minimum acceptable value** of that metric to ship.
5. **Costs**: relative cost of a false positive and a false negative (any consistent unit).
6. **Windows**: observation window and prediction horizon, in days.

Summarise back in 3 lines and confirm before emitting. If the user's answers are internally contradictory, say so and re-ask that one item.

## Emit

End your reply with a `## DECISIONS` heading followed by a free-text markdown record and nothing after it. Write it for a reader who has seen none of the conversation (a person or another LLM): plain prose or bullets, no required schema. Include target definition, expected positive rate, primary metric, minimum acceptable value, FP/FN costs, observation window and prediction horizon (days). Give the reason behind each choice in the user's own words where you have them. State every number with its unit. No artifacts.
