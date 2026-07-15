# Security: Per-framework enforcement (reference)

Part of `../security.md` (tags, threat model, and review method live there). This
is a reference matrix, not a hunt class — it maps the security-critical primitives
onto each backend.

The four security-critical primitives — token verify, tenant-scoped query,
Redis-backed rate limit, raw-body webhook verify — must hold identically in every
backend. Invariants across all five: pin the JWKS algorithm (reject `alg:none` and
RS256→HS256 downgrade), scope every query by the token-derived owner with bound
params, keep rate counters in **Redis** (Coolify runs multiple replicas behind
Traefik — in-process counters silently don't limit), and verify webhooks over the
exact raw bytes.

## Fastify (TS, default)

- **Token verify** — a `fastify-plugin`-wrapped `auth.ts` reads the bearer and
  decorates `req.userId`; Clerk uses its backend SDK, JWKS providers use **jose**
  (`createRemoteJWKSet` + `jwtVerify`, `algorithms: ["RS256"]`).
- **Tenant scope** — `.where(and(eq(t.id, id), eq(t.ownerId, req.userId)))`; no match → 404.
- **Rate limit** — `@fastify/rate-limit` with `{ redis }` and a user-id/pinned-IP `keyGenerator`. **[HARDEN]** default store is in-process; without `redis` each replica counts alone.
- **[BLOCKER] Webhook raw body** — Fastify JSON-parses by default; capture with `fastify-raw-body` scoped to the webhook route and verify `req.rawBody`, not `JSON.stringify(req.body)`.

## Hono (TS, portable)

- **Token verify** — native `hono/jwk` middleware (matches `kid`, rejects HS\*, validates `exp`/`nbf`). **[HARDEN]** it does *not* check `iss`/`aud` — enforce them, or verify with **jose** and `c.set("userId", payload.sub)`.
- **Tenant scope** — `.where(and(eq(t.id, id), eq(t.ownerId, c.get("userId"))))`; bound params only.
- **Rate limit** — `hono-rate-limiter` with `@hono-rate-limiter/redis` (or `INCR`+`EXPIRE`); the default in-memory store is per-instance.
- **[BLOCKER] Webhook raw body** — `const raw = await c.req.text()` then verify that exact string; on edge/Workers use `constructEventAsync` (WebCrypto). Never `c.req.json()` first.

## Go (chi + pgx/sqlc)

- **[BLOCKER] Token verify** — build a `jwt.Keyfunc` with `MicahParks/keyfunc/v3` (or `lestrrat-go/jwx/v4`) and parse with **`jwt.WithValidMethods([]string{"RS256"})`**; without it golang-jwt trusts the token's own `alg` (HS256 confusion). Put the user id in `context.WithValue` under a typed key.
- **Tenant scope** — the predicate lives in the sqlc query (`WHERE id = $1 AND owner_id = $2`); never `fmt.Sprintf` into SQL.
- **Rate limit** — chi `httprate` backed by `httprate-redis` (or `go-redis/redis_rate`); in-memory `httprate` is per-instance.
- **[BLOCKER] Webhook raw body** — `io.ReadAll(r.Body)` (inside `http.MaxBytesReader`) *before* any decode, verify with stripe-go `webhook.ConstructEvent`, then unmarshal the same bytes; `json.NewDecoder(r.Body)` consumes the reader.

## Rust (axum + sqlx)

- **Token verify** — a `FromRequestParts` extractor selects the JWKS key by `decode_header(token)?.kid`, builds `DecodingKey::from_jwk`, and decodes with `Validation::new(Algorithm::RS256)` + `set_issuer`/`set_audience` (crate: `jsonwebtoken`).
- **Tenant scope** — `sqlx::query_as!(T, "... WHERE id=$1 AND owner_id=$2", id, user_id)` — compile-time-checked, bound params.
- **Rate limit** — `tower_governor` layer; **[HARDEN]** its state is per-replica, so back a shared limit with Redis or enforce at Traefik.
- **[BLOCKER] Webhook raw body** — take the `axum::body::Bytes` extractor as the **last** handler arg (it consumes the body); `Json<T>` first loses the raw bytes.

## Python (FastAPI + SQLAlchemy)

- **[BLOCKER] Token verify** — a dependency uses **PyJWT** `PyJWKClient`: `get_signing_key_from_jwt` then `jwt.decode(token, key.key, algorithms=["RS256"], audience=aud, issuer=iss)`. Never `options={"verify_signature": False}`. (python-jose is unmaintained.)
- **Tenant scope** — `select(T).where(T.id == id, T.owner_id == user_id)`; never an f-string or `text()` interpolation.
- **Rate limit** — `slowapi` with `storage_uri="redis://..."`; the in-memory default is per-worker and gunicorn runs several.
- **[BLOCKER] Webhook raw body** — `payload = await request.body()` then `stripe.Webhook.construct_event(payload, sig, secret)`; a Pydantic body param makes FastAPI parse + re-serialize, breaking the signature.
