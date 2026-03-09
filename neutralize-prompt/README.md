# Neutralize Prompt Skill for AI Agents

An AI agent skill that rewrites analysis-oriented prompts to get deeper, more objective output from downstream agents. Detects preset conclusions, binary narrowing, anchoring estimates, and missing evaluation dimensions.

Works with **Cursor**, **Codex**, or any AI editor that supports agent skills.

## What's Inside

```
neutralize-prompt/
├── SKILL.md                          # Core rules — the AI reads this
├── README.md                         # You are here
└── references/
    └── prompt-patterns.md            # Extended examples with before/after rewrites
```

| File | Purpose |
|------|---------|
| `SKILL.md` | Prompt classification, four detection patterns, rewriting rules, output format |
| `prompt-patterns.md` | 30+ examples per pattern (EN/CN), pass-through cases, multi-pattern cases |

## How It Works

Not every prompt needs rewriting. The skill first classifies the prompt:

| Category | Action | Examples |
|----------|--------|---------|
| **Action** | Pass through | Fix, Optimize, Create, Implement |
| **Explanation** | Pass through | Explain, Describe, "Walk me through" |
| **Analysis** | Rewrite | Review, Evaluate, Analyze, Estimate, Compare |

Only analysis prompts enter the rewriting pipeline.

## Four Detection Patterns

| Pattern | Before | After |
|---------|--------|-------|
| **Preset conclusion** | "Confirm the migration is safe" | "Evaluate the risks and safety of this migration" |
| **Binary narrowing** | "Is this production-ready?" | "Evaluate production readiness across reliability, performance, security, and maintainability" |
| **Anchoring estimate** | "This should take about 2 hours, right?" | "Estimate the effort required, breaking down by subtask" |
| **Missing dimensions** | "Review this PR" | "Review this PR for correctness, performance, security, and maintainability" |

## Quick Start

1. Copy the `neutralize-prompt/` directory into your project:

```bash
# For Codex
cp -r neutralize-prompt/ .agents/skills/neutralize-prompt/

# For Cursor
cp -r neutralize-prompt/ .cursor/skills/neutralize-prompt/
```

2. Done. Use trigger phrases like "neutralize prompt", "优化提问", or "提升提问质量" to activate.

## Design Principles

- **Minimal rewrite** — only change what clearly improves output quality
- **Preserve intent** — never drop the user's actual objective
- **Pass-through by default** — action and explanation prompts are never touched

## Inspiration

This skill was inspired by: https://x.com/systematicls/article/2028814227004395561

## License

MIT
