# Architecture Reference

## Goal

Keep routing thin and keep business behavior inside feature modules.

## Layers

```text
src/
├── app/         # routes, layouts, pages, route handlers
├── features/    # business domains
├── shared/      # reusable UI and utilities
├── core/        # infrastructure and app configuration
├── lib/         # third-party setup/wrappers
└── types/       # global types
```

## Dependency Direction

```text
app/       -> features/ via public barrels, shared/, lib/
features/  -> core/, shared/, lib/
shared/    -> lib/
core/      -> standalone when practical
lib/       -> standalone
```

## Layer Rules

### `app/`

Allowed:

- route groups
- `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`
- `route.ts` handlers when the project is intentionally fullstack
- composition of feature components

Avoid:

- domain business logic
- direct API client orchestration when a feature hook/API exists
- reusable UI components

### `features/`

Each feature owns one business domain and exposes a public API through `index.ts`.

Allowed:

- domain components
- domain hooks
- domain API functions
- domain types
- optional Zustand store

Avoid:

- importing internals from another feature
- exporting every internal file by default
- mixing unrelated domains in one feature

### `core/`

Infrastructure used by features.

Allowed:

- API client
- endpoint constants
- environment configuration
- error normalization

Avoid by default:

- importing from `features/`
- reading Zustand feature stores directly outside the sample-compatible auth shortcut
- UI code

### `shared/`

Cross-feature reusable code.

Allowed:

- base UI components
- layout components
- shared utilities

Avoid:

- domain-specific logic
- imports from `features/`

## Public API Rule

Outside a feature, import from its barrel:

```ts
import { ProductList, useProducts } from '@/features/products';
```

Inside the same feature, relative imports are fine:

```ts
import { productApi } from '../api/productApi';
```

## Source of Truth

The sample project is the observed pattern. This reference documents the Claude-friendly version of that pattern while preserving sample-compatible behavior and marking stricter boundaries as upgrades.
