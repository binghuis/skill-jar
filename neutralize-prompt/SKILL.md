---
name: neutralize-prompt
description: >-
  Transform any user prompt into a neutral, unbiased formulation that avoids
  presupposing outcomes. Strips leading questions, confirmation bias, and
  directional language while preserving the original intent. Outputs the
  rewritten prompt ready for downstream agents.
  Use when user says "neutralize prompt", "neutral rewrite", "中立化",
  "去偏见", "中立表述", "rewrite neutral", or wants to remove bias from a prompt
  before sending it to another agent.
---

# Neutralize Prompt

Transform any input prompt into a neutral formulation that does not presuppose conclusions.

## Instructions

1. Receive the user's original prompt (the text to be neutralized).
2. Analyze the prompt against the five bias patterns defined in the Bias Detection step.
3. Rewrite the prompt following the Rewriting Rules.
4. Output the result in the Output Format below.
5. If no bias is detected, return the original prompt unchanged and state "No bias detected."

## Workflow

### 1. Bias Detection

Scan the input for these five bias patterns (see `references/bias-patterns.md` for extended examples):

| # | Pattern | Signal | Example |
|---|---------|--------|---------|
| 1 | **Presupposed existence** | Prompt assumes something exists that may not | "Find the bug in…" assumes a bug exists |
| 2 | **Directional verb** | Verb steers toward a specific outcome | "Prove that…", "Confirm that…", "Fix the…" |
| 3 | **Loaded qualifier** | Adjective/adverb injects judgment | "the obviously broken…", "the poorly written…" |
| 4 | **Binary framing** | Forces a yes/no when the space is wider | "Is this code good or bad?" |
| 5 | **Anchoring value** | Provides a number/estimate that biases evaluation | "This should take about 2 hours, right?" |

For each detected pattern, record: the pattern type, the offending fragment, and why it biases.

### 2. Rewriting Rules

Apply these transformations in order:

1. **Existence → Exploration**: Replace "find X" with "examine … and report whether X exists and any other observations."
2. **Directional → Open-ended**: Replace outcome-steering verbs with neutral verbs.
   - "Fix" → "Examine and describe the current behavior of"
   - "Prove/Confirm" → "Evaluate whether"
   - "Optimize" → "Analyze the performance characteristics of"
3. **Loaded qualifiers → Remove or soften**: Delete judgmental adjectives, or replace with "current."
   - "the broken function" → "the function"
   - "the poorly designed API" → "the API"
4. **Binary → Spectrum**: Expand yes/no questions into open-ended analysis requests.
   - "Is X good?" → "Evaluate X across relevant quality dimensions."
5. **Anchoring → Defer**: Remove specific numbers/estimates and ask the agent to derive its own.
   - "This should take 2 hours" → "Estimate the effort required"
6. **Preserve intent**: The rewritten prompt must accomplish the same goal. Never drop the user's actual objective.
7. **Append observation clause**: End with "Report all findings, including cases where no issues are found" to normalize null results.

### 3. Output Format

Return the result in this structure:

```
## Neutralized Prompt

<rewritten prompt>

## Bias Analysis

| # | Pattern Detected | Original Fragment | Transformation Applied |
|---|-----------------|-------------------|----------------------|
| 1 | ...             | "..."             | ...                  |

(If no bias detected: "No bias detected. Original prompt is already neutral.")
```

## Resources

- `references/bias-patterns.md` — Extended examples of each bias pattern with before/after rewrites.
