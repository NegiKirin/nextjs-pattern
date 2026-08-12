---
name: nextjs-pattern
description: Use this skill to scaffold, organize, and review frontend-only Next.js App Router projects using Magic UI's hybrid structure by default and Feature-Sliced Design for large applications.
---

# Next.js Project Structure

Use this skill when creating or reorganizing a frontend-only Next.js App Router project, route, component, utility, hook, or business module.

This skill does not create backend code, `app/api/**/route.ts` handlers, database access, server-owned authentication, webhooks, queues, or backend-for-frontend endpoints. Call external backend APIs through frontend utilities in `lib/`.

This pattern has two modes:

1. **Hybrid structure** is the default for small and medium projects.
2. **Feature-Sliced Design (FSD)** is for large applications with multiple business domains and teams.

Start with hybrid. Do not introduce FSD layers, a state manager, a query library, or a new folder merely for consistency.

## Decide the Mode

Use the hybrid structure unless at least one is true:

- Multiple business domains need isolated ownership, such as products, cart, orders, users, and payments.
- Features combine entity UI, user actions, and page sections across multiple routes.
- Cross-domain dependencies are becoming hard to review.
- The project needs explicit layer boundaries for several developers or teams.

If none applies, use hybrid. Do not mix hybrid business folders and FSD layers for the same code.

## Hybrid Structure: Default

Use `src/` as the required application source root. Always create App Router files in `src/app/` and application code in `src/`; do not use a root-level `app/` directory.

```text
.
├── public/                      # static assets served directly
├── src/
│   ├── app/                     # frontend routes, layouts, special files
│   ├── components/              # reusable React components
│   │   └── ui/                  # reusable UI primitives, such as shadcn/ui
│   ├── hooks/                   # shared React hooks
│   ├── lib/                     # utilities, integrations, configuration
│   └── styles/                  # shared styles when not colocated in app
├── package.json
├── next.config.*
├── tsconfig.json
└── .env.*
```

- Keep `public/`, `package.json`, `next.config.*`, `tsconfig.json`, and `.env.*` at the repository root.
- If using `src/`, move other application folders such as `components/` and `lib/` inside it.
- Do not keep both `app/` and `src/app/`; Next.js ignores `src/app/` when root `app/` exists.
- Put global App Router CSS in `src/app/globals.css` and import it from the root layout. Do not create `src/styles/globals.css` unless the project deliberately configures and imports it.

### Hybrid Placement Rules

| Code | Location |
| --- | --- |
| URL route, layout, loading UI, error UI, metadata | `src/app/` |
| Component reused across routes or domains | `src/components/` |
| Reusable UI primitive | `src/components/ui/` |
| Shared React hook | `src/hooks/` |
| Utility, integration, configuration, auth helper | `src/lib/` |
| Global or reusable styles | `src/styles/` or `src/app/globals.css` |
| Asset needing a direct URL, favicon, downloadable file | `public/` |
| Local image used only by a component or feature | colocate and import it with `next/image` |
| UI/helper used by one route subtree only | colocate under that route, preferably in a `_` folder |

Do not move a route-local component to `components/` before another route needs it.

## App Router Rules

`app/` owns URL structure and Next.js special files.

```text
src/app/
├── (marketing)/                 # route group; absent from URL
│   └── page.tsx
├── (auth)/
│   ├── sign-in/
│   │   └── page.tsx
│   └── sign-up/
│       └── page.tsx
├── dashboard/
│   ├── _components/             # private route-local files
│   │   └── DashboardHeader.tsx
│   ├── loading.tsx
│   ├── error.tsx
│   └── page.tsx
├── globals.css
├── layout.tsx
├── not-found.tsx
└── page.tsx
```

- Use `page.tsx` for route UI and `layout.tsx` for shared or nested layout.
- Use `loading.tsx`, `error.tsx`, and `not-found.tsx` when the route needs those states.
- Do not create `route.ts`; this is a frontend-only pattern.
- Use `(group)` to organize routes without affecting the URL.
- Use `_components/`, `_lib/`, `_actions/`, or another `_` prefixed folder for private route files. Folders beginning with `_` are excluded from routing.
- Co-locate files within a route segment when they are only used by that segment. A folder becomes a page route when it contains `page.tsx`.
- Use default exports for React special files such as `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, and `not-found.tsx`.
- App Router components are Server Components by default. Add `'use client'` only when a component needs browser APIs, event handlers, effects, or client-only state.
- Keep client-component boundaries small so route UI does not ship unnecessary JavaScript.

Do not use route groups to choose SSR, SSG, ISR, or caching. Configure rendering and caching behavior per route.

## Naming and Imports

Use the project naming convention if one exists. For a new project, use:

- Next.js special files: lowercase exact conventions, such as `page.tsx`, `layout.tsx`, and `loading.tsx`.
- React components: PascalCase, such as `SignInForm.tsx` and `ProductCard.tsx`.
- Hooks: kebab-case with `use-` prefix, such as `use-local-storage.ts` and `use-auth-session.ts`.
- Utilities: descriptive kebab-case, such as `format-currency.ts`.
- Route folders: kebab-case, such as `order-history/` and `add-to-cart/`.
- FSD slice folders: preserve the project's convention; for a new FSD project, use the article's domain names such as `product/`, `addToCart/`, and `productCard/`.

Configure TypeScript aliases for source folders when the project uses `src/`:

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

Use aliases consistently:

```ts
import { Button } from '@/components/ui/Button';
import { formatCurrency } from '@/lib/format-currency';
import { useAuthSession } from '@/hooks/use-auth-session';
```

## Feature-Sliced Design: Large Applications Only

For a large application, organize `src/` into FSD layers:

```text
src/
├── app/                         # physical App Router: routes, layouts, providers, global styles
├── views/                       # FSD page-composition layer; not a Next.js router
├── widgets/                     # composed page sections
├── features/                    # user actions and business capabilities
├── entities/                    # business entities
└── shared/                      # reusable UI, utilities, API/config primitives
```

`src/app/` remains the physical Next.js router. Keep each `src/app/**/page.tsx` thin and compose the matching FSD page module from `src/views/`. Do not create `src/pages/`: Next.js reserves it for the legacy Pages Router.

### Layer Purpose

| Layer | Owns |
| --- | --- |
| `shared` | framework-independent utilities, reusable UI, API/config primitives |
| `entities` | domain objects such as user, product, order |
| `features` | user actions such as sign-in, add-to-cart, checkout |
| `widgets` | composed sections that combine features and entities |
| `views` | route-level page compositions; implementation name for FSD's conceptual `pages` layer |
| `app` | providers, global styles, and Next.js routing setup |

Each layer may depend only on layers below it:

```text
app -> views -> widgets -> features -> entities -> shared
```

A feature must not import a widget or view. An entity must not import a feature, widget, view, or app module. `shared/` must not import from any higher layer.

Within an FSD slice, create only the segments that exist:

```text
src/features/addToCart/
├── model/
└── ui/
```

```text
src/entities/product/
├── model/
└── ui/
```

Use `ui/` for components and `model/` for state, types, and business logic. Add `<slice>/api/` only when that domain needs data access. When another slice or layer consumes a slice, add its `index.ts` and import only through that public API. A slice used only internally does not need `index.ts`.

```ts
// src/features/addToCart/index.ts
export { AddToCartButton } from './ui/AddToCartButton';

// consumer
import { AddToCartButton } from '@/features/addToCart';
import { ProductPrice } from '@/entities/product';
```

## External API Access

This is a frontend-only pattern.

- In hybrid mode, put external API clients, integrations, and configuration in `src/lib/`.
- In FSD mode, put shared API-client primitives and configuration in `src/shared/api/` or `src/shared/config/`; put domain-specific requests in that domain's `<slice>/api/`.
- Consume external API results from components, hooks, or FSD slices.

Never create `app/api/**/route.ts`, database code, webhooks, queues, or server-owned authentication in this pattern. Do not put private backend credentials in `NEXT_PUBLIC_` environment variables.

## Scaffold Workflow

1. Use `src/` as the only application source root. If a project has root-level application folders, stop and ask before restructuring it.
2. Inspect whether the project uses hybrid or FSD. Preserve the established mode.
3. Determine whether the requested code is route-local, reusable, or a business capability.
4. Use the placement rules for the selected mode.
5. Create the fewest files and folders required; do not create empty conventional folders.
6. Add aliases only when imports already justify them.
7. Run the smallest relevant project check: typecheck, test, lint, or build.

## Review Checklist

- `src/` is the only application source root; no root-level `app/`, `components/`, `hooks/`, `lib/`, or `styles/` folders exist.
- The project uses one mode per area: hybrid by default or FSD when justified.
- `src/app/` contains routing, layouts, special files, and composition; route-local code is colocated.
- In FSD mode, `src/app/**/page.tsx` composes the matching `src/views/` module; no `src/pages/` directory exists.
- Server Components remain the default; `'use client'` is limited to interactive or browser-only boundaries.
- Route groups do not appear in URL assumptions.
- Private `_` folders do not become routes.
- Hybrid reusable UI is in `components/` and primitives are in `components/ui/`.
- Hybrid utilities, integrations, and configuration are in `lib/`; shared hooks are in `hooks/`.
- FSD dependencies flow only downward: `app -> views -> widgets -> features -> entities -> shared`.
- A slice imported by another slice or layer exposes `index.ts`; consumers do not import its internals.
- No `app/api/**/route.ts` handler or other backend code was introduced.
- No unnecessary FSD layer, library, state manager, or empty folder was introduced.
