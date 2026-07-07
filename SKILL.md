---
name: nextjs-pattern
description: Analyze, scaffold, and review Next.js frontend feature-based modular architecture with App Router, API client layer, React Query, and Zustand. Use when the user asks about Next.js base modular patterns, FE-only vs fullstack classification, or adding frontend features to this pattern.
version: 1.0.0
source: local-pattern-analysis
reference_project: nextjs-base-modular-partent-main
---

# Next.js Pattern

Use this skill for Next.js projects following a **frontend feature-based modular pattern** like `nextjs-base-modular-partent-main`.

## First Decision: FE-only or Fullstack?

Classify the repo before changing anything.

### FE-only evidence

Treat the project as **frontend-only** when most evidence matches:

- `package.json` has Next/React/client state dependencies but no DB/ORM/backend dependencies.
- API code is an HTTP client layer, usually Axios or `fetch`.
- Env uses public API config like `NEXT_PUBLIC_API_URL`.
- Feature APIs call endpoints such as `/auth/login` or `/products`.
- No real route handlers under `src/app/api/**/route.ts`.
- No server-side persistence: Prisma/Drizzle/schema/migrations/repositories are absent.

Recommended wording:

> This is a Next.js frontend base structure. It has an API client layer for calling a backend, but it does not implement the backend itself.

### Fullstack evidence

Treat the project as **fullstack** only when it has real backend implementation:

- `src/app/api/**/route.ts` or equivalent Next.js route handlers.
- Server actions that mutate server-side data.
- DB/ORM files: Prisma, Drizzle, SQL migrations, repository implementations.
- Server-side auth/session/token handling.
- Validation at route boundaries.
- Service/controller/repository layers that own business logic.

Do not call a repo fullstack just because the README mentions `app/api`; verify files exist.

## Target Architecture

Use this structure for FE-only modular Next.js apps:

```text
src/
├── app/                  # App Router pages/layouts/route groups
│   ├── (auth)/           # Auth pages
│   ├── (main)/           # Main/protected pages
│   ├── layout.tsx
│   └── page.tsx
├── core/                 # App infrastructure
│   ├── api/              # API client, endpoints
│   └── config/           # env/config
├── features/             # Domain modules
│   └── {feature}/
│       ├── api/          # HTTP calls for this feature
│       ├── components/   # Feature UI
│       ├── hooks/        # React Query hooks
│       ├── store/        # Zustand store when shared client state is needed
│       ├── types/        # Feature types
│       └── index.ts      # Public exports
├── lib/                  # Third-party setup
├── shared/               # Shared UI/utilities
└── types/                # Global types
```

## API Pattern

Use this flow:

```text
feature API function → React Query hook → component/page
```

Rules:

- Keep endpoint strings in `src/core/api/endpoints.ts`.
- Keep Axios/fetch setup in `src/core/api/client.ts`.
- Add auth token/interceptors in the client layer, not every feature.
- Components use hooks; avoid direct API calls from components.
- Use React Query for server state.
- Use Zustand only for client/UI/session state that must be shared.
- Keep request/response types in the feature `types/` folder.

Example shape:

```ts
// src/features/products/api/productApi.ts
export const productApi = {
  getProducts: async (filter?: ProductFilter): Promise<Product[]> => {
    const { data } = await apiClient.get<Product[]>(API_ENDPOINTS.PRODUCTS.LIST, {
      params: filter,
    })
    return data
  },
}
```

```ts
// src/features/products/hooks/useProducts.ts
export function useProducts(filter?: ProductFilter) {
  return useQuery({
    queryKey: ['products', filter],
    queryFn: () => productApi.getProducts(filter),
  })
}
```

## Adding a Feature

Before coding, check whether the feature already fits an existing module.

Minimum steps:

1. Add types in `src/features/{feature}/types/`.
2. Add endpoints to `src/core/api/endpoints.ts`.
3. Add API calls in `src/features/{feature}/api/`.
4. Add hooks in `src/features/{feature}/hooks/`.
5. Add components in `src/features/{feature}/components/`.
6. Export only the public surface from `src/features/{feature}/index.ts`.
7. Add pages under `src/app/...` only for routes.

Keep the diff small. Do not add stores, providers, schemas, or route handlers unless requested or required.

## When User Asks for Backend

If the existing project is FE-only, say so and ask whether they want:

1. Frontend integration with an external backend API.
2. Conversion to fullstack Next.js using `src/app/api/**/route.ts`.
3. Separate backend service outside Next.js.

Do not silently create backend architecture in a FE-only pattern.

## Review Checklist

Use this checklist when reviewing or scaffolding:

- App Router files stay in `src/app/`.
- Domain code stays in `src/features/{feature}/`.
- Shared code is genuinely reused before moving to `src/shared/`.
- No component calls raw endpoint strings directly.
- No duplicate endpoint constants.
- No Zustand store for data React Query already owns.
- No server-only secrets exposed through `NEXT_PUBLIC_*`.
- No claim of fullstack without actual API routes and persistence.

## Common Answer: Is This Full BE + FE?

For the reference project, answer:

> It is frontend-only. It is a Next.js App Router frontend pattern with API client, endpoint constants, feature modules, React Query, and Zustand. It calls backend APIs but does not implement backend routes, database, repositories, or server-side auth.
