# Backend Playbook

Default backend is standalone Fastify TypeScript on Coolify/VDS. The backend is
pluggable — pick a stack, then follow its playbook. Whatever you pick must
satisfy the Backend Swap Contract below.

## Architecture

```txt
Expo app  \
           -> Backend API -> Postgres
Web app   /          -> Redis
                     -> Auth provider verification (default Clerk; see auth.md)
                     -> Stripe, RevenueCat webhooks, AI providers, email, storage
```

Clients (Expo app, web client code) never talk directly to databases, Redis, private payment APIs, or model providers. Web-only exception: for the web-only fullstack shape in `web.md`, Next.js server-side code IS the backend and must satisfy this contract like any other stack.

## Choosing a backend

- **Fastify** (TS, default): low boilerplate, richest plugin ecosystem, fast.
- **Hono** (TS): portable across Node/Bun/edge runtimes; pairs with better-auth.
- **Go**: performance, single binary, strong concurrency.
- **Rust**: maximum performance and safety.
- **Python/FastAPI**: Python ecosystem, ML-adjacent work.

## Backend Swap Contract

Every backend, in any language, must satisfy these:

1. `GET /health` responds without auth.
2. Protected routes verify the auth token **server-side on every request**, per
   the Auth Swap Contract in `auth.md` (JWKS / provider SDK / introspection).
3. Never accept `userId`, ownership, price, credits, entitlement, or role from
   client input; derive identity from the verified token, other trusted values
   from server config or DB.
4. Validate request bodies/params before any DB write.
5. Return typed DTOs, not raw DB rows, when fields are sensitive.
6. Schema changes go through committed migrations, run before production deploy;
   use transactions for multi-table writes; index foreign keys, lookup fields,
   and uniqueness constraints.
7. Webhooks verify the provider signature before mutating state; store the raw
   body when signature verification needs it.
8. Secrets (DB URL, Redis URL, provider keys, webhook secrets) live in server
   env only, never in the Expo bundle or web client bundle.
9. Deployable on Coolify/VDS: a Dockerfile (or Coolify-supported build), the
   `/health` route, an explicit env var list, a documented migration command,
   and logs visible in Coolify.
10. HTTPS in production; CORS scoped to known clients (not `*`); rate limits
    before expensive public endpoints; never log tokens, cookies, payment
    secrets, provider keys, or full auth headers.

## Auth

Auth is pluggable. Default is Clerk. Pick a provider and follow its playbook;
every provider must satisfy the Auth Swap Contract. See `auth.md`.

Provider-agnostic API rules (hold regardless of provider):

- The API verifies incoming authenticated requests server-side.
- Derive actor identity only from the verified token, never from client input.
- Map the provider user id to internal user records when the product needs
  app-specific profile data.
- Webhooks must verify signatures before mutating user records.

## Shared guidance

Language-agnostic. Each playbook names the concrete tool (Drizzle vs sqlc vs
SQLAlchemy, etc.).

### Database

For ORM, managed-host, on-device, and background-job options, see `data.md`; the
rules below are server-side and provider-agnostic.

- Use the ORM/schema as the source of truth.
- Generate migrations for every schema change.
- Keep migrations committed.
- Run migrations before production deployment.
- Use transactions for multi-table writes.
- Prefer explicit indexes on foreign keys, lookup fields, and uniqueness constraints.

### Redis

Use Redis for:

- Rate limits.
- AI credit counters.
- Short-lived locks.
- Queue state.
- Cache only when recomputation or DB reads become expensive.

Do not use Redis as the source of truth for durable product data.

### Coolify Deploy

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
7. Point client env (Expo app, web app) to the deployed API URL.

### Security Defaults

- Use HTTPS for public API.
- Set CORS to the expected clients, not `*`, once domains are known.
- Never log tokens, cookies, payment payload secrets, provider keys, or full auth headers.
- Store webhook raw body when required for signature verification.
- Add rate limits before public expensive endpoints.

## Add a new backend

Copy this skeleton into `backend-<stack>.md` and fill it in:

```markdown
# Backend Playbook: <Stack>

Use <Stack> when: <selection criteria>.

## Project layout
- Directory/module structure (equivalent to the Fastify project layout).

## Libraries
- HTTP framework, DB access/ORM, migration tool, validation, Redis client.

## Auth verification
- How this stack verifies the token per auth.md's Auth Swap Contract
  (e.g. JWKS validation library, provider SDK).

## Database & migrations
- ORM/driver and the exact migration command (goes in the deploy runbook).

## Health, webhooks, security
- Health route, webhook signature verification, CORS/rate-limit approach.

## Coolify deploy
- Dockerfile notes specific to this stack; build/run commands; env list.

## Env vars
- Secret vs public.

## Contract compliance
- Restate the 10 Backend Swap Contract invariants; confirm each is met.

## Official References
- Framework, ORM, migration, and JWKS-verify docs + llms.txt (from the registry).
```

To support a stack not listed, copy the skeleton into `backend-<stack>.md`,
satisfy every contract invariant, add its rows to the LLM Docs Registry in
`skills.md`, and link it from this file and the index files.

## Backend playbooks

- `backend-fastify.md` — Fastify + Drizzle (TS, default).
- `backend-hono.md` — Hono + Drizzle (TS, portable/edge).
- `backend-go.md` — chi + pgx/sqlc + golang-migrate.
- `backend-rust.md` — axum + sqlx.
- `backend-python.md` — FastAPI + SQLAlchemy/Alembic.

## Official References

- Drizzle PostgreSQL: https://orm.drizzle.team/docs/get-started/postgresql-new
- Drizzle migrations: https://orm.drizzle.team/docs/migrations
- Coolify docs: https://coolify.io/docs
