# Security Playbook

Security is not pluggable — it holds for every stack choice. This file does not
repeat the baseline; it adds the layer above it.

**Baseline (assumed, enforced elsewhere — do not restate here):**

- `rules.md` Hard Boundaries (secrets never in client bundles, clients never
  touch infra directly, never trust client-supplied identity/price/role).
- `backend.md` Backend Swap Contract (server-side token verification, request
  validation, typed DTOs, signed webhooks, HTTPS/CORS/rate-limits).
- `auth.md` Auth Swap Contract (secure token storage, per-request verification,
  identity from the verified token only).

**This file adds:** a pre-build threat-model ritual, the trust-boundary map,
stack-specific rules with copy-paste snippets and verify steps, per-framework
enforcement, and the pre-ship security review pass.

Every rule is tagged **[BLOCKER]** (exploitable, data-loss, or privilege
escalation — fix before ship) or **[HARDEN]** (defense-in-depth — do it, but it
alone won't block a launch). Snippets show `// wrong` vs `// right`; each rule
ends with a `Verify:` you can actually run.

## When to read

- Before building any feature that touches auth, money, PII, uploads, or AI.
- At the `ship-checklist.md` **Security** gate before submission.
- When a security review is requested on existing code.

## Threat-model ritual (before you build)

Six steps; output a short `<feature>-threat-model.md`. Attacker-goal buckets
(exfil / privesc / cross-tenant / integrity / DoS), not STRIDE.

1. **Scope** — the feature's entry points (endpoints, uploads, webhooks, job
   triggers, admin/log sinks), data flows, internet exposure.
2. **Boundaries & assets** — write each trust boundary as a concrete edge
   (protocol / auth / encryption / validation / rate-limit); list the assets it
   exposes (PII, tokens/sessions, authz state, secrets, money-state, compute).
3. **Attacker calibration** — realistic capabilities *and* explicit
   non-capabilities, so severity is not inflated.
4. **Abuse paths** — a few high-quality multi-step stories (entry → boundary →
   asset), not one-line generic threats.
5. **Rank & mitigate** — likelihood × impact, *adjusted for existing controls*;
   tie each fix to a concrete location + control type.
6. **Gate** — every entry point and boundary appears in ≥1 threat; separate
   runtime trust from CI/dev trust; write down assumptions.

## Trust boundaries (shipit)

- **client ↔ API** (Expo/Next → API): client fully untrusted. Re-validate every
  input server-side, verify token + authz per request, rate-limit. Never trust
  client-sent price/role/userId/entitlement.
- **browser ↔ web server** (Next.js): the boundary is Server Actions / route
  handlers / RSC, not the shipped bundle. Auth-check inside each; secrets
  server-only.
- **API ↔ DB/Redis** (Drizzle+Postgres, Redis): private network only, never
  public. Parameterized queries; tenant-scope every query; Redis auth; never
  deserialize untrusted cached blobs.
- **API ↔ webhooks** (Stripe, RevenueCat): inbound is untrusted. Verify
  signature/secret, enforce idempotency, distrust amounts/entitlements in the
  body; reconcile against the provider API before granting.
- **API ↔ AI provider** (OpenRouter): outbound egress. Key server-only; treat
  model output as untrusted input; cap tokens/cost + per-user rate-limit; strip
  PII from prompts.

## Access control & data ownership

The highest-value category — most breaches here are authorization gaps, not
missing auth.

- **[BLOCKER] Authority from token + stored row, never the payload** — derive
  identity, role, and tenant from the verified token per request, and on mutation
  from the existing DB row. A `role`/`plan`/`ownerId` field in the body is a
  privesc/revenue hole.
  ```ts
  // wrong: caller asserts its own entitlement
  if (body.plan === "pro" || body.role === "admin") grant()
  // right: from server-verified state, keyed to the verified user
  const sub = await db.query.subs.findFirst({
    where: and(eq(subs.userId, userId), eq(subs.status, "active")) })
  if (sub?.tier === "pro") grant()
  ```
  Verify: grep authz decisions for `body.role|body.plan|body.tier|body.userId`; forge those fields in a payload → access still denied.
- **[BLOCKER] Scope every query by the session owner** — bind each row to the
  session-derived owner/tenant in the same `WHERE`; for a child row bind it to the
  session-scoped parent. Checking membership elsewhere then fetching by id alone
  is cross-tenant IDOR. Default-deny: no match → 404, not a global read.
  ```ts
  // wrong: fetched by id; ownership checked somewhere else (or not at all)
  await db.query.tasks.findFirst({ where: eq(tasks.id, taskId) })
  // right: bind to the session-derived owner/parent in one WHERE
  await db.query.tasks.findFirst({
    where: and(eq(tasks.id, taskId), eq(tasks.orgId, session.orgId)) })
  ```
  Verify: user A (org1) requests org2's id → expect 404; grep handlers for `findFirst`/`.where(eq(` on a path-param id with no owner/parent `eq`.
- **[BLOCKER] Same authz on create AND update** — the classic hole is "CREATE
  validates, PATCH doesn't": a user creates a valid row, then PATCHes it into a
  privileged or corrupt state (self-assigns role, nulls a foreign key). Authorize
  against the *existing* row's owner, not the incoming payload's. Red-team the
  *sequence*, not the endpoint.
  Verify: for each resource, PATCH a row you own to set `ownerId`/`role`/`tenantId` to another value → rejected; treat `ownerId`/`tenantId`/`createdAt`/`status` as immutable post-create.
- **[BLOCKER] No mass-assignment** — never spread the request body into `.set()`/
  `.insert()`. Pick mutable columns explicitly per role; sensitive columns
  (`role`, `balance`, `price`, `tenantId`) are server-set only.
  ```ts
  // wrong: body decides which columns change
  await db.update(users).set(req.body).where(eq(users.id, userId))
  // right: an explicit, role-bounded column allowlist
  await db.update(users).set({ displayName: input.displayName }).where(eq(users.id, userId))
  ```
  Verify: grep for `.set(req.body`/`.set(body`/`.values(req.body`; send an extra `role`/`price` key → ignored, not written.
- **[HARDEN] Field-allowlist ≠ ownership** — *which* fields may change is not *who*
  may change them; always pair a column allowlist with the ownership check. Verify: a valid field edit on a row you don't own → 404.
- **[HARDEN] IDOR on bulk & alternate paths** — enforce the same per-item check on
  bulk/export/import endpoints and any weaker route to the same mutation, not just
  the single-item one. Verify: call the bulk/export endpoint with a mix of your ids and others' → others' rows are absent.
- **[HARDEN] Tenant-scope cache keys** — Redis keys and Next `unstable_cache`
  keys/tags embed owner/tenant, or one tenant's cached response is served to the
  next. Verify: grep `redis.set(|unstable_cache(|revalidateTag(` for keys/tags with no userId/tenant segment.

## Input & injection

- **[BLOCKER] Parameterized queries + allowlisted identifiers** — Drizzle/`sql`
  interpolation is parameterized; `sql.raw()` and string concatenation are the
  escape hatch. Columns, tables, sort, and direction can't be parameterized — map
  user input through a fixed allowlist of real column refs.
  ```ts
  // wrong: user string becomes a column/direction
  const rows = await db.select().from(posts).orderBy(sql.raw(req.query.sort))
  // right: allowlist maps input -> a real column ref
  const cols = { created: posts.createdAt, title: posts.title } as const
  const col = cols[req.query.sort as keyof typeof cols] ?? posts.createdAt
  const rows = await db.select().from(posts).orderBy(desc(col))
  ```
  Verify: grep `sql.raw(`/`$dynamic(`/`orderBy(` fed by `req.`; never `child_process`/`eval`/shell on untrusted input; send an unknown sort → safe fallback, no SQL error leaked.
- **[BLOCKER] SSRF on any server-fetched URL** — user-settable webhook/callback or
  any server-side `fetch` of a supplied URL: restrict to http/https, block
  private/link-local/metadata ranges (`10/8`, `127/8`, `169.254.0.0/16` — Hetzner
  metadata lives there too), cap redirects, and re-validate the host *after* each
  redirect/DNS resolution (rebinding).
  Verify: point a server-fetch feature at `http://169.254.169.254/…` and at an internal IP → both refused, before and after a redirect.
- **[BLOCKER] Never deserialize untrusted input with a code-executing loader** —
  JSON only for external data, then validate; a function/object-constructing
  deserializer on attacker bytes is RCE.
  ```ts
  // wrong
  nodeSerialize.unserialize(cookie)   // node-serialize runs embedded functions -> RCE
  yaml.load(input)                     // js-yaml v3 resolves !!js/function tags -> RCE (v4 load is safe)
  // right
  const obj = JSON.parse(input)        // then Zod-validate
  ```
  Python: never `pickle.loads`/`yaml.load(..., Loader)` on untrusted input — use `json`/`yaml.safe_load`. Redis: values you wrote are trusted, but never `eval`/`unserialize` a cached blob.
  Verify: grep `unserialize(|yaml.load(|pickle.loads(|vm.runInContext(` on external-input paths → only `JSON.parse`/`safe_load` reach untrusted data.
- **[BLOCKER] Disable external entities in XML parsing (XXE)** — any XML from SAML,
  RSS/sitemap, SVG/Office uploads, or XML webhooks: turn off DTD + external
  entities, or the parser reads local files and makes server-side requests.
  ```ts
  // wrong: entity expansion on -> file read / SSRF
  libxml.parseXml(xml, { noent: true })   // noent EXPANDS entities
  // right
  libxml.parseXml(xml, { noent: false, nonet: true })
  // fast-xml-parser: leave processEntities off; never a DTD-resolving parser on untrusted XML
  ```
  Python: use `defusedxml`, not `xml.etree`/`lxml` defaults. Verify: grep `parseXml|DOMParser|etree|lxml|SAXParser` on request/upload data; feed an XML referencing an external entity in staging → not resolved.
- **[BLOCKER] Path traversal & zip-slip** — never build a filesystem path or an
  archive-extraction target from user input; resolve and confirm it stays under
  the base dir.
  ```ts
  // wrong: a "../../etc/passwd" filename escapes the base
  fs.readFile(path.join(UPLOAD_DIR, req.body.filename))
  // right: basename + resolve + containment check
  const p = path.resolve(UPLOAD_DIR, path.basename(req.body.filename))
  if (!p.startsWith(path.resolve(UPLOAD_DIR) + path.sep)) throw forbidden()
  ```
  Zip extraction: validate each entry stays under the target before writing. (R2 keys are owner-scoped in Storage.) Verify: grep `path.join(|readFile(|createReadStream(|extract(` fed by `req.`/upload names; a `../` filename → rejected.
- **[HARDEN] No server-side template injection (SSTI)** — a user string is data,
  never the template; rendering user input as a template is RCE in Handlebars/EJS/
  Nunjucks (React Email/JSX is safe — data, not template).
  ```ts
  // wrong: user input IS the template
  Handlebars.compile(userInput)(data)     // also ejs.render(userInput), nunjucks.renderString(userInput)
  // right: fixed template, user input only as data
  Handlebars.compile(FIXED_TEMPLATE)({ name: userInput })
  ```
  Verify: grep `compile(|renderString(|new Function(` fed by request data → none; templates are static assets.
- **[HARDEN] Zod `.strict()` at every boundary** — request objects reject unknown
  keys (never `.passthrough()`); validate before any DB write; bound string length
  and array size; audit `z.coerce` usage. Verify: grep `z.object(` on request bodies without `.strict()`, and any `.passthrough(`; oversized array/string → 400.
- **[HARDEN] Prototype pollution** — reject `__proto__`/`constructor`/`prototype`
  keys; never deep-merge or `Object.assign` a raw body into config/query objects.
  Verify: POST a body carrying a `__proto__` key → rejected/ignored, no global mutation.
- **[HARDEN] Stored & indirect injection** — catch input safe on write but
  dangerous on later read, and injection via field names/keys/headers, not just
  values. Verify: store a payload with an injection marker, then exercise every read path that renders or queries it.

## Auth & sessions

Server-side token verification is baseline (`auth.md`). These are the depth rules
the contract doesn't spell out. (Per-framework verify middleware:
**Per-framework enforcement** below.)

- **[BLOCKER] Verify, never decode; pin the algorithm** — resolve the key from a
  server-pinned JWKS/secret with an explicit `algorithms` allowlist; enforce
  `exp`/`nbf`/`aud`/`iss`; never resolve the key from a token-controlled
  `kid`/`jku`/`x5u`.
  ```ts
  import { createRemoteJWKSet, jwtVerify } from "jose"
  // wrong
  const claims = jwt.decode(token)            // no signature check at all
  await jwtVerify(token, key)                  // no algorithms -> alg:none / RS256->HS256 confusion
  // right — key source is server-pinned, never token.kid/jku/x5u
  const JWKS = createRemoteJWKSet(new URL(process.env.JWKS_URL!))
  const { payload } = await jwtVerify(token, JWKS, {
    algorithms: ["RS256"], issuer: process.env.AUTH_ISS!, audience: process.env.AUTH_AUD!,
    clockTolerance: 5,
  })
  ```
  Verify: grep the auth path — no `jwt.decode`, every `jwtVerify`/`jwt.verify` pins `algorithms`; an `alg:none` token and an HS256-header-swapped token both 401.
- **[BLOCKER] Cookie flags + server-side invalidation** — `HttpOnly`+`Secure`+
  `SameSite` + `__Host-` prefix; rotate the id on login/privilege-change; revoke
  server-side on logout/password-change (stateless JWT needs a `jti`/version
  denylist checked per request).
  ```ts
  reply.setCookie("__Host-sid", sid, {
    httpOnly: true, secure: true, sameSite: "lax",  // never "none" without cause
    path: "/",                                        // __Host- requires path=/ and NO Domain
    maxAge: 60 * 60 * 8,
  })
  ```
  Verify: inspect `Set-Cookie` for `HttpOnly; Secure; SameSite`; replay a pre-logout token → 401 (denylist hit).
- **[BLOCKER] Rate-limit auth in Redis, pinned to a trusted IP** — fixed window
  per IP+route in Redis. In-memory is per-instance, so a multi-instance Coolify
  deploy silently loses brute-force protection; behind Traefik pin the client IP
  (trusted-proxy hop count), never raw spoofable `X-Forwarded-For`.
  ```ts
  const redis = new Redis(process.env.REDIS_URL!)
  async function allow(ip: string, route: string, limit = 5, windowS = 60) {
    const key = `rl:${route}:${ip}`
    const n = await redis.incr(key)
    if (n === 1) await redis.expire(key, windowS)
    return n <= limit                                 // false -> 429
  }
  ```
  Verify: `POST /sign-in` 6x from one IP → 429; the counter survives an instance restart (`redis-cli GET rl:sign-in:<ip>`).
- **[HARDEN] Rotate refresh tokens + detect reuse** — rotate on every refresh,
  track the token family, revoke the whole family when a spent token is replayed.
  Verify: replay a used refresh token → family-wide invalidation, not a silent new session.
- **[HARDEN] freshAge re-auth for sensitive mutations** — require recent re-auth
  (~1h) for email/password/2FA/payout changes; long sessions widen the
  stolen-token window. Verify: mutate email with a stale session → step-up challenge.
- **[HARDEN] trustedOrigins list the native scheme** — enumerate web domain(s) +
  preview wildcards + the Expo/native scheme (`myapp://`, `exp://`); missing breaks
  mobile auth, a bare `*` widens CSRF trust. Verify: grep the config — native scheme present, no bare `*`.
- **[HARDEN] Never disable OAuth state globally** — scope any state-cookie skip to
  the native client (or a DB state strategy); a global off-switch drops CSRF
  protection on web too. Verify: grep `state:\s*false`/`disableCSRF` → gated by platform.
- **[HARDEN] High-entropy secret, no env-fallback footgun** — 32+ byte random
  signing secret, env-only; a fallback var name or inline `|| "default"` can
  silently override it. Verify: assert secret length ≥ 32 at boot; grep for `|| "` defaults on the secret.
- **[HARDEN] Encrypt stored provider tokens** — AES-256-GCM at rest so a Postgres
  dump doesn't yield usable third-party OAuth/refresh tokens. Verify: read the token column → ciphertext, not a `ya29.`/`gho_` prefix.
- **[HARDEN] Account-enumeration parity** — generic "Invalid credentials",
  constant-time (dummy hash on unknown user), always background the email on
  signup/reset. Verify: unknown vs known email on reset → identical status, body, and timing.

## Cryptography & transport

- **[BLOCKER] CSPRNG for every security token** — reset tokens, OTPs, session ids,
  API keys, and nonces come from a CSPRNG; `Math.random()` is predictable, so a
  forgeable reset token is account takeover.
  ```ts
  // wrong
  const token = Math.random().toString(36).slice(2)
  // right
  import { randomBytes, randomUUID } from "crypto"
  const token = randomBytes(32).toString("base64url")   // randomUUID() for ids
  ```
  Go `crypto/rand` (not `math/rand`); Python `secrets` (not `random`); Rust `OsRng`. Verify: grep `Math.random(|math/rand|random.random(|random.randint(` in token/id/secret/otp paths → none.
- **[BLOCKER] Password hashing is a slow salted KDF** — self-hosted auth
  (custom-JWT, better-auth) hashes with argon2id (or bcrypt/scrypt), per-password
  salt; never MD5/SHA-1/SHA-256/plaintext.
  ```ts
  // wrong
  const h = crypto.createHash("sha256").update(password).digest("hex")
  // right
  import { hash, verify } from "@node-rs/argon2"
  const stored = await hash(password)                   // argon2id, salted
  ```
  Verify: grep `createHash(.*(sha|md5)` in the password path → none; stored hashes start with `$argon2id$`/`$2b$`.
- **[BLOCKER] Never disable TLS verification** — server-to-server calls (webhook
  verify, provider APIs, DB) keep certificate validation on; `rejectUnauthorized:
  false` / `NODE_TLS_REJECT_UNAUTHORIZED=0` / `verify=False` / `InsecureSkipVerify`
  is a MITM hole that voids every signature and secret in transit.
  ```ts
  // wrong
  new https.Agent({ rejectUnauthorized: false })
  // right — verification on; pin a private CA explicitly only if you run one
  new https.Agent({ ca: fs.readFileSync("private-ca.pem") })
  ```
  Verify: grep `rejectUnauthorized:\s*false|NODE_TLS_REJECT_UNAUTHORIZED|verify=False|InsecureSkipVerify` → none in app/prod code.
- **[HARDEN] No weak or homemade crypto** — AES-GCM / ChaCha20-Poly1305 for
  symmetric (never ECB, never a static/reused IV/nonce); no hand-rolled crypto;
  constant-time compare (`timingSafeEqual`) for secrets and MACs. Verify: grep `createCipheriv(.*ecb` and secret comparisons using `===`/`==` instead of `timingSafeEqual`.

## Web / Next.js App Router

- **[BLOCKER] Server Actions are unauthenticated POST endpoints** — every
  `"use server"` action re-runs authN + authZ + input validation + rate-limit;
  reachability is never authorization, and Next middleware is not an authz boundary
  (bypassable, e.g. CVE-2025-29927).
  ```ts
  "use server"
  export async function deletePost(form: FormData) {
    const { userId } = await auth()                                  // authN
    if (!userId) throw new Error("unauthorized")
    const { postId } = Input.parse({ postId: form.get("postId") })   // validate
    const row = await db.query.posts.findFirst({ where: eq(posts.id, postId) })
    if (row?.ownerId !== userId) throw new Error("forbidden")        // authZ on the row
    await db.delete(posts).where(eq(posts.id, postId))
  }
  ```
  Verify: every `"use server"` file resolves `auth()` before any `db.*`; `serverActions.allowedOrigins` is unset or an exact `APP_ORIGIN`, never a wildcard/preview suffix; Next ≥ 14.2.25 / 15.2.3.
- **[BLOCKER] CORS is an exact-match allowlist, never reflect Origin** —
  credentialed cross-origin reads require an allowlisted Origin echoed verbatim +
  `Vary: Origin`. The bug is Origin *reflection* or weak suffix-match, not `*`
  (the browser already rejects `*` with credentials).
  ```ts
  const ALLOWED = new Set([process.env.APP_ORIGIN!])
  const origin = req.headers.get("origin")
  // wrong: res.headers.set("Access-Control-Allow-Origin", origin ?? "*")   // reflects any caller
  // wrong: origin?.endsWith("myapp.com")   // also matches evil-myapp.com and myapp.com.evil.dev
  if (origin && ALLOWED.has(origin)) {
    res.headers.set("Access-Control-Allow-Origin", origin)
    res.headers.set("Access-Control-Allow-Credentials", "true")
    res.headers.append("Vary", "Origin")
  }
  ```
  Verify: `curl -H "Origin: https://evil.com" https://APP/api/x -i | grep -i access-control-allow-origin` returns nothing.
- **[BLOCKER] Ship a nonce CSP + framing/transport headers** — per-request nonce
  with `strict-dynamic`; `frame-ancestors 'none'` + HSTS kill clickjacking/
  downgrade; `base-uri 'none'` blocks `<base>` injection. Set from `middleware.ts`
  and mirror HSTS at Cloudflare.
  ```ts
  // middleware.ts
  const nonce = Buffer.from(crypto.randomUUID()).toString("base64")
  const csp = [
    `default-src 'self'`, `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
    `style-src 'self' 'nonce-${nonce}'`, `connect-src 'self'`,   // + Sentry/PostHog ingest
    `img-src 'self' data:`, `object-src 'none'`, `base-uri 'none'`,
    `form-action 'self'`, `frame-ancestors 'none'`,
  ].join("; ")
  const res = NextResponse.next({ request: { headers: /* x-nonce */ } })
  res.headers.set("Content-Security-Policy", csp)
  res.headers.set("Strict-Transport-Security", "max-age=63072000; includeSubDomains; preload")
  res.headers.set("X-Frame-Options", "DENY")
  res.headers.set("X-Content-Type-Options", "nosniff")
  res.headers.set("Referrer-Policy", "strict-origin-when-cross-origin")
  ```
  Verify: `curl -I https://APP | grep -iE "content-security-policy|strict-transport|x-frame-options"`; no `'unsafe-inline'` in prod `script-src`.
- **[BLOCKER] Per-user responses are uncacheable and shared-cache-safe** — authed
  reads carry `no-store, private` + `Vary: Cookie`; never `force-static`/`use cache`
  over user data; Cloudflare must not "Cache Everything" on authed paths.
  ```ts
  export const dynamic = "force-dynamic"   // never force-static on an authed route
  return Response.json(await getMe(), {
    headers: { "Cache-Control": "private, no-store", "Vary": "Cookie" } })
  ```
  Verify: `curl -I https://APP/api/me | grep -i cache-control` shows `no-store, private`; user A's vs B's cookie return different bodies and `cf-cache-status` is never `HIT`.
- **[HARDEN] Server/client boundary is a trust boundary** — DAL modules start with
  `import "server-only"`; no DB client, secret, or non-`NEXT_PUBLIC` env read is
  reachable from a `"use client"` module (every `NEXT_PUBLIC_*` ships in the
  bundle). Verify: `grep -rniE "NEXT_PUBLIC.*(SECRET|KEY|TOKEN|PASSWORD)" .` is empty.
- **[HARDEN] `.bind()` args are signed, not encrypted** — never bind secrets into
  an action closure; scrub action args from Sentry/PostHog. Verify: grep `.bind(` for secret-looking values.
- **[HARDEN] Build absolute URLs from a trusted constant** — emails, redirects, and
  OAuth callbacks use `env.APP_ORIGIN`, never `Host`/`x-forwarded-host` (reset-link
  poisoning). Verify: `grep -rniE "headers\(\).*(host|x-forwarded-host)"` → none feed link construction.
- **[HARDEN] Open redirects: same-origin relative only** — honor `next`/`returnTo`
  only when it matches `^/(?!/)` (single leading slash, no scheme). Verify: a protocol-relative `returnTo` → rejected.
- **[HARDEN] Guard DOM/markup sinks** — sanitize before `dangerouslySetInnerHTML`;
  render markdown through an allowlist sanitizer; anchor `postMessage` `event.origin`
  to an allowlist. Verify: `grep -rn "dangerouslySetInnerHTML" .` → each has a sanitizer upstream.
- **[HARDEN] Lock server-side fetch + image optimization (SSRF)** —
  `images.remotePatterns` lists exact hosts, never `**`; no user-controlled host in
  server `fetch`. Verify: grep `remotePatterns` for `**`.
- **[HARDEN] Complete edge cache key; no CRLF** — keep unkeyed inputs
  (`x-forwarded-host`, cookies) out of cached HTML; route per-user pages so a
  `.json`/`.css` suffix or `//` path can't cache them as static; emit redirects/
  cookies via `NextResponse`/`cookies().set()` (they encode CR/LF). Verify: request an authed page with a static-looking suffix → `cf-cache-status` never `HIT` across users.

## Per-framework enforcement

The four security-critical primitives — token verify, tenant-scoped query,
Redis-backed rate limit, raw-body webhook verify — must hold identically in every
backend. Invariants across all five: pin the JWKS algorithm (reject `alg:none` and
RS256→HS256 downgrade), scope every query by the token-derived owner with bound
params, keep rate counters in **Redis** (Coolify runs multiple replicas behind
Traefik — in-process counters silently don't limit), and verify webhooks over the
exact raw bytes.

### Fastify (TS, default)

- **Token verify** — a `fastify-plugin`-wrapped `auth.ts` reads the bearer and
  decorates `req.userId`; Clerk uses its backend SDK, JWKS providers use **jose**
  (`createRemoteJWKSet` + `jwtVerify`, `algorithms: ["RS256"]`).
- **Tenant scope** — `.where(and(eq(t.id, id), eq(t.ownerId, req.userId)))`; no match → 404.
- **Rate limit** — `@fastify/rate-limit` with `{ redis }` and a user-id/pinned-IP `keyGenerator`. **[HARDEN]** default store is in-process; without `redis` each replica counts alone.
- **[BLOCKER] Webhook raw body** — Fastify JSON-parses by default; capture with `fastify-raw-body` scoped to the webhook route and verify `req.rawBody`, not `JSON.stringify(req.body)`.

### Hono (TS, portable)

- **Token verify** — native `hono/jwk` middleware (matches `kid`, rejects HS\*, validates `exp`/`nbf`). **[HARDEN]** it does *not* check `iss`/`aud` — enforce them, or verify with **jose** and `c.set("userId", payload.sub)`.
- **Tenant scope** — `.where(and(eq(t.id, id), eq(t.ownerId, c.get("userId"))))`; bound params only.
- **Rate limit** — `hono-rate-limiter` with `@hono-rate-limiter/redis` (or `INCR`+`EXPIRE`); the default in-memory store is per-instance.
- **[BLOCKER] Webhook raw body** — `const raw = await c.req.text()` then verify that exact string; on edge/Workers use `constructEventAsync` (WebCrypto). Never `c.req.json()` first.

### Go (chi + pgx/sqlc)

- **[BLOCKER] Token verify** — build a `jwt.Keyfunc` with `MicahParks/keyfunc/v3` (or `lestrrat-go/jwx/v4`) and parse with **`jwt.WithValidMethods([]string{"RS256"})`**; without it golang-jwt trusts the token's own `alg` (HS256 confusion). Put the user id in `context.WithValue` under a typed key.
- **Tenant scope** — the predicate lives in the sqlc query (`WHERE id = $1 AND owner_id = $2`); never `fmt.Sprintf` into SQL.
- **Rate limit** — chi `httprate` backed by `httprate-redis` (or `go-redis/redis_rate`); in-memory `httprate` is per-instance.
- **[BLOCKER] Webhook raw body** — `io.ReadAll(r.Body)` (inside `http.MaxBytesReader`) *before* any decode, verify with stripe-go `webhook.ConstructEvent`, then unmarshal the same bytes; `json.NewDecoder(r.Body)` consumes the reader.

### Rust (axum + sqlx)

- **Token verify** — a `FromRequestParts` extractor selects the JWKS key by `decode_header(token)?.kid`, builds `DecodingKey::from_jwk`, and decodes with `Validation::new(Algorithm::RS256)` + `set_issuer`/`set_audience` (crate: `jsonwebtoken`).
- **Tenant scope** — `sqlx::query_as!(T, "... WHERE id=$1 AND owner_id=$2", id, user_id)` — compile-time-checked, bound params.
- **Rate limit** — `tower_governor` layer; **[HARDEN]** its state is per-replica, so back a shared limit with Redis or enforce at Traefik.
- **[BLOCKER] Webhook raw body** — take the `axum::body::Bytes` extractor as the **last** handler arg (it consumes the body); `Json<T>` first loses the raw bytes.

### Python (FastAPI + SQLAlchemy)

- **[BLOCKER] Token verify** — a dependency uses **PyJWT** `PyJWKClient`: `get_signing_key_from_jwt` then `jwt.decode(token, key.key, algorithms=["RS256"], audience=aud, issuer=iss)`. Never `options={"verify_signature": False}`. (python-jose is unmaintained.)
- **Tenant scope** — `select(T).where(T.id == id, T.owner_id == user_id)`; never an f-string or `text()` interpolation.
- **Rate limit** — `slowapi` with `storage_uri="redis://..."`; the in-memory default is per-worker and gunicorn runs several.
- **[BLOCKER] Webhook raw body** — `payload = await request.body()` then `stripe.Webhook.construct_event(payload, sig, secret)`; a Pydantic body param makes FastAPI parse + re-serialize, breaking the signature.

## Mobile / Expo

- **[BLOCKER] No real secret in the JS bundle** — `EXPO_PUBLIC_*` and any imported
  literal are inlined into the shipped bundle at build; only publishable/anon keys
  belong client-side, every real key stays behind the API. Verify: `grep -r EXPO_PUBLIC apps/mobile | grep -iE 'secret|sk_live|service_role|private'` → empty; unzip the IPA/APK and grep the JS.
- **[HARDEN] Lock SecureStore accessibility** — tokens in `expo-secure-store`
  (baseline) with `keychainAccessible: WHEN_UNLOCKED_THIS_DEVICE_ONLY` so they
  never sync to iCloud/backups. Verify: `grep -rn SecureStore apps/mobile` shows the option; no token in AsyncStorage/MMKV.
- **[HARDEN] Strip logs and PII from release builds** — `console.*` persists to
  device logs and can carry tokens/PII; drop it in production
  (`transform-remove-console`), never log request/response bodies. Verify: `adb logcat` during sign-in shows no token/PII.
- **[HARDEN] Verified links for auth, not custom schemes** — custom schemes are
  claimable by other apps; use Universal Links / App Links (AASA, `assetlinks.json`)
  for OAuth/magic-link callbacks, validate every param, and treat a deep link as
  never proof of auth. Verify: a crafted `myapp://…` link → no privileged nav without a verified session.
- **[HARDEN] Block sensitive screens from snapshots** — Android `FLAG_SECURE` /
  `expo-screen-capture` + a privacy overlay on background, so the OS app-switcher
  snapshot and screenshots capture nothing on token/PII screens. Verify: background on a sensitive screen → recents thumbnail is blank/blurred.
- **[HARDEN] Don't trust the client for integrity** — root/jailbreak flags are
  bypassable and must never gate server-side authorization; for high-value flows
  use server-verified Play Integrity / App Attest, and sign OTA updates (EAS Update
  code signing). Verify: server rejects the action when attestation is absent; a tampered OTA update is rejected.

## Webhooks & payments

Signed-webhook handling is baseline (`backend.md`); the traps:

- **[BLOCKER] Verify against the raw bytes the framework never parsed** — the
  signature is an HMAC of the exact payload; a parser that decodes first (then you
  re-`stringify`) changes bytes and lets forged events through. (Per-framework
  raw-body access: **Per-framework enforcement** above.)
  ```ts
  const event = stripe.webhooks.constructEvent(
    req.rawBody, req.headers["stripe-signature"]!, process.env.STRIPE_WEBHOOK_SECRET!)
  ```
  Verify: `stripe trigger` a real event → verifies; swap `req.rawBody` → `JSON.stringify(req.body)` → `constructEvent` throws.
- **[BLOCKER] RevenueCat auth is a static header secret, not an HMAC** — there is no
  signature; compare the `Authorization` header in constant time, or anyone POSTs
  fake purchase events and self-grants entitlements.
  ```ts
  const got = Buffer.from(req.headers.authorization ?? "")
  const want = Buffer.from(process.env.RC_WEBHOOK_SECRET!)
  if (got.length !== want.length || !timingSafeEqual(got, want)) return reply.code(401).send()
  ```
  Verify: POST with a wrong/missing `Authorization` → 401, no entitlement row written.
- **[BLOCKER] Revoke on refund, dispute, and cancel** — grant-only logic lets a
  buyer pay, refund/chargeback, and keep access. Handle `charge.refunded` /
  `charge.dispute.created` / `customer.subscription.deleted` (RevenueCat:
  `CANCELLATION`/`EXPIRATION`/`REFUND`). Verify: refund a test charge → entitlement flips to revoked before the next authorized request.
- **[HARDEN] Bind the payment to your user server-side** — resolve identity via
  `client_reference_id`/`metadata.userId` or your Stripe-customer↔user table; never
  grant by an email read from the body. Verify: a webhook with a spoofed email grants nothing.
- **[HARDEN] Dedup in the same transaction as the side effect** — insert `event.id`
  under a unique constraint and grant in one tx; check-then-write races on
  concurrent retries and double-grants. Verify: replay the same webhook id twice concurrently → exactly one grant.

## Storage / Cloudflare R2

- **[BLOCKER] Sign a server-derived key, never a client-supplied path** — the object
  key comes from an ownership lookup; if the client names the key it swaps in
  someone else's object (IDOR).
  ```ts
  const file = await db.query.files.findFirst({ where: eq(files.id, fileId) })
  if (!file || file.ownerId !== session.userId) throw forbidden()          // authz BEFORE signing
  const url = await getSignedUrl(r2,
    new GetObjectCommand({ Bucket: "private", Key: file.key }),            // key from DB, not the request
    { expiresIn: 120 })
  ```
  Verify: mint a URL for your file, then request another user's id → 403; a signed URL for object A cannot GET object B.
- **[HARDEN] Bucket is private** — no `r2.dev` and no public custom domain; all reads
  go through signed URLs or an authorizing Worker. Verify: curl the object's public/`r2.dev` URL → 401/403.
- **[HARDEN] Constrain uploads at presign time** — `createPresignedPost` with a
  `content-length-range` + fixed `Content-Type` + per-user key prefix
  (`u/${userId}/…`). Verify: a presigned upload of 2x the size limit or wrong content-type → rejected.
- **[HARDEN] Serve user files from a cookieless domain, or force download** — an
  uploaded HTML/SVG served from the app origin is stored XSS; set
  `Content-Disposition: attachment` and a non-renderable `Content-Type`. Verify: upload an `.html`, open its URL → it downloads, does not execute on your origin.

## AI features / OpenRouter proxy

- **[BLOCKER] Tool-calls authorize the requesting user against the target resource**
  — the model supplies the args (untrusted); authorization uses the server session
  identity on the specific resource, never a service key, or the agent is a confused
  deputy acting above the caller's level.
  ```ts
  // ctx.userId comes from the authenticated session, NOT from the model
  async function deleteDoc(args: { docId: string }, ctx: { userId: string }) {
    const doc = await db.query.docs.findFirst({ where: eq(docs.id, args.docId) })
    if (!doc || doc.ownerId !== ctx.userId) throw new Error("forbidden")   // per-resource authz
    await db.delete(docs).where(eq(docs.id, args.docId))
  }
  ```
  Verify: as user B, prompt the agent to act on user A's `docId` → tool throws forbidden, no mutation.
- **[BLOCKER] Model output is untrusted input to its sink** — never `eval`, run as
  SQL, or render model output as HTML; prompt injection is a real finding only when
  it crosses a boundary (a victim context, a capability the requester lacks, a
  server-side sink) — a guardrail prompt is *not* a control.
  ```ts
  // wrong: model output reaches a dangerous sink
  eval(out); db.execute(sql.raw(out)); <div dangerouslySetInnerHTML={{ __html: out }} />
  // right: treat as untrusted text
  render(escapeText(out))
  ```
  Verify: trace model output into `dangerouslySetInnerHTML|eval|sql.raw|child_process` → none; validate tool args with a Zod schema before use.
- **[HARDEN] Key proxy-only; block SSRF in model/RAG fetch** — the OpenRouter key
  and all provider creds live in the API; clients only call your endpoint. Any
  model- or user-driven fetch (RAG, image, URL tool) denies non-http(s) and
  private/link-local/metadata ranges. Verify: grep the client bundle for the OpenRouter key → none; a fetch tool for `http://169.254.169.254/…` → refused.
- **[HARDEN] Cap output, budget, and indirect injection** — set `max_tokens`, a
  per-user spend cap, and a global kill-switch; treat RAG/upload/retrieved content
  as a prompt-injection source; strip PII from prompts. Verify: loop past the cap → 429 before spend continues.
- **[HARDEN] Sanitize what the proxy returns** — stream only the completion; never
  forward upstream error bodies/headers or echo the system prompt (leaks routing,
  cost, key metadata). Verify: force an upstream 401 → client sees a generic error, no provider detail.

## Secrets & config

- **[BLOCKER] A committed secret is burned — rotate, don't just delete** — removing
  the line leaves it in git history, forks, and backups; rotate at the provider
  first, then purge. Report as `path:line` + credential type, never the value.
  ```bash
  gitleaks detect --no-git --redact -v          # working tree; drop --no-git for full history
  # fallback (detection patterns, not secrets):
  grep -rnE 'sk_live_|rk_live_|AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}|-----BEGIN [A-Z ]+PRIVATE KEY-----' .
  ```
  Verify: `gitleaks detect --no-git --redact` exits 0; after a burn, confirm the old key is revoked at the provider (a rotated-but-not-revoked key is still live).
- **[BLOCKER] Diagnostics endpoints are not public** — prod exposes nothing at
  `/debug`, `/admin`, `/metrics`, `/env`, `/actuator`, Node `--inspect`, or Go
  `pprof`/`expvar`; bind to localhost/private network or require auth at Traefik.
  ```bash
  for p in /metrics /debug/pprof/ /env /actuator/health; do
    curl -s -o /dev/null -w "$p %{http_code}\n" https://api.example.com$p; done   # expect 404/401
  ```
  Verify: the loop above returns 404/401, never 200.
- **[HARDEN] `.gitignore` covers secret files and none are tracked** — ignore
  `.env*`, `*.pem`, `*.key`, `credentials.json`; a file added before the ignore rule
  stays tracked forever. Verify: `git ls-files | grep -E '\.env($|\.)|\.pem$|\.key$|credentials'` → empty.
- **[HARDEN] Secrets from the environment, not the image or repo** — inject via
  Coolify env/secret store at runtime; never bake a secret into a Docker layer or
  commit it to `eas.json`/CI YAML. Verify: `docker history --no-trunc <image>` and grep `eas.json .github/` show no secret literals.
- **[HARDEN] Error hygiene — generic to the client, detailed to the log** — no stack
  traces, SQL, driver errors, file paths, or internal hostnames in client responses;
  log detail server-side keyed by a request id; no publicly served source maps.
  Verify: force a DB error via the API → generic body + id, full detail only in the server log/Sentry.

## Logging & data minimization

- **[BLOCKER] Scrub before events leave the process** — Sentry/PostHog capture
  headers, bodies, breadcrumbs, and local variables by default; strip
  auth/cookies/tokens/PII in `beforeSend`.
  ```ts
  Sentry.init({ dsn: process.env.SENTRY_DSN, sendDefaultPii: false,
    beforeSend(event) {
      const DROP = /authorization|cookie|x-api-key|token|password|secret/i
      const scrub = (o?: Record<string, unknown>) =>
        o && Object.keys(o).forEach((k) => DROP.test(k) && (o[k] = "[redacted]"))
      scrub(event.request?.headers); scrub(event.extra)
      if (event.request) { delete event.request.data; delete event.request.query_string }
      if (event.user) event.user = { id: event.user.id }        // drop email/ip/username
      return event
    } })
  ```
  Verify: trigger a captured error on an authed request → the Sentry event has no `Authorization`/cookie header, no request body, `user` is id-only.
- **[HARDEN] Redact at the logger, not the call site** — a central redaction list
  (pino `redact`, a scrub middleware) so a new `logger.info(req)` can't leak; never
  `JSON.stringify(req)` or log full bodies. Verify: grep `logger.*req|console.log(.*body` and confirm the redactor covers `authorization`, `cookie`, `password`, `token`.
- **[HARDEN] Minimize what you collect** — no email/full IP/precise location/device
  ids to PostHog; stable pseudonymous id, masked autocapture inputs
  (`data-ph-no-capture`), session-replay masking on. Verify: inspect a PostHog event → no raw email/PII; replay masks input fields.
- **[HARDEN] Route auth events to an audit trail** — session create/revoke, login
  success+failure, password/email change, account link, role change → structured
  audit events (actor, target, source ip, timestamp). Verify: a role change produces a structured audit event; failed-login spikes are queryable/alertable.

## Dependencies

- **[BLOCKER] CI fails on reachable critical/high advisories** — a read-only audit
  gate per ecosystem, blocking on critical/high in runtime or build/distribution
  paths; dev-only/unreachable advisories are triaged, not silently ignored (and
  never auto-fixed in CI).
  ```yaml
  - run: pnpm audit --prod --audit-level=high     # apps/web, apps/mobile
  - run: pip-audit -r apps/api/requirements.txt    # Python API
  - run: cargo audit --deny warnings               # Rust API
  - run: govulncheck ./...                         # Go API (reachability-aware)
  ```
  Verify: add a package with a known high advisory → the CI job fails and blocks the merge.
- **[HARDEN] Lockfile committed, pinned, no drift** — commit the lockfile, install
  with `--frozen-lockfile` in CI, dedupe versions across `apps/*`. Verify: `pnpm install --frozen-lockfile` is clean in CI; `git ls-files '*lock*'` shows it tracked.
- **[HARDEN] SRI or self-host third-party scripts** — a CDN `<script>`/`<link>`
  carries `integrity` + `crossorigin`, or you self-host; a mutable CDN URL is a
  supply-chain injection path. Verify: view-source → each external script tag has an `integrity=` hash.
- **[HARDEN] Pin CI actions and base images by digest** — GitHub Actions by commit
  SHA (not `@v3`) and Docker `FROM` by `@sha256:`, so a moved tag can't swap in
  malicious build steps. Verify: grep workflows for `uses:.*@[0-9a-f]{40}`; `grep FROM Dockerfile` shows `@sha256:` digests.

## Security review pass (before ship)

The `ship-checklist.md` Security gate. Method:

1. **Recon** — map architecture, trust boundaries, and input surfaces (fan out
   parallel read-only readers on a repo of any size).
2. **Hunt** — one pass per attack class (access control, injection, auth, SSRF,
   secrets, config, AI), routed to the sections above.
3. **Validate** — *disprove-first*: try to refute every finding; dedup; drop
   by-design behavior (a proxy convention, an ADR decision). A stale ADR the code
   has drifted from is itself a finding.
4. **Report** — the finding format below, ordered by leverage.

**Severity = likelihood × impact.** **Source-visibility gate:** if confirming a
finding needs infra you can't read (proxy chain, live cache, prod auth), mark it
"requires deployment testing" — do not report it as confirmed.

Finding format (never copy a secret value into a finding):

```markdown
### [CATEGORY] Short imperative title
- Evidence: `path/file.ts:123` — what's there (2–5 strongest locations).
- Impact: concrete production consequence.
- Effort: S / M / L (for the fix, including tests).
- Confidence: HIGH (read the code) / MED (needs verification) / LOW (investigate).
- Fix sketch: 1–3 sentences (secrets: always include rotation).
```
