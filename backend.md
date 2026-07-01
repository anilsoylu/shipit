# Backend Playbook

Default backend: standalone Fastify TypeScript API on Coolify/VDS.

## Architecture

```txt
Expo app -> Fastify API -> Postgres
                     -> Redis
                     -> Auth provider verification (default Clerk; see auth.md)
                     -> Stripe, RevenueCat webhooks, AI providers, email, storage
```

The Expo app never talks directly to databases, Redis, private payment APIs, or model providers.

## API Shape

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

Rules:

- `GET /health` must work without auth.
- Protected routes must verify the auth provider's session/JWT server-side (default Clerk; see `auth.md`).
- Do not accept `userId` from client input for ownership.
- Derive actor identity from verified auth context.
- Validate request bodies before DB writes.
- Return typed DTOs, not raw DB rows when fields are sensitive.

## Auth

Auth is pluggable. Default is Clerk. Pick a provider and follow its playbook;
every provider must satisfy the Auth Swap Contract. See `auth.md`.

Provider-agnostic API rules (hold regardless of provider):

- The API verifies incoming authenticated requests server-side.
- Derive actor identity only from the verified token, never from client input.
- Map the provider user id to internal user records when the product needs
  app-specific profile data.
- Webhooks must verify signatures before mutating user records.

## Database

- Use Drizzle schema as source of truth.
- Generate migrations for every schema change.
- Keep migrations committed.
- Run migrations before production deployment.
- Use transactions for multi-table writes.
- Prefer explicit indexes on foreign keys, lookup fields, and uniqueness constraints.

## Redis

Use Redis for:

- Rate limits.
- AI credit counters.
- Short-lived locks.
- Queue state.
- Cache only when recomputation or DB reads become expensive.

Do not use Redis as the source of truth for durable product data.

## Coolify Deploy

API deploy requirements:

- Dockerfile or Coolify-supported build config.
- Health check route.
- Explicit env var list.
- Migration command documented.
- Postgres and Redis services attached through private networking when available.
- Logs visible in Coolify.

Deployment order:

1. Provision Postgres and Redis.
2. Configure API env vars.
3. Run migrations.
4. Deploy API.
5. Smoke test `/health`.
6. Smoke test one protected endpoint.
7. Point Expo app env to deployed API URL.

## Security Defaults

- Use HTTPS for public API.
- Set CORS to the expected clients, not `*`, once domains are known.
- Never log tokens, cookies, payment payload secrets, provider keys, or full auth headers.
- Store webhook raw body when required for signature verification.
- Add rate limits before public expensive endpoints.

## Official References

- Fastify docs: https://fastify.dev/docs/latest/
- Clerk backend requests: https://clerk.com/docs/backend-requests/overview
- Drizzle PostgreSQL: https://orm.drizzle.team/docs/get-started/postgresql-new
- Drizzle migrations: https://orm.drizzle.team/docs/migrations
- Coolify docs: https://coolify.io/docs
