# Analyze Complexity Skill for AI Agents

An AI agent skill that evaluates frontend project complexity and produces a structured report with weighted scoring. Uses **scc** for code metrics and **madge** for dependency analysis. Helps AI agents identify refactoring priorities before making changes.

Works with **Cursor**, **Codex**, or any AI editor that supports agent skills.

## What's Inside

```
analyze-complexity/
├── SKILL.md                          # Core rules — the AI reads this
├── README.md                         # You are here
└── references/
    └── scoring-rubric.md             # Detailed scoring criteria per dimension
```

| File | Purpose |
|------|---------|
| `SKILL.md` | Full workflow: tool check, 7 analysis dimensions, report template |
| `scoring-rubric.md` | 1-5 scoring criteria, weighted overall calculation, tool output mapping |

## What It Analyzes

| Dimension | Weight | What It Measures |
|-----------|--------|-----------------|
| Component Coupling | 2.0 (core) | Circular imports, fan-in ranking, orphan files via madge |
| State Management | 2.0 (core) | Store count, cross-store deps, mixed paradigms |
| Type Safety & Code Quality | 1.5 (core) | TS strict mode, `any` density, test coverage, lint health |
| Project Scale | 1.0 | LOC, file count, COCOMO complexity via scc |
| Dependency Weight | 1.0 | Dependency count, heavy libs, version conflicts |
| Routing Complexity | 0.75 | Route count, nesting depth, dynamic routes, guards |
| Build Configuration | 0.75 | Custom plugins, loaders, code-splitting strategy |

Core dimensions carry more weight because they directly determine how hard it is to change code safely.

## Prerequisites

| Tool | Required | Install |
|------|----------|---------|
| **scc** | Yes (system-level) | `brew install scc` (macOS) or [other platforms](https://github.com/boyter/scc#install) |
| **madge** | No install needed | Runs via `npx madge` automatically |

## Quick Start

1. Copy the `analyze-complexity/` directory into your project:

```bash
# For Codex
cp -r analyze-complexity/ .agents/skills/analyze-complexity/

# For Cursor
cp -r analyze-complexity/ .cursor/skills/analyze-complexity/
```

2. Make sure `scc` is installed (`brew install scc`).
3. Done. Use trigger phrases like "analyze complexity", "分析复杂度", or "complexity report" to activate.

## Output

The skill generates a Markdown report containing:

- **Summary table** with weighted scores per dimension
- **Top 3 refactoring hotspots** with reasons
- **Circular dependencies** list
- **High fan-in modules** (top 10 most-imported files)
- **Orphan files** (dead code candidates)
- **Detailed findings** per dimension
- **Prioritized recommendations**

## License

MIT
