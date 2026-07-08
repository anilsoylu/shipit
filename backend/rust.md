# Backend Playbook: Rust

Use Rust when: you want maximum performance and compile-time safety for the API.

## Project layout

```txt
src/
  main.rs
  routes/       # health, me, webhooks
  auth.rs       # JWKS verification
  db.rs         # sqlx queries
  redis.rs
migrations/     # sqlx migrate .sql files
Cargo.toml
Dockerfile
```

## Libraries

- HTTP framework: axum (`tokio-rs/axum`).
- Postgres: sqlx (async, compile-time-checked queries).
- Migration tool: sqlx-cli (`cargo install sqlx-cli`).
- Validation: `validator` crate (or manual) on deserialized bodies.
- Redis client: `redis` crate (async).

## Auth verification

- Verify the token per `../auth.md`'s Auth Swap Contract via JWKS.
- Use the `jsonwebtoken` crate: fetch the provider JWKS, select the key by `kid`,
  build a `DecodingKey::from_jwk`, and `decode` with a `Validation` that checks
  `iss`/`aud`/`exp`. JWKS fetch + cache is done in an axum middleware/extractor.

## Database & migrations

- sqlx over Postgres; queries checked at compile time.
- Migrations via sqlx-cli: `sqlx migrate add <name>` then `sqlx migrate run`.
- Run before production deploy. Use sqlx transactions for multi-table writes.

## Health, webhooks, security

- `GET /health` route with no auth.
- Webhook routes verify provider signatures against the raw body before mutating.
- axum CORS (`tower-http`) scoped to known clients; a rate-limit layer before
  expensive endpoints.

## Coolify deploy

- Multi-stage Dockerfile: build the release binary, copy into a minimal base.
  Expose `/health`.
- Run `sqlx migrate run` as a release step.

## Env vars

- Public (bundle-safe): none server-side.
- Secret (server only): `DATABASE_URL`, `REDIS_URL`, auth provider secrets, plus
  payment/AI keys in scope.

## Contract compliance

1. `GET /health` responds without auth. ✓
2. Protected routes verify the JWT via `jsonwebtoken` + JWKS extractor. ✓
3. No trusted values from client input; identity from the verified token. ✓
4. Deserialized bodies validated before DB writes. ✓
5. Typed DTOs returned, not raw rows, for sensitive fields. ✓
6. sqlx migrations committed + run pre-deploy; transactions; indexes. ✓
7. Webhooks verify signatures against the raw body before mutating. ✓
8. Secrets stay in server env only. ✓
9. Multi-stage Dockerfile + `/health` + env list + documented migrate command
   on Coolify. ✓
10. HTTPS, scoped CORS, rate limits, no secret logging. ✓

## Official References

- axum: https://github.com/tokio-rs/axum
- sqlx: https://github.com/launchbadge/sqlx
- jsonwebtoken: https://github.com/Keats/jsonwebtoken
