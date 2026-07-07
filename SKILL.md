---
name: nextjs-pattern
description: Use this skill to analyze, scaffold, and review the sample-compatible Next.js feature-based modular architecture with App Router, API client layers, React Query, Zustand auth/session, and frontend/fullstack boundaries.
---

# nextjs-pattern

Use this skill when the user asks to work with a Next.js modular pattern, including:

- deciding whether a Next.js repo is frontend-only or fullstack
- scaffolding a new domain feature
- reviewing `app/`, `features/`, `core/`, `shared/`, `lib`, and `types` boundaries
- checking API client, React Query hooks, Zustand stores, or App Router pages
- converting an ad-hoc Next.js project toward the sample pattern

## Pattern Identity

This is a frontend-first Next.js App Router pattern inspired by React Native feature modules.

Main flow:

```text
feature API -> React Query hook or auth store action -> component/page
```

Use it for admin panels, dashboards, and frontend apps that call an external backend API.

## Core Rule

`app/` should stay thin: routes, layouts, pages, route handlers, and composition. Business logic belongs in `features/` or shared infrastructure.

## Standard Structure

```text
src/
├── app/         # App Router routes
├── features/    # business domain modules
├── shared/      # reusable UI and utilities
├── core/        # infrastructure: API client, env, config
├── lib/         # third-party setup/wrappers
└── types/       # global TypeScript types
```

## Dependency Rules

Preferred boundary:

```text
app/       -> features/ via index.ts, shared/, lib/
features/  -> core/, shared/, lib/
shared/    -> lib/
core/      -> standalone when practical
lib/       -> standalone
```

Sample-compatible shortcut: small FE-only admin apps may wire auth by letting `core/api/client.ts` read `features/auth/store/authStore` for token injection and 401 logout.

Preferred upgrade: expose `setApiClientAuth(...)` from the API client and wire auth from `features/auth` so `core/` does not import `features/`.

Use the shortcut for simple frontend shells. Use the upgrade when the app grows, auth changes often, tests need isolation, or the API client may run in server contexts.

## Feature Module Shape

```text
features/{feature-name}/
├── api/           # {feature}Api.ts: HTTP calls only
├── hooks/         # use{Feature}.ts: React Query hooks
├── components/    # feature UI
├── store/         # optional Zustand client/session state
├── types/         # {feature}.types.ts
└── index.ts       # public barrel export
```

## Scaffolding Order

1. Create feature folders if missing.
2. Add feature types in `features/{name}/types/`.
3. Add endpoint constants in `core/api/endpoints.ts`.
4. Add API functions in `features/{name}/api/`.
5. Add React Query hooks in `features/{name}/hooks/` for CRUD server data.
6. Add components in `features/{name}/components/`.
7. Export the public API from `features/{name}/index.ts`.
8. Compose the page in `app/(group)/{name}/page.tsx`.
9. Add navigation only if the user requested a visible route.

Detailed examples: [references/scaffolding.md](references/scaffolding.md).

## Import Rules

```ts
// Good: outside a feature, import through its public barrel.
import { ProductList, useProducts } from '@/features/products';

// Avoid: outside a feature, do not import internals; export intentional public APIs from the barrel.
import { productApi } from '@/features/products/api/productApi';

// Good: inside the same feature, relative internal imports are fine.
import { productApi } from '../api/productApi';
```

## Server Component Exception

Client components should use feature hooks.

Server Components may call feature API functions directly for server-side data fetching. Prefer importing APIs from the feature barrel when exported:

```ts
import { productApi } from '@/features/products';
```

Avoid internal imports like `@/features/products/api/productApi` unless that API is intentionally not part of the public barrel.

## Export Rules

- Use named exports for components, hooks, APIs, stores, and utilities.
- Use `export default` only where Next.js requires it: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, etc.
- Keep each feature's `index.ts` as the public API. Export internals only when another layer truly needs them.

## State Rules

- Server data: TanStack Query.
- Auth/session state: Zustand with `persist` is acceptable.
- Client UI state: Zustand only when state must be shared across components or routes.
- Do not move ordinary server lists/details into Zustand when React Query owns them.
- Mutations should invalidate the relevant query keys.

## Auth Store Exception

The sample keeps `user`, `token`, `isAuthenticated`, `loading`, `login`, `register`, and `logout` in `features/auth/store/authStore.ts` using Zustand persist.

This is sample-compatible for frontend-only admin apps. For stricter architecture, keep the store as session state and move API-client auth wiring to callback injection.

## FE-only vs Fullstack Boundary

A project with pages plus Axios/fetch clients that call an external API is frontend-only, even if it uses the App Router.

Do not classify a project as fullstack just because docs mention `app/api`. Only classify it as fullstack when actual `app/api/**/route.ts` files implement backend behavior such as database access, auth/session server code, queues, webhooks, or durable mutations.

Detailed checklist: [references/fullstack-boundary.md](references/fullstack-boundary.md).

## Navigation

In the sample, main navigation lives in `shared/components/layout/Sidebar.tsx`. When adding a visible route under `app/(main)`, update `menuItems` only if requested.

## Review Checklist

Before approving a Next.js pattern change, verify:

- `app/` is thin and composes features.
- cross-feature imports go through `features/{name}/index.ts`.
- `core -> features/auth/store` is treated as a sample shortcut, not a scalable default.
- API files use endpoint constants, not hardcoded repeated URLs.
- components handle loading, error, and empty states where data is async.
- mutations invalidate precise query keys.
- Zustand is used for auth/session or true client state, not ordinary server lists.
- no default exports outside Next.js route convention files.

Detailed checklist: [references/review-checklist.md](references/review-checklist.md).

## Reference Files

- [references/architecture.md](references/architecture.md) — layer rules and dependency boundaries.
- [references/scaffolding.md](references/scaffolding.md) — complete feature scaffolding examples.
- [references/core-infrastructure.md](references/core-infrastructure.md) — API client, env, endpoints, and interceptor pattern.
- [references/fullstack-boundary.md](references/fullstack-boundary.md) — FE-only vs fullstack classification.
- [references/review-checklist.md](references/review-checklist.md) — architecture review checklist.

## Example Project

A local sample app may live at `nextjs-base-modular-partent-main/`. It is ignored by Git and used as the observed pattern. Reference files document the Claude-friendly version of that pattern while preserving sample-compatible behavior.
