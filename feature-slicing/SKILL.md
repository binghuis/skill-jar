---
name: feature-slicing
description: Proactively apply when creating new features/components/pages or setting up frontend project structure. Triggers on FSD, feature slicing, frontend architecture, layer structure, module boundaries, scalable frontend, slice organization. Use when restructuring React/Next.js/Vue/Remix projects, organizing frontend code, fixing import violations, or migrating legacy codebases. Feature-Sliced Design (FSD) architecture for frontend projects.
---

# Feature-Sliced Design Architecture

Frontend architecture methodology with strict layer hierarchy and import rules for scalable, maintainable applications. FSD organizes code by **business domain** rather than technical role.

> **Official Docs:** [feature-sliced.design](https://feature-sliced.design) | **GitHub:** [feature-sliced](https://github.com/feature-sliced)

---

## THE IMPORT RULE (Critical)

**Modules can ONLY import from layers strictly below them. Never sideways or upward.**

```
Layer (top → bottom):    Can import from:
─────────────────────    ────────────────────────────────────
app/                     pages, widgets, features, entities, shared
pages/                   widgets, features, entities, shared
widgets/                 features, entities, shared
features/                entities, shared
entities/                shared (use @x for cross-entity)
shared/                  external packages only
```

### Import Matrix

Row = importer (who imports), Column = importee (what is imported):

|  | app | pages | widgets | features | entities | shared |
|--|-----|-------|---------|----------|----------|--------|
| **app** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **pages** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **widgets** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| **features** | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **entities** | ❌ | ❌ | ❌ | ❌ | @x* | ✅ |
| **shared** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅** |

*Use @x notation for cross-entity references
**shared/ internal cross-segment imports are allowed (e.g. shared/ui → shared/lib)

| Violation | Example | Fix |
|-----------|---------|-----|
| Cross-slice (same layer) | `features/auth` → `features/user` | Extract to `entities/` or `shared/` |
| Upward import | `entities/user` → `features/auth` | Move shared code down |
| Shared importing up | `shared/` → `entities/` | Shared has NO internal deps |

**Exception:** `app/` and `shared/` have no slices, so internal cross-imports are allowed within them.

---

## Layer Hierarchy

| Layer | Purpose | Has Slices | Required |
|-------|---------|------------|----------|
| `app/` | Initialization, routing, providers, global styles | No | Yes |
| `pages/` | Route-based screens (one slice per route) | Yes | Yes |
| `widgets/` | Complex reusable UI blocks (header, sidebar) | Yes | No |
| `features/` | User interactions with business value (login, checkout) | Yes | No |
| `entities/` | Business domain models (user, product, order) | Yes | No |
| `shared/` | Project-agnostic infrastructure (UI kit, API client, utils) | No | Yes |

**Note:** `processes/` layer is DEPRECATED. Use pages with composition instead.

### Minimal FSD Setup

Start small, add layers as needed:

```
src/
├── app/
├── pages/
└── shared/
```

Add `entities/`, `features/`, `widgets/` when complexity grows.

---

## Quick Decision Trees

### "Where does this code go?"

```
START: Where does this code go?
│
├─ Reusable infrastructure without business logic?
│  └─ YES → shared/
│
├─ Business domain object/data model?
│  └─ YES → entities/
│
├─ User interaction with business value?
│  ├─ YES, reused across pages → features/
│  └─ YES, single page only → Keep in pages/ slice
│
├─ Complex, reusable UI composition?
│  └─ YES → widgets/
│
├─ Route/screen component?
│  └─ YES → pages/
│
└─ App-wide initialization/config?
   └─ YES → app/
```

### "Feature or Entity?"

| Entity (noun) | Feature (verb) |
|---------------|----------------|
| `user` — user data model | `auth` — login/logout actions |
| `product` — product info | `add-to-cart` — adding to cart |
| `comment` — comment data | `write-comment` — creating comments |
| `order` — order record | `checkout` — completing purchase |

**Rule:** Entities represent THINGS with identity. Features represent ACTIONS with side effects.

### "Which segment?"

```
Core segments (within a slice):
├─ ui/      → React components, styles
├─ api/     → Backend calls, data fetching, DTOs
├─ model/   → Types, schemas, stores, business logic
├─ lib/     → Slice-specific utilities
└─ config/  → Feature flags, constants

Additional segments as needed:
├─ routes/  → Route path constants, matching patterns (shared only)
├─ i18n/    → Translation files, locale config (shared only)
└─ store/   → Global store setup (app only)
```

**Naming:** Use purpose-driven names (`api/`, `model/`) not essence-based (`hooks/`, `types/`).

### File Naming Convention (Heterogeneous Directories)

When a directory contains **multiple distinct file types** (heterogeneous), use suffix conventions to distinguish them. This applies to any segment or co-located component directory — not just `model/`.

| Suffix | Purpose | Example |
|--------|---------|---------|
| `.type.ts` | Type definitions, interfaces | `user.type.ts` |
| `.schema.ts` | Zod / Yup validation rules | `user.schema.ts` |
| `.store.ts` | Zustand store (state + actions) | `auth.store.ts` |
| `.const.ts` | `as const` constants, label/color maps | `task-status.const.ts` |
| `.logic.ts` | Pure functions, calculations | `order.logic.ts` |

**Format:** `{business-name}.{category}.ts` — kebab-case business name + category suffix.

**Rule:** Only use suffixes from the table above — do not invent new ones (e.g. `.util.ts`, `.helpers.ts`, `.utils.ts` are forbidden).

Directories whose files are **homogeneous** (e.g. a `ui/` folder containing only React components) do not need suffixes.

---

## Layers & Directory Structure

```
src/
├── app/          # Initialization, routing, providers, global styles (no slices)
├── pages/        # Route-based screens (one slice per route)
├── widgets/      # Complex composed UI blocks (optional)
├── features/     # User interactions with business value (optional)
├── entities/     # Business domain models (optional)
└── shared/       # Project-agnostic infrastructure (no slices)
```

### Shared

Foundation layer for external connections and utilities. **No business domain knowledge.**

```
shared/
├── ui/           # Design system components
├── api/          # API client, interceptors
├── lib/          # Utilities (dates, validation)
├── config/       # Environment, constants
├── routes/       # Route path constants and matching patterns
└── i18n/         # Translations
```

**Guidelines:**
- Avoid generic names: `components/`, `hooks/`, `utils/`, `types/`
- Use purpose-driven segment names
- Should be extractable to a separate package
- NO domain logic

**TypeScript Types:**
- Utility types → `shared/lib/utility-types`
- DTOs → `shared/api` near request functions
- Do NOT create a generic `shared/types/` folder — `types` is an essence-based name, not purpose-driven

### Entities

Real-world business concepts the application works with.

```
entities/{entity-name}/
├── ui/                    # Entity UI (Card, Avatar)
├── api/                   # CRUD operations
├── model/                 # Types, mappers, validation
│   ├── {name}.type.ts
│   ├── {name}.schema.ts
│   ├── {name}.const.ts
│   └── {name}.logic.ts
└── index.ts               # Public API
```

**What belongs here:**
- Data models and TypeScript interfaces
- API functions for CRUD operations
- Reusable UI representations
- Validation schemas (Zod, Yup)
- Entity-specific mappers (DTO → Domain)

**What doesn't belong:**
- User interactions (→ features)
- Page layouts (→ pages)
- Composed UI blocks (→ widgets)

### Features

User-facing interactions that provide business value.

```
features/{feature-name}/
├── ui/                    # Feature UI
├── api/                   # Feature API calls
├── model/                 # State, schemas
│   ├── {name}.type.ts
│   ├── {name}.store.ts
│   └── {name}.schema.ts
└── index.ts               # Public API
```

**Key principle:** Not everything is a feature. Per [FSD v2.1](https://github.com/feature-sliced/documentation/releases/tag/v2.1), keep non-reused interactions in page slices.

### Widgets

Large, self-sufficient UI blocks. Most useful when reused across multiple pages, or when a page has multiple large independent blocks.

```
widgets/{widget-name}/
├── ui/                    # Composed UI
└── index.ts               # Public API
```

**When to use widgets:**
- Component is reused across multiple pages
- Component is complex with multiple children
- Component delivers a complete use case

**Widget vs Feature:**
- Widget = composed UI block (visual)
- Feature = user interaction (behavioral)

Widgets often compose multiple features:
```tsx
// widgets/header/ui/header.tsx
import { UserAvatar } from '@/entities/user';
import { LogoutButton } from '@/features/auth';
import { SearchBox } from '@/features/search';
```

**Don't create widgets for:**
- Single-use components (keep in page)
- Simple compositions (compose in page directly)

### Pages

Individual screens or routes. One slice per route (generally).

```
pages/{page-name}/
├── ui/                    # Page components
├── api/                   # Loaders, server actions
├── model/                 # Page-specific state
└── index.ts               # Public API
```

**Guidelines:**
- One slice per route (generally)
- Similar pages can share a slice (login/register)
- Pages compose widgets, features, entities
- Minimal business logic — delegate to lower layers
- Non-reused interactions stay in page slice (v2.1)

### App

Application-wide configuration and initialization.

```
app/
├── providers/        # React context providers
│   ├── theme-provider.tsx
│   ├── query-provider.tsx
│   └── index.ts
├── routes/           # Router configuration
│   └── router.tsx
├── store/            # Global store configuration
│   └── index.ts
├── styles/           # Global CSS, theme tokens
│   ├── globals.css
│   └── theme.ts
└── index.tsx         # Entry point
```

**Responsibilities:**
- Initialize application state
- Set up routing
- Configure global providers
- Define global styles
- Application-wide error boundaries

---

## Public API Pattern

A public API is a **contract** between a slice and consuming code. It controls which objects are accessible and how they can be imported.

**Three goals of quality public APIs:**
1. **Protection from structural changes** — Shield consumers from internal refactoring
2. **Behavioral transparency** — Significant changes reflect in the API
3. **Selective exposure** — Only necessary parts exposed

Every slice MUST expose a public API via `index.ts`. External code imports ONLY from this file.

```typescript
// entities/user/index.ts
export { UserCard } from './ui/user-card';
export { UserAvatar } from './ui/user-avatar';
export { getUser, updateUser } from './api/user-api';
export type { User, UserRole } from './model/user.type';
export { userSchema } from './model/user.schema';
```

```typescript
// ✅ Correct
import { UserCard, type User } from '@/entities/user';

// ❌ Wrong
import { UserCard } from '@/entities/user/ui/user-card';
```

**Avoid wildcard exports** — they expose internals and harm tree-shaking:
```typescript
// ❌
export * from './ui';

// ✅
export { UserCard } from './ui/user-card';
```

### Avoid Segment-Level Index Files

The official FSD docs recommend **against** having index files in segments on layers that have slices. Extra barrel files slow down dev servers and add unnecessary indirection.

```
entities/user/
├── ui/
│   ├── user-card.tsx
│   └── user-avatar.tsx       # ← NO index.ts in segments
├── api/
│   └── user-api.ts
├── model/
│   ├── user.type.ts
│   └── user.schema.ts
└── index.ts                  # ← Only ONE index.ts per slice
```

**Exception:** `shared/ui` and `shared/lib` may use per-component index files to improve tree-shaking.

### Avoiding Circular Imports

**Problem:** Importing from index within a slice causes circulars.

```typescript
// ❌
import { UserCard } from '../index';

// ✅
import { UserCard } from '../ui/user-card';
```

**Rule:** Within a slice, use relative imports. External consumers use the public API.

### Tree-Shaking Optimization

For large shared UI libraries, split into component-level indices:

```
shared/ui/
├── button/
│   ├── button.tsx
│   └── index.ts
├── input/
│   ├── input.tsx
│   └── index.ts
├── modal/
│   ├── modal.tsx
│   └── index.ts
└── index.ts
```

**Import patterns:**
```typescript
import { Button, Input } from '@/shared/ui';

import { Button } from '@/shared/ui/button';
```

### Index File Challenges

**Four major issues:**

1. **Circular imports** — Internal files reimporting from index
2. **Tree-shaking failures** — Unrelated utilities bundled together
3. **Weak enforcement** — Nothing prevents direct imports technically
4. **Performance degradation** — Too many indices slow dev servers

**Solutions:**
- Use relative imports within slices
- Create separate indices per component in `shared/`
- Review imports during code review
- Consider monorepo for very large projects

---

## Cross-Entity References (@x Notation)

When entities legitimately reference each other, use the `@x` notation:

```
entities/
├── product/
│   ├── @x/
│   │   └── order.ts    # API specifically for order entity
│   └── index.ts
└── order/
    └── model/order.type.ts  # Imports from product/@x/order
```

```typescript
// entities/product/@x/order.ts
export type { ProductId } from '../model/product.type';

// entities/order/model/order.type.ts
import type { ProductId } from '@/entities/product/@x/order';
```

**Guidelines:**
- Keep cross-imports minimal
- Document why the cross-reference exists
- Consider merging entities if references are extensive
- Use only on Entities layer

---

## Hook & Action Placement

Hooks and actions are **implementation techniques**, not segments. Place them by what they do, not what they are.

### Hook placement

| Hook responsibility | Segment | Example |
|---------------------|---------|---------|
| Manage / derive state | `model/` | `useAuthStore` (inside `.store.ts`) |
| Fetch / mutate data | `api/` | `useUserQuery` (inside `queries.ts`) |
| UI interaction logic | `ui/` (same directory) | `use-login-form.ts` beside `login-form.tsx` |
| Generic utility | `lib/` or `shared/lib/` | `useDebounce`, `useMediaQuery` |

- Use `use-` prefix when extracting a hook to its own file — no `.hook.ts` suffix needed.
- **Never** create a `hooks/` segment.

### Action placement

| Action type | Where | Example |
|-------------|-------|---------|
| Zustand action | Inside `.store.ts` | `addItem`, `clearCart` in `cart.store.ts` |
| Server Action | `api/` | `login.action.ts` with `"use server"` |
| UI event handler | Component body | `handleSubmit` inside the component |

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Cross-slice import | `features/a` → `features/b` | Extract shared logic down |
| Generic segments | `components/`, `hooks/` | Use `ui/`, `lib/`, `model/` |
| Wildcard exports | `export * from './button'` | Explicit named exports |
| Business logic in shared | Domain logic in `shared/lib` | Move to `entities/` |
| Single-use widgets | Widget used by one page | Keep in page slice |
| Skipping public API | Import from internal paths | Always use `index.ts` |
| Making everything a feature | All interactions as features | Only reused actions |
| Too many layers | All 6 layers from day one | Start with `app/`, `pages/`, `shared/`; add as needed |

---

## TypeScript Configuration

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

---

## Reference Documentation

| File | Purpose |
|------|---------|
| [references/implementation.md](references/implementation.md) | Code patterns: entities, features, widgets, pages, shared, React Query |
| [references/nextjs.md](references/nextjs.md) | App Router integration, page re-exports, server actions |
| [references/migration.md](references/migration.md) | Incremental migration strategy, common patterns |

## Resources

### Official Sources
- **Official Documentation**: https://feature-sliced.design
- **GitHub Organization**: https://github.com/feature-sliced
- **Official Examples**: https://github.com/feature-sliced/examples
- **Specification**: https://feature-sliced.design/docs/reference

### Community
- **Awesome FSD**: https://github.com/feature-sliced/awesome (curated articles, videos, tools)
- **v2.1 Release Notes**: https://github.com/feature-sliced/documentation/releases/tag/v2.1
