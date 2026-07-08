# Backend Playbook: Hono

Use Hono when: you want a TypeScript API that stays portable across Node, Bun,
and edge runtimes (Cloudflare Workers, Deno, Lambda). Pairs naturally with
better-auth (TS) from `../auth.md`.

## Project layout

```txt
src/
  index.ts
  env.ts
  routes/
    health.ts
    me.ts
    webhooks.ts
  middleware/
    auth.ts
  db/
    schema.ts
    migrations/
```

## Libraries

- HTTP framework: Hono (Web-Standards, runs on any JS runtime).
- DB access/ORM: Drizzle over Postgres.
- Migration tool: drizzle-kit (`drizzle-kit generate`, `drizzle-kit migrate`).
- Validation: `@hono/zod-validator` wrapping `zod`.
- Redis client: `ioredis` on Node/Bun (or an HTTP Redis like Upstash on edge).

## Auth verification

- Verify the token per `../auth.md`'s Auth Swap Contract.
- Hono ships a native `jwk` middleware that fetches the provider's `jwks_uri`,
  matches by `kid`, rejects HS\* algorithms, and validates `exp`/`nbf`/`iat` — no
  extra library needed for JWKS-based providers.
- Alternatively verify with `jose` (`createRemoteJWKSet` + `jwtVerify`).
- With better-auth, verify its JWT against the better-auth JWKS endpoint.

## Database & migrations

- Drizzle schema is the source of truth.
- `drizzle-kit generate` to emit SQL; `drizzle-kit migrate` to apply.
- Run migrate before production deploy.

## Health, webhooks, security

- `GET /health` with no auth.
- Webhook routes verify provider signatures against the raw body before mutating.
- Hono CORS middleware scoped to known clients; rate-limit middleware before
  expensive endpoints.

## Coolify deploy

- On a VDS: Node or Bun Dockerfile running the Hono server; expose `/health`.
- Run the drizzle-kit migrate command as a release step.

## Env vars

- Public (bundle-safe): none server-side.
- Secret (server only): `DATABASE_URL`, `REDIS_URL`, auth provider secrets, plus
  any payment/AI keys in scope.

## Contract compliance

1. `GET /health` responds without auth. ✓
2. Protected routes verify the token via the `jwk` middleware / `jose`. ✓
3. No trusted values from client input; identity from the verified token. ✓
4. `@hono/zod-validator` validates bodies/params before DB writes. ✓
5. Typed DTOs returned, not raw rows, for sensitive fields. ✓
6. drizzle-kit migrations committed + run pre-deploy; transactions; indexes. ✓
7. Webhooks verify signatures against the raw body before mutating. ✓
8. Secrets stay in server env only. ✓
9. Dockerfile + `/health` + env list + documented migrate command on Coolify. ✓
10. HTTPS, scoped CORS, rate limits, no secret logging. ✓

## Official References

- Hono docs: https://hono.dev/
- Hono LLM docs: https://hono.dev/llms.txt (full: https://hono.dev/llms-full.txt)
- Hono JWK middleware: https://hono.dev/docs/middleware/builtin/jwk
- Drizzle migrations: https://orm.drizzle.team/docs/migrations
