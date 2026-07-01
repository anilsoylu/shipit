# Backend Playbook: Fastify

Use Fastify when: you want the default — TypeScript, low boilerplate, the richest
plugin ecosystem, fast endpoints. This is the reference the other playbooks mirror.

## Project layout

Recommended `apps/api` layout:

```txt
src/
  index.ts
  env.ts
  server.ts
  plugins/
    auth.ts
    db.ts
    redis.ts
  routes/
    health.ts
    me.ts
    webhooks.ts
  db/
    schema.ts
    migrations/
```

## Libraries

- HTTP framework: Fastify.
- DB access/ORM: Drizzle over Postgres.
- Migration tool: drizzle-kit (`drizzle-kit generate`, `drizzle-kit migrate`).
- Validation: zod (or Fastify's JSON schema) on request bodies/params.
- Redis client: `ioredis` (or `redis`) in a Fastify plugin.

## Auth verification

- Default provider is Clerk; verify per `auth.md`'s Auth Swap Contract.
- Shape: an `auth.ts` plugin reads the bearer token, verifies it via Clerk's
  backend SDK / JWKS, and decorates the request with the verified `userId`.
- Swapping provider (better-auth, Supabase, custom JWT) only changes this plugin;
  see the matching `auth-*.md`.

## Database & migrations

- Drizzle schema in `src/db/schema.ts` is the source of truth.
- Generate migrations with `drizzle-kit generate`; apply with `drizzle-kit migrate`.
- Run the migrate command before production deploy (documented in the runbook).

## Health, webhooks, security

- `routes/health.ts`: `GET /health` with no auth.
- `routes/webhooks.ts`: verify provider signatures (Stripe, RevenueCat, Clerk)
  against the raw body before mutating state.
- CORS scoped to known clients; `@fastify/rate-limit` before expensive endpoints.

## Coolify deploy

- Node Dockerfile (or Coolify Nixpacks build) running the compiled server.
- Expose `/health` for the Coolify health check.
- Env list below; run the migrate command as a release step before deploy.

## Env vars

- Public (bundle-safe): none server-side — the API base URL lives in the Expo app.
- Secret (server only): `DATABASE_URL`, `REDIS_URL`, auth provider secrets (see
  `auth.md`), plus any Stripe/RevenueCat/AI keys in scope.

## Contract compliance

1. `GET /health` responds without auth. ✓
2. Protected routes verify the token server-side via the `auth.ts` plugin. ✓
3. No `userId`/price/role from client input; identity from the verified token. ✓
4. Request bodies/params validated (zod/JSON schema) before DB writes. ✓
5. Typed DTOs returned, not raw rows, for sensitive fields. ✓
6. drizzle-kit migrations committed + run pre-deploy; transactions; indexes. ✓
7. Webhooks verify signatures against the raw body before mutating. ✓
8. Secrets stay in server env only. ✓
9. Dockerfile + `/health` + env list + documented migrate command on Coolify. ✓
10. HTTPS, scoped CORS, rate limits, no secret logging. ✓

## Official References

- Fastify docs: https://fastify.dev/docs/latest/
- Fastify LLM docs: https://fastify.dev/llms.txt
- Clerk backend requests: https://clerk.com/docs/backend-requests/overview
- Drizzle PostgreSQL: https://orm.drizzle.team/docs/get-started/postgresql-new
- Drizzle migrations: https://orm.drizzle.team/docs/migrations
