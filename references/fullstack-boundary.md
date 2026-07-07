# FE-only vs Fullstack Boundary

## Classify as FE-only

A Next.js project is frontend-only when it has:

- App Router pages and layouts
- feature modules and UI components
- Axios/fetch clients that call an external API
- environment variables such as `NEXT_PUBLIC_API_URL`
- mock data or display-only dashboards

This remains FE-only even when it has client-side auth state, React Query, or Zustand.

## Classify as fullstack

Do not classify a project as fullstack just because documentation mentions `app/api`. Treat it as fullstack only when actual files contain real backend behavior:

- `src/app/api/**/route.ts` or `app/api/**/route.ts`
- database clients, ORM schemas, migrations, or seed scripts
- server-side auth/session handling
- webhooks, queues, cron jobs, or background jobs
- server actions that mutate durable data
- file upload/storage logic
- direct calls to internal backend services from server code

## Quick Decision Tree

1. Does the repo contain real backend behavior: route handlers, server actions, database access, server-side auth, queues, or durable mutations?
   - No: likely FE-only.
   - Yes: continue.
2. Is that behavior only a proxy/mock/display shell without durable business logic?
   - Yes: still likely FE-only shell/demo.
   - No: fullstack.
3. If unsure, check for database/auth/server infrastructure before calling the repo backend-capable.

## Recommended Answer Format

```text
This repo is frontend-only with an API client layer.
Evidence:
- pages live in `src/app`
- API calls go through `core/api/client.ts`
- no `app/api/**/route.ts` backend implementation found
- no database/migration/server persistence layer found
```

If fullstack:

```text
This repo is fullstack Next.js.
Evidence:
- route handlers exist in `src/app/api/.../route.ts`
- backend code accesses durable data through ...
- server-side auth/session logic exists in ...
```
