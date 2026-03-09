# Complexity Scoring Rubric

Each dimension is scored 1-5. Use the criteria below for consistent evaluation.

## How to Use Tool Output

### scc JSON → Project Scale Score

Parse the `scc --format json` output. Sum the `Code` field across all languages to get total LOC. Sum `Lines` for total files.

| scc Output | Maps to |
|------------|---------|
| Sum of `Code` across languages | Total LOC for Project Scale scoring |
| Sum of `Comment` / Sum of `Code` | Comment-to-code ratio (< 0.05 in large projects = risk) |
| `EstimatedScheduleMonths` (COCOMO) | Supplementary complexity indicator |

### madge → Component Coupling Score

| madge Command | Output | Maps to |
|---------------|--------|---------|
| `madge --circular` | List of cycles | Any cycles → score >= 4 |
| `madge --json` | Dependency graph | Count importers per module for fan-in ranking |
| `madge --orphans` | Unreferenced files | Dead code candidates (informational, not scored) |

**Fan-in scoring guide:**
- Max fan-in < 5 → score 1
- Max fan-in 5-15 → score 2
- Max fan-in 15-25 → score 3
- Max fan-in 25-40 with no cycles → score 4
- Max fan-in > 40 or widespread cycles → score 5

---

## Project Scale

| Score | Criteria |
|-------|----------|
| 1 | < 5k LOC, < 30 files |
| 2 | 5k-20k LOC, 30-100 files |
| 3 | 20k-60k LOC, 100-300 files |
| 4 | 60k-150k LOC, 300-800 files |
| 5 | > 150k LOC or > 800 files |

## Dependency Weight

| Score | Criteria |
|-------|----------|
| 1 | < 10 dependencies, no heavy frameworks |
| 2 | 10-25 dependencies, one major framework |
| 3 | 25-50 dependencies, multiple UI/state libraries |
| 4 | 50-80 dependencies, custom build plugins |
| 5 | > 80 dependencies, or multiple conflicting versions of key libs |

## Component Coupling

| Score | Criteria |
|-------|----------|
| 1 | Flat structure, no file imported by > 5 others, no cycles |
| 2 | Mild coupling, a few shared utilities, no cycles |
| 3 | Some modules imported by > 15 files, directory depth > 4, no cycles |
| 4 | Circular imports detected, high fan-in modules (> 25 importers) |
| 5 | Widespread circular imports, god modules imported everywhere |

## State Management

| Score | Criteria |
|-------|----------|
| 1 | Local state only (useState / ref) |
| 2 | Single lightweight store (Zustand, Pinia single store) |
| 3 | Multiple stores/slices, some cross-store dependencies |
| 4 | Complex Redux with middleware, sagas, or heavy normalized state |
| 5 | Mixed state paradigms (Redux + Context + local), unclear data flow |

## Routing Complexity

| Score | Criteria |
|-------|----------|
| 1 | < 5 flat routes |
| 2 | 5-15 routes, simple nesting |
| 3 | 15-40 routes, dynamic params, route guards |
| 4 | 40-80 routes, deeply nested layouts, complex auth guards |
| 5 | > 80 routes, or route config spread across multiple files with dynamic generation |

## Build Configuration

| Score | Criteria |
|-------|----------|
| 1 | Zero-config (CRA, Vite defaults, Next.js defaults) |
| 2 | Minor config tweaks (aliases, env vars) |
| 3 | Custom plugins or loaders (< 5) |
| 4 | Heavy customization (5-10 plugins), monorepo setup |
| 5 | Ejected CRA, > 10 custom plugins, or multi-target builds |

## Type Safety & Code Quality

`any` density = count of explicit `any` / total TypeScript LOC × 1000 (per-mille).

| Score | Criteria |
|-------|----------|
| 1 | Full TypeScript strict, `any` density < 1‰, test coverage > 80%, clean lint |
| 2 | TypeScript strict, `any` density < 3‰, tests 50-80%, few lint overrides |
| 3 | TypeScript non-strict or partial adoption, `any` density 3-8‰, tests 20-50% |
| 4 | Mixed JS/TS, `any` density > 8‰, test coverage < 20% |
| 5 | Pure JS with no types, no tests, many disabled lint rules |

## Overall Score Calculation

Dimensions are weighted by their impact on refactoring difficulty:

| Dimension | Weight | Tier |
|-----------|--------|------|
| Component Coupling | 2.0 | Core |
| State Management | 2.0 | Core |
| Type Safety & Code Quality | 1.5 | Core |
| Project Scale | 1.0 | Supporting |
| Dependency Weight | 1.0 | Supporting |
| Routing Complexity | 0.75 | Supporting |
| Build Configuration | 0.75 | Supporting |

```
overall = round(weighted_sum / total_weight)
```

Core dimensions (Coupling, State, Type Safety) carry the most weight because they directly determine how hard it is to change code safely. Supporting dimensions provide context but rarely block refactoring on their own.

Interpretation:
- **1-2**: Low complexity — straightforward to refactor.
- **3**: Moderate — plan refactoring in phases.
- **4**: High — requires careful dependency mapping before changes.
- **5**: Very high — recommend incremental strangler-fig approach.
