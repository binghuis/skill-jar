# Migrating to Feature-Sliced Design

> **Source:** [Migration from Custom Architecture](https://feature-sliced.design/docs/guides/migration/from-custom) | [Migration from v2.0 to v2.1](https://feature-sliced.design/docs/guides/migration/from-v2-0)

## When to Migrate

Consider migrating to FSD if:
- Project has grown too large and interconnected
- Implementing new features takes longer than expected
- Onboarding new developers is difficult
- Circular dependencies are common
- Code ownership is unclear

**Don't migrate if** the current architecture works well for your team.

---

## Migration Strategy: Incremental Adoption

FSD supports incremental adoption. Don't rewrite everything at once.

```
Phase 1: Setup FSD structure alongside existing code
Phase 2: Migrate shared utilities
Phase 3: Extract entities
Phase 4: Extract features
Phase 5: Migrate pages
Phase 6: Clean up and enforce rules
```

---

## Phase 1: Setup FSD Structure

### Create Directory Structure

```bash
mkdir -p src/{app,pages,widgets,features,entities,shared}/{ui,api,model,lib}
```

### Configure Path Aliases

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"]
    }
  }
}
```

---

## Phase 2: Migrate Shared Utilities

### Before (Typical Structure)

```
src/
├── utils/
│   ├── api.ts
│   ├── dates.ts
│   ├── validation.ts
│   └── constants.ts
├── hooks/
│   ├── useLocalStorage.ts
│   └── useDebounce.ts
└── components/
    ├── Button.tsx
    ├── Input.tsx
    └── Modal.tsx
```

### After (FSD Shared Layer)

```
src/shared/
├── api/
│   ├── client.ts          # from utils/api.ts
│   └── index.ts
├── lib/
│   ├── dates.ts           # from utils/dates.ts
│   ├── validation.ts      # from utils/validation.ts
│   ├── use-local-storage.ts # from hooks/
│   ├── use-debounce.ts     # from hooks/
│   └── index.ts
├── config/
│   ├── constants.ts       # from utils/constants.ts
│   └── index.ts
└── ui/
    ├── button/
    │   ├── button.tsx     # from components/
    │   └── index.ts
    ├── input/
    ├── modal/
    └── index.ts
```

### Migration Script

```bash
# Move utils
mv src/utils/api.ts src/shared/api/client.ts
mv src/utils/dates.ts src/shared/lib/dates.ts
mv src/utils/validation.ts src/shared/lib/validation.ts
mv src/utils/constants.ts src/shared/config/constants.ts

# Move hooks to lib (rename to kebab-case)
for hook in src/hooks/*.ts; do
  kebab=$(basename "$hook" | sed 's/\([A-Z]\)/-\L\1/g; s/^-//')
  mv "$hook" "src/shared/lib/$kebab"
done

# Move components to ui (rename to kebab-case)
for component in src/components/*.tsx; do
  pascal=$(basename "$component" .tsx)
  kebab=$(echo "$pascal" | sed 's/\([A-Z]\)/-\L\1/g; s/^-//')
  mkdir -p "src/shared/ui/$kebab"
  mv "$component" "src/shared/ui/$kebab/$kebab.tsx"
  echo "export { $pascal } from './$kebab';" > "src/shared/ui/$kebab/index.ts"
done
```

### Update Imports

```typescript
// Before
import { formatDate } from '@/utils/dates';
import { Button } from '@/components/Button';

// After
import { formatDate } from '@/shared/lib';
import { Button } from '@/shared/ui';
```

---

## Phase 3: Extract Entities

### Identify Entities

Look for business domain objects:
- Types/interfaces representing domain concepts
- API calls for CRUD operations
- Reusable UI components showing domain data

### Before (Scattered)

```
src/
├── types/
│   └── user.ts
├── api/
│   └── userApi.ts
├── components/
│   ├── UserAvatar.tsx
│   └── UserCard.tsx
└── store/
    └── userSlice.ts
```

### After (FSD Entity)

```
src/entities/user/
├── ui/
│   ├── user-avatar.tsx
│   └── user-card.tsx
├── api/
│   └── user-api.ts
├── model/
│   ├── user.type.ts
│   └── user.store.ts
└── index.ts
```

### Entity Public API

```typescript
// entities/user/index.ts
export { UserAvatar } from './ui/user-avatar';
export { UserCard } from './ui/user-card';
export { getUser, updateUser, deleteUser } from './api/user-api';
export type { User, UserRole } from './model/user.type';
export { useUserStore } from './model/user.store';
```

---

## Phase 4: Extract Features

### Identify Features

Features are user interactions with business value:
- Login/logout functionality
- Add to cart
- Search
- Submit forms

### Before (Mixed Concerns)

```
src/
├── components/
│   ├── LoginForm.tsx
│   └── LogoutButton.tsx
├── api/
│   └── authApi.ts
└── store/
    └── authSlice.ts
```

### After (FSD Feature)

```
src/features/auth/
├── ui/
│   ├── login-form.tsx
│   └── logout-button.tsx
├── api/
│   └── auth-api.ts
├── model/
│   ├── auth.type.ts
│   ├── auth.schema.ts
│   └── auth.store.ts
└── index.ts
```

---

## Phase 5: Migrate Pages

### Before

```
src/pages/
├── Home.tsx
├── ProductList.tsx
└── ProductDetail.tsx
```

### After

```
src/pages/
├── home/
│   ├── ui/
│   │   └── home-page.tsx
│   └── index.ts
├── products/
│   ├── ui/
│   │   └── products-page.tsx
│   └── index.ts
└── product-detail/
    ├── ui/
    │   └── product-detail-page.tsx
    ├── api/
    │   └── loader.ts
    └── index.ts
```

### Refactor Page Components

```typescript
// Before: src/pages/ProductDetail.tsx
import { useParams } from 'react-router-dom';
import { useQuery } from '@tanstack/react-query';
import { fetchProduct } from '@/api/products';
import { AddToCartButton } from '@/components/AddToCartButton';

export function ProductDetail() {
  const { id } = useParams();
  const { data: product } = useQuery(['product', id], () => fetchProduct(id));
  // ...
}

// After: src/pages/product-detail/ui/product-detail-page.tsx
import { useParams } from 'react-router-dom';
import { useProduct } from '@/entities/product';
import { AddToCart } from '@/features/add-to-cart';
import { ProductReviews } from '@/widgets/product-reviews';

export function ProductDetailPage() {
  const { id } = useParams();
  const { data: product } = useProduct(id!);
  // ...
}
```

---

## Common Migration Patterns

### Handling Circular Dependencies

**Problem:** Existing code has circular imports.

**Solution:** Extract shared dependencies to lower layers.

```typescript
// Before: Circular dependency
// components/UserCard.tsx imports from hooks/useAuth.ts
// hooks/useAuth.ts imports from components/UserCard.tsx

// After: Break the cycle
// entities/user/ui/user-card.tsx — no auth dependency
// features/auth/model/auth.store.ts — no UserCard dependency
// pages/profile/ui/profile-page.tsx — composes both
```

### Handling Global State

**Problem:** Monolithic store accessed everywhere.

**Solution:** Split store by domain into entity/feature models.

```typescript
// Before: Monolithic store
export const store = configureStore({
  reducer: {
    user: userReducer,
    products: productsReducer,
    cart: cartReducer,
    auth: authReducer,
  },
});

// After: Distributed stores (Zustand example)
// entities/user/model/user.store.ts — user data
// entities/product/model/product.store.ts — product data
// features/cart/model/cart.store.ts — cart state
// features/auth/model/auth.store.ts — auth state
```

### Shared Components with Business Logic

**Problem:** Component has business logic mixed in.

**Solution:** Split into entity/feature UI and shared UI.

```typescript
// Before: ProductCard with add-to-cart logic
export function ProductCard({ product }) {
  const addToCart = useAddToCart();
  return (
    <div>
      <img src={product.image} />
      <h3>{product.name}</h3>
      <button onClick={() => addToCart(product)}>Add to Cart</button>
    </div>
  );
}

// After: Separated concerns
// entities/product/ui/product-card.tsx — display only
export function ProductCard({ product, actions }) {
  return (
    <div>
      <img src={product.image} />
      <h3>{product.name}</h3>
      {actions}
    </div>
  );
}

// features/add-to-cart/ui/add-to-cart-button.tsx — interaction
export function AddToCartButton({ product }) {
  const addToCart = useCartStore((s) => s.addItem);
  return <button onClick={() => addToCart(product)}>Add to Cart</button>;
}

// Composed in page/widget
<ProductCard
  product={product}
  actions={<AddToCartButton product={product} />}
/>
```

---

## Migration Checklist

- [ ] Create FSD directory structure
- [ ] Configure path aliases
- [ ] Migrate utilities to `shared/lib/`
- [ ] Migrate API client to `shared/api/`
- [ ] Migrate UI kit to `shared/ui/`
- [ ] Identify and extract entities
- [ ] Create entity public APIs
- [ ] Identify and extract features
- [ ] Create feature public APIs
- [ ] Migrate pages to page slices
- [ ] Extract reusable widgets
- [ ] Setup `app/` layer with providers
- [ ] Remove old directory structure
- [ ] Update documentation

---

## Rollback Strategy

Keep old structure working during migration:

```json
{
  "paths": {
    "@/*": ["./src/*"],
    "@components/*": ["src/components/*"],
    "@hooks/*": ["src/hooks/*"]
  }
}
```

Use feature flags to gradually switch:

```typescript
import { UserCard as LegacyUserCard } from '@components/UserCard';
import { UserCard as FSDUserCard } from '@/entities/user';

export const UserCard = process.env.USE_FSD ? FSDUserCard : LegacyUserCard;
```

---

## Resources

| Resource | Link |
|----------|------|
| Migration Guide | [feature-sliced.design/docs/guides/migration](https://feature-sliced.design/docs/guides/migration/from-custom) |
| v2.1 Changes | [Pages Come First](https://github.com/feature-sliced/documentation/releases/tag/v2.1) |
| Community Article | [Migrating a Legacy React Project](https://medium.com/@O5-25/migrating-a-legacy-react-project-to-feature-sliced-design-benefits-challenges-and-considerations-0aeecbc8b866) |
