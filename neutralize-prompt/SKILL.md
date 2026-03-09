---
name: neutralize-prompt
description: >-
  Rewrite analysis-oriented prompts to elicit deeper, more objective AI output.
  Detects preset conclusions, binary narrowing, anchoring estimates, and missing
  evaluation dimensions in prompts that request review, evaluation, analysis, or
  estimation. Passes through action-oriented prompts unchanged.
  Use when user says "neutralize prompt", "improve prompt", "优化提问",
  "中立化", "改写分析", "提升提问质量", or wants higher-quality analytical output
  from a downstream agent.
---

# Neutralize Prompt

Rewrite analysis-oriented prompts so downstream agents produce deeper, more objective output.

## Key Principle

Not every prompt needs rewriting. This skill distinguishes between three categories:

- **Action prompts** (Fix / Optimize / Create / Implement) — the user wants execution. Pass through unchanged.
- **Explanation prompts** (Explain / Describe / Walk through / Teach) — the user wants understanding. Pass through unchanged.
- **Analysis prompts** (Review / Evaluate / Analyze / Estimate / Compare) — the user wants independent judgment. Rewrite to improve output quality.

## Instructions

1. Receive the user's original prompt.
2. Classify it as Action, Explanation, or Analysis using the Classification step.
3. If Action or Explanation, return the original prompt unchanged.
4. If Analysis, scan for the four patterns defined in the Pattern Detection step.
5. Apply the Rewriting Rules for each detected pattern.
6. Output the result in the Output Format.

## Workflow

### 0. Prompt Classification

Determine whether the prompt requests **execution**, **understanding**, or **judgment**.

| Signal | Category | Examples |
|--------|----------|---------|
| Imperative verb requesting a change | Action | Fix, Refactor, Optimize, Create, Implement, Deploy, Migrate |
| Verb requesting explanation or teaching | Explanation | Explain, Describe, "Walk me through", "What does X do", "这段代码是什么意思" |
| Verb requesting evaluation or judgment | Analysis | Review, Evaluate, Analyze, Estimate, Compare, Assess, Audit |
| Question asking for a judgment call | Analysis | "Is this safe?", "Should we use X?", "How long will this take?" |
| Question asking for factual lookup | Action | "What does this function return?", "Where is X defined?" |

**Ambiguous cases:** If the prompt mixes action and analysis ("Fix the bug and evaluate whether the architecture needs refactoring"), split into two parts. Rewrite only the analysis portion.

If the prompt is classified as Action or Explanation, output:

```
## Result

No rewrite needed — this is an [action/explanation] prompt.

Original prompt passed through unchanged.
```

Stop here. Do not proceed to pattern detection.

### 1. Pattern Detection

Scan the analysis prompt for these four patterns (see `references/prompt-patterns.md` for extended examples):

| # | Pattern | What it does | Why rewriting helps |
|---|---------|-------------|-------------------|
| 1 | **Preset conclusion** | Prompt assumes a specific outcome before analysis | AI skews toward confirming the assumption instead of investigating openly |
| 2 | **Binary narrowing** | Forces yes/no when the answer space is richer | AI gives a shallow verdict instead of multi-dimensional analysis |
| 3 | **Anchoring estimate** | Provides a number that biases AI's own estimation | AI's estimate clusters around the anchor instead of deriving independently |
| 4 | **Missing dimensions** | Asks for evaluation but omits key analysis angles | AI covers only the mentioned dimension, missing critical trade-offs |

For each detected pattern, record: the pattern number, the offending fragment, and why it limits output quality.

### 2. Rewriting Rules

**Minimal rewrite principle:** Only rewrite parts that clearly improve output quality. If a pattern match is weak (e.g. an ambiguous anchoring word) and the rewrite would significantly increase prompt length or alter tone, skip the rewrite. Note it in the Analysis table as "Detected but rewrite cost exceeds benefit."

Apply in order. Each rule includes its rationale.

**Rule 1 — Preset conclusion → Open investigation**

Replace verbs/phrases that presuppose an outcome with open-ended investigation verbs.

| Original | Rewrite | Rationale |
|----------|---------|-----------|
| "Confirm that X is safe" | "Evaluate the safety of X" | Removes confirmation pressure; AI reports both risks and strengths |
| "Prove this is O(n log n)" | "Analyze the time complexity" | AI derives the answer instead of defending a given one |
| "Verify there are no side effects" | "Analyze the impact scope and report all observed behavior changes" | AI reports what it finds, not what you hope it finds |

**Do NOT rewrite** action verbs in execution context: "Fix the bug", "Optimize the query" — these are intent, not bias.

**Rule 2 — Binary narrowing → Multi-dimensional analysis**

Expand yes/no questions into requests for structured evaluation.

| Original | Rewrite | Rationale |
|----------|---------|-----------|
| "Is this production-ready?" | "Evaluate production readiness across reliability, performance, security, and maintainability" | AI produces a structured assessment instead of a single verdict |
| "Should we use Redis or Postgres?" | "Compare Redis and Postgres for this use case across performance, consistency, operational cost, and team familiarity" | AI weighs trade-offs instead of picking a winner |

**Rule 3 — Anchoring estimate → Independent derivation**

Remove numbers that are **requests for AI to agree** rather than **hard constraints**.

| Context | Action |
|---------|--------|
| "This should take about 2 hours, right?" | Remove anchor: "Estimate the effort required" |
| "Latency must be under 100ms" (requirement) | Keep as-is — this is a constraint, not an anchor |
| "I think there are around 10 bugs" | Remove anchor: "Examine and report all issues found" |

**Test:** Would removing the number change the user's actual requirement? If yes, keep it. If no, remove it.

**Rule 4 — Missing dimensions → Dimension prompting**

When the prompt asks for evaluation but specifies no dimensions, append relevant ones based on context.

| Domain | Commonly missing dimensions |
|--------|-----------------------------|
| Code review | Performance, security, maintainability, readability, test coverage |
| Architecture | Scalability, cost, team capability, migration risk, operational complexity |
| Estimation | Scope uncertainty, dependency risk, testing effort, deployment complexity |
| Tech selection | Learning curve, ecosystem maturity, long-term maintenance, vendor lock-in |

Append as: "Consider the following dimensions: [relevant dimensions]."

Constraints:
- Only add dimensions the user clearly omitted. Do not add dimensions already covered in the prompt.
- Append at most 3-4 dimensions. Pick the most relevant to the prompt's context. Fewer focused dimensions produce deeper analysis than many shallow ones.

**Rule 5 — Preserve intent**

The rewritten prompt must accomplish the same goal. Never drop the user's actual objective. If unsure whether a rewrite changes intent, keep the original.

### 3. Output Format

```
## Rewritten Prompt

<rewritten prompt>

## Analysis

| # | Pattern Detected | Original Fragment | Rewrite Applied | Expected Improvement |
|---|-----------------|-------------------|-----------------|---------------------|
| 1 | ...             | "..."             | "..."           | ...                 |

(If no patterns detected: "No improvement opportunities found. Original prompt is already well-structured for analysis.")
```

## Resources

- `references/prompt-patterns.md` — Extended examples of each pattern with before/after rewrites and pass-through cases.
