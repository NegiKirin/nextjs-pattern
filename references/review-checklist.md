# Review Checklist

Use this checklist when reviewing a Next.js feature-based modular project.

## Structure

- [ ] `src/app` contains routes, layouts, pages, and route handlers only.
- [ ] `src/features` contains domain modules.
- [ ] `src/shared` contains reusable UI/utilities only.
- [ ] `src/core` contains infrastructure only.
- [ ] `src/lib` contains third-party setup/wrappers.

## Imports

- [ ] External callers import features through `features/{name}/index.ts`.
- [ ] Feature internals use relative imports inside the same feature.
- [ ] No feature imports another feature's internal `api/`, `hooks/`, `store/`, or `components/` path.
- [ ] `core -> features/auth/store` is recognized as the sample-compatible auth shortcut, not the scalable default.
- [ ] `shared/` does not import from `features/`.

## API Layer

- [ ] API calls live in `features/{name}/api/{name}Api.ts`.
- [ ] API files use `core/api/client.ts`.
- [ ] Repeated URLs are centralized in `core/api/endpoints.ts`.
- [ ] API functions return typed data.
- [ ] API functions do not contain UI behavior.

## React Query

- [ ] GET reads use `useQuery`.
- [ ] writes use `useMutation`.
- [ ] query keys are feature-prefixed.
- [ ] detail queries use stable IDs in keys.
- [ ] mutations invalidate the relevant list/detail keys.
- [ ] conditional queries use `enabled` instead of branching in components.

## Components

- [ ] Async components handle loading, error, and empty states.
- [ ] feature components do not call API functions directly when hooks exist.
- [ ] shared UI components remain domain-agnostic.
- [ ] `className` composition uses the project utility when available.

## State

- [ ] Zustand stores are only used for cross-component client state.
- [ ] local component state stays local.
- [ ] persisted Zustand state is used only when reload persistence is required.

## Exports

- [ ] Named exports are used for components, hooks, APIs, and utilities.
- [ ] Default exports are limited to Next.js convention files.
- [ ] `index.ts` exports only the intended public surface.

## Boundary Classification

- [ ] FE-only/fullstack classification is stated with evidence.
- [ ] `app/api` presence is checked before calling a project fullstack.
- [ ] database/auth/server persistence evidence is checked before calling a project backend-capable.
