# Security: Auth & sessions

Part of `../security.md` (tags, threat model, and review method live there).

Server-side token verification is baseline (`auth.md`). These are the depth rules
the contract doesn't spell out. Per-framework verify middleware lives in
`per-framework.md`.

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
- **[BLOCKER] CSRF: cookie-session mutations need a control beyond SameSite** — for
  any cookie-authenticated mutation (better-auth / custom-JWT web sessions),
  `SameSite=lax` is necessary but not sufficient (GET side effects, same-site
  subdomains, older clients). Require an Origin/Referer allowlist check or a
  double-submit token on every non-GET. Bearer-token APIs (`Authorization`) are
  not CSRF-able; Next Server Actions already enforce an origin check (`web-nextjs.md`).
  ```ts
  // wrong: a mutating cookie route relying on SameSite alone
  app.post("/account/email", handler)
  // right: reject cross-origin before mutating
  const origin = req.headers.origin ?? req.headers.referer
  if (!origin || !ALLOWED_ORIGINS.has(new URL(origin).origin)) return reply.code(403).send()
  ```
  Verify: replay a mutating POST with a foreign `Origin` and a valid session cookie → 403; a same-origin request still succeeds.
- **[BLOCKER] Rate-limit auth in Redis, pinned to a trusted IP** — fixed window
  per IP+route in Redis. In-memory is per-instance, so a multi-instance Coolify
  deploy silently loses brute-force protection; behind Traefik pin the client IP
  (trusted-proxy hop count, see `edge-proxy.md`), never raw spoofable
  `X-Forwarded-For`.
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
- **[BLOCKER] OAuth flow integrity: exact `redirect_uri`, PKCE, verified `id_token`**
  — an OAuth/OIDC callback is an account-takeover surface. Match `redirect_uri`
  against an exact allowlist (never substring/prefix, never an open redirect on an
  allowlisted host); require PKCE on public/native clients (Expo — the auth code is
  interceptable without it); verify the `id_token` with the same discipline as the
  bearer rule above (signature, `aud`, `iss`) **plus** the `nonce` you issued. State
  is baseline (the CSRF rule); these are the code-injection and token-forgery half.
  ```ts
  // wrong: prefix match + code accepted without PKCE / nonce
  if (redirectUri.startsWith("https://app.example.com")) ok()   // app.example.com.evil.dev passes
  // right: exact allowlist + PKCE verifier + nonce bound to this session
  if (!REDIRECT_URIS.has(redirectUri)) throw forbidden()
  // exchange sends code_verifier; then verify the id_token like any JWT + check nonce
  const { payload } = await jwtVerify(idToken, JWKS, { algorithms: ["RS256"], audience: CLIENT_ID, issuer: ISS })
  if (payload.nonce !== session.oauthNonce) throw forbidden()
  ```
  Verify: a callback with a look-alike `redirect_uri` → rejected; a public-client flow without PKCE → rejected; an `id_token` minted for another `aud` or carrying a stale `nonce` → rejected.
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
