# Security: Web / Next.js App Router

Part of `../security.md` (tags, threat model, and review method live there).

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
  poisoning; see `edge-proxy.md`). Verify: `grep -rniE "headers\(\).*(host|x-forwarded-host)"` → none feed link construction.
- **[HARDEN] Open redirects: same-origin relative only** — honor `next`/`returnTo`
  only when it matches `^/(?!/)` (single leading slash, no scheme). Verify: a protocol-relative `returnTo` → rejected.
- **[HARDEN] Guard DOM/markup sinks** — sanitize before `dangerouslySetInnerHTML`;
  render markdown through an allowlist sanitizer; anchor `postMessage` `event.origin`
  to an allowlist. Verify: `grep -rn "dangerouslySetInnerHTML" .` → each has a sanitizer upstream.
- **[HARDEN] Lock server-side fetch + image optimization (SSRF)** —
  `images.remotePatterns` lists exact hosts, never `**`; no user-controlled host in
  server `fetch`. See `ssrf.md`. Verify: grep `remotePatterns` for `**`.
- **[HARDEN] Complete edge cache key; no CRLF** — keep unkeyed inputs
  (`x-forwarded-host`, cookies) out of cached HTML; route per-user pages so a
  `.json`/`.css` suffix or `//` path can't cache them as static; emit redirects/
  cookies via `NextResponse`/`cookies().set()` (they encode CR/LF). Verify: request an authed page with a static-looking suffix → `cf-cache-status` never `HIT` across users.
