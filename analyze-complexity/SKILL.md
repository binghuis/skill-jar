---
name: analyze-complexity
description: >-
  Analyze frontend project complexity and generate a structured complexity report
  covering file structure, component dependencies, state management, bundle size,
  and code hotspots. Uses scc and madge for precise analysis. Helps AI agents
  identify refactoring priorities.
  Use when user says "analyze complexity", "complexity analysis", "分析复杂度",
  "复杂度分析", "评估复杂度", "complexity report", "复杂度报告",
  or wants to evaluate a frontend project before refactoring.
---

# Analyze Complexity

Evaluate a frontend project's complexity and produce a structured report that AI agents can reference during refactoring.

## Prerequisites

This skill uses two external tools for precise analysis. Check availability before starting.

### Required Tools

| Tool | Purpose | Install |
|------|---------|---------|
| **scc** | Fast code counting: LOC, comments, blanks, COCOMO complexity | `brew install scc` (macOS). User must install manually. |
| **madge** | Module dependency graph, circular import detection, orphan files | Run via `npx madge`. No install needed. |

## Instructions

1. Identify the project root. If the user provides a path, use it; otherwise use the current workspace root.
2. **Check tool availability** — run the Tool Check step below. If madge is missing, auto-install it using the project's detected package manager. If scc is missing, prompt the user to install it (requires system-level install).
3. Detect the framework (React / Vue / Angular / Svelte / plain JS/TS) and package manager from config files.
4. Run each analysis dimension listed in the Workflow section.
5. Score every dimension on a 1-5 scale using `references/scoring-rubric.md`.
6. Aggregate scores into an overall complexity rating and generate the final report.
7. Highlight the **top 3 refactoring hotspots** — files or modules that contribute most to complexity.
8. Output the report to the user. If the user requests a file, write it to `complexity-report.md` in the project root.

## Workflow

### 0. Tool Check

Check scc availability:

```bash
command -v scc && scc --version || echo "MISSING: scc"
```

**madge** — always run via `npx madge` (no global install required). npx downloads it on first use and caches it automatically.

**If scc is MISSING**: scc requires system-level install. Prompt the user to run `brew install scc` (macOS) or see https://github.com/boyter/scc#install. Do not proceed with full analysis until scc is available (or skip scc-dependent sections and note in the report).

### 1. Project Overview (scc)

Run scc in JSON mode and parse the output:

```bash
scc --format json --exclude-dir node_modules,.next,dist,build,.nuxt,coverage <project_root>
```

Extract from the JSON output:
- Total files, code lines, comment lines, blank lines per language.
- COCOMO estimated complexity.
- Comment-to-code ratio (low ratio in large projects = maintenance risk).

Also:
- List top-level directory structure.
- Identify entry points (e.g. `main.ts`, `index.tsx`, `App.vue`).

### 2. Dependency Analysis

- Parse `package.json` for dependency count and notable heavy libraries.
- Flag outdated or deprecated packages if lockfile is available.
- Calculate the ratio of `dependencies` to `devDependencies`.

### 3. Component & Module Structure (madge)

Run madge to analyze the dependency graph:

```bash
# Circular dependency detection
npx madge --circular --extensions ts,tsx,js,jsx,vue <src_dir>

# Full dependency graph as JSON
npx madge --json --extensions ts,tsx,js,jsx,vue <src_dir>

# Orphan files (no importers)
npx madge --orphans --extensions ts,tsx,js,jsx,vue <src_dir>
```

If a `tsconfig.json` exists, add `--ts-config tsconfig.json` for alias resolution.

Extract from the output:
- **Circular imports**: list all cycles; each cycle is a high-priority refactoring target.
- **Fan-in ranking**: from the JSON graph, count how many files import each module. List top 10 highest fan-in modules.
- **Orphan files**: dead code candidates.
- **Directory depth**: compute max nesting depth from file paths.

### 4. State Management Complexity

- Detect state management approach (Redux, Vuex/Pinia, MobX, Zustand, Context API, signals, etc.) via Grep.
- Count stores / slices / contexts.
- Flag files mixing UI rendering with heavy business logic (> 300 LOC with both JSX/template and non-UI logic).

### 5. Routing & Page Complexity

- Parse route config to count routes and nesting depth.
- Identify dynamic routes, guards, and lazy-loaded chunks.
- Flag routes with complex parameter patterns or deeply nested layouts.

### 6. Build & Bundle Indicators

- Read build config (webpack / vite / next.config / etc.) for custom plugins and aliases.
- Count custom webpack/vite plugins and loaders.
- Note code-splitting strategy and chunk count if available.

### 7. Type Safety & Code Quality Signals

- Check for TypeScript adoption: strict mode, `any` usage ratio (count of `any` / total TS LOC), type coverage.
- Count ESLint/Prettier rule overrides and disabled rules.
- Detect test coverage config; estimate test file ratio (`*.test.*` / `*.spec.*` vs source).

### 8. Report Generation

Produce a Markdown report with the following structure:

```
# Complexity Analysis Report

> Generated by analyze-complexity skill
> Tools used: scc vX.X, madge vX.X

## Summary
| Dimension                  | Weight | Score (1-5) | Notes |
|----------------------------|--------|-------------|-------|
| Component Coupling         | 2.0    |             |       |
| State Management           | 2.0    |             |       |
| Type Safety & Code Quality | 1.5   |             |       |
| Project Scale              | 1.0    |             |       |
| Dependency Weight          | 1.0    |             |       |
| Routing Complexity         | 0.75   |             |       |
| Build Configuration        | 0.75   |             |       |
| **Overall (weighted)**     |        |             |       |

## Top 3 Refactoring Hotspots
1. **<file/module>** — reason
2. **<file/module>** — reason
3. **<file/module>** — reason

## Circular Dependencies
[List all cycles detected by madge, or "None detected"]

## Orphan Files
[List files with no importers, or "None detected"]

## High Fan-in Modules (Top 10)
| Module | Imported by (count) |
|--------|---------------------|
| ...    | ...                 |

## Detailed Findings
### Project Overview
- LOC breakdown (from scc): ...
- COCOMO complexity: ...
- Comment-to-code ratio: ...

### Dependency Analysis
...
### Component & Module Structure
...
### State Management
...
### Routing & Pages
...
### Build & Bundle
...
### Type Safety & Code Quality
...

## Recommendations
- Priority 1: ...
- Priority 2: ...
- Priority 3: ...
```

## Resources

- `references/scoring-rubric.md` — Detailed scoring criteria and tool output mapping for each dimension.
