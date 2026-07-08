# Backend Playbook: Go

Use Go when: you want high performance, a single deployable binary, and strong
concurrency for the API.

## Project layout

```txt
cmd/api/main.go
internal/
  server/       # router + middleware wiring
  auth/         # JWKS verification
  handlers/     # health, me, webhooks
  db/           # sqlc-generated queries
  redis/
migrations/     # golang-migrate .sql files
sqlc.yaml
Dockerfile
```

## Libraries

- HTTP router: chi (`github.com/go-chi/chi/v5`).
- Postgres: pgx (`github.com/jackc/pgx/v5`) + sqlc for type-safe queries.
- Migration tool: golang-migrate (CLI).
- Validation: struct validation (e.g. `go-playground/validator`) on decoded bodies.
- Redis client: `github.com/redis/go-redis/v9`.

## Auth verification

- Verify the token per `../auth.md`'s Auth Swap Contract via JWKS.
- Use a maintained JWKS library: `github.com/MicahParks/keyfunc/v3` (builds a
  `jwt.Keyfunc` for golang-jwt) or `github.com/lestrrat-go/jwx/v4` (jwk package).
- Middleware fetches the provider JWKS, matches the `kid`, verifies signature +
  claims, and puts the verified user id in the request context.

## Database & migrations

- sqlc generates type-safe Go from raw SQL; pgx is the driver.
- Migrations via golang-migrate:
  `migrate -source file://migrations -database "$DATABASE_URL" up`
- Run before production deploy. Use `pgx` transactions for multi-table writes.

## Health, webhooks, security

- `GET /health` handler with no auth.
- Webhook handlers verify provider signatures against the raw body before mutating.
- chi CORS middleware scoped to known clients; a rate-limit middleware before
  expensive endpoints.

## Coolify deploy

- Multi-stage Dockerfile: build the static binary, copy into a minimal base
  (distroless/alpine). Expose `/health`.
- Run the golang-migrate command as a release step.

## Env vars

- Public (bundle-safe): none server-side.
- Secret (server only): `DATABASE_URL`, `REDIS_URL`, auth provider secrets, plus
  payment/AI keys in scope.

## Contract compliance

1. `GET /health` responds without auth. ✓
2. Protected routes verify the JWT via keyfunc/jwx JWKS middleware. ✓
3. No trusted values from client input; identity from the verified token. ✓
4. Decoded request structs validated before DB writes. ✓
5. Typed DTOs returned (sqlc structs mapped to response types). ✓
6. golang-migrate migrations committed + run pre-deploy; pgx transactions;
   indexes. ✓
7. Webhooks verify signatures against the raw body before mutating. ✓
8. Secrets stay in server env only. ✓
9. Single-binary Dockerfile + `/health` + env list + documented migrate command
   on Coolify. ✓
10. HTTPS, scoped CORS, rate limits, no secret logging. ✓

## Official References

- chi: https://github.com/go-chi/chi
- pgx: https://github.com/jackc/pgx
- sqlc: https://sqlc.dev/
- golang-migrate: https://github.com/golang-migrate/migrate
- keyfunc (JWKS): https://github.com/MicahParks/keyfunc
