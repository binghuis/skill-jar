# Feature-Sliced Design Skill for AI Agents

An AI agent skill that teaches coding assistants to follow [Feature-Sliced Design](https://feature-sliced.design) (FSD) architecture when generating, reviewing, or refactoring frontend code.

Works with **Cursor**, **Codex**, or any AI editor that supports agent skills.

## What's Inside

```
feature-slicing/
├── SKILL.md                      # Core rules — the AI reads this
├── README.md                     # You are here
└── references/
    ├── implementation.md         # Code patterns for each layer
    ├── nextjs.md                 # Next.js App Router / Pages Router integration
    └── migration.md              # Incremental migration strategy
```

| File | Purpose |
|------|---------|
| `SKILL.md` | Architecture rules, layer hierarchy, import constraints, decision trees, naming conventions |
| `implementation.md` | Complete code examples: entity, feature, widget, page, shared, app layer patterns |
| `nextjs.md` | How to reconcile FSD with Next.js file-based routing (App Router & Pages Router) |
| `migration.md` | Step-by-step guide for incrementally migrating a legacy codebase to FSD |

## Key Rules at a Glance

These are enforced by `SKILL.md` — see it for full details.

- **6-layer hierarchy** — `app/ > pages/ > widgets/ > features/ > entities/ > shared/` with strict top-down imports
- **Slice / Segment organization** — layers contain slices (business domains); slices contain segments (`ui/`, `api/`, `model/`, `lib/`, `config/`)
- **Public API pattern** — one `index.ts` per slice, explicit named exports only, no wildcards
- **`model/` file naming** — suffix conventions: `.type.ts`, `.schema.ts`, `.store.ts`, `.const.ts`, `.logic.ts`
- **Hook & Action placement** — by responsibility (state → `model/`, data → `api/`, UI → `ui/`), never a `hooks/` segment
- **Anti-patterns checklist** — cross-slice imports, generic segments, business logic in shared, etc.

## Quick Start

1. Copy the `feature-slicing/` directory into your project:

```bash
# For Codex
cp -r feature-slicing/ .agents/skills/feature-slicing/

# For Cursor
cp -r feature-slicing/ .cursor/skills/feature-slicing/
```

2. Done. The AI will automatically apply FSD rules when creating components, pages, or refactoring code.

No additional configuration required.

## Customization Guide

The skill ships with generic examples. Adapt the following areas to match your project's tech stack.

### State Management

`SKILL.md` and `implementation.md` use **Zustand** in `.store.ts` examples.

If your project uses Redux, Jotai, Pinia, MobX, or another library:
- Update the `.store.ts` description in `SKILL.md` (model/ File Naming Convention table)
- Replace store examples in `implementation.md` (features/auth/model/auth.store.ts, etc.)

### UI Library

`implementation.md` uses plain JSX + Tailwind CSS for `shared/ui` components.

If your project uses Ant Design, MUI, shadcn/ui, or another component library:
- Replace `shared/ui` component examples in `implementation.md`
- The FSD rules themselves don't change — only the code samples

### API Layer

Examples use hand-written **Axios** + **TanStack React Query** hooks.

If your project uses Orval, openapi-typescript, tRPC, or SWR:
- Update `api/` segment examples in `implementation.md`
- If using code generation (e.g. Orval), note in `SKILL.md` that `api/` files may be auto-generated

### Routing

Examples use `react-router-dom`. If your project uses a different router:

| Router | Action |
|--------|--------|
| Next.js App Router | See `references/nextjs.md` (already included) |
| React Router 7 | Update loader/action patterns in `implementation.md` |
| TanStack Router | Replace route config examples in `implementation.md` |

### CSS Strategy

Examples use **Tailwind CSS** utility classes.

If your project uses CSS Modules, styled-components, or vanilla-extract:
- Update component examples in `implementation.md` to reflect your styling approach
- The FSD layer/segment rules are CSS-agnostic

### Component Declaration Style

All examples use `function` declarations:

```tsx
export function UserCard({ user }: UserCardProps) { ... }
```

If your team prefers arrow functions, update code examples across all files.

### File Naming

All files use **kebab-case** (`user-card.tsx`, `auth.store.ts`).

If your team uses PascalCase or camelCase, do a global find-and-replace across the skill files.

## Automated Linting

Pair this skill with automated tools to enforce FSD rules in CI:

### Steiger (recommended)

Official FSD architecture linter. Checks file structure and import boundaries.

```bash
npx steiger src
```

Key rules: `forbidden-imports`, `excessive-slicing`, `insignificant-slice`

GitHub: [feature-sliced/steiger](https://github.com/feature-sliced/steiger)

### @feature-sliced/eslint-config

ESLint plugin for FSD (beta).

```bash
npm install -D @feature-sliced/eslint-config eslint-plugin-import eslint-plugin-boundaries
```

Core rules: `layers-slices`, `public-api`, `import-order`

GitHub: [feature-sliced/eslint-config](https://github.com/feature-sliced/eslint-config)

### Manual ESLint Alternative

If you prefer not to use the above plugins, configure `eslint-plugin-import` with `no-restricted-paths` to manually enforce layer boundaries.

## Resources

- [Official Documentation](https://feature-sliced.design)
- [GitHub Organization](https://github.com/feature-sliced)
- [Official Examples](https://github.com/feature-sliced/examples)
- [Awesome FSD](https://github.com/feature-sliced/awesome) — curated articles, videos, tools
- [FSD v2.1 Release Notes](https://github.com/feature-sliced/documentation/releases/tag/v2.1)

## License

MIT
