# Security: Edge & reverse proxy (Traefik / Coolify)

Part of `../security.md` (tags, threat model, and review method live there).

Self-hosted on Coolify/Hetzner, Traefik is the only public hop. The app trusts
Traefik; it must trust nothing a client can forge through it.

- **[BLOCKER] Pin the client IP to the trusted proxy hop, don't trust raw
  `X-Forwarded-For`** — behind Traefik the client fully controls the inbound
  `X-Forwarded-For`; if the app believes it, an attacker spoofs any IP to bypass
  rate-limits and IP allowlists. Configure the framework's trusted-proxy count so
  only Traefik's appended hop is believed.
  ```ts
  // wrong: read the client-controlled header
  const ip = (req.headers["x-forwarded-for"] as string)?.split(",")[0]   // fully spoofable
  // right: framework trusts exactly the known proxy hop, then req.ip is real
  const app = Fastify({ trustProxy: 1 })   // Traefik is hop 1
  const ip = req.ip                          // rate-limit / allowlist key on this
  ```
  Go/Rust/Python equivalents: set the trusted-proxy/real-IP config to the Traefik hop count, never accept the raw header. Verify: send `X-Forwarded-For: 1.2.3.4` directly to the app → `req.ip` is your real address, not `1.2.3.4`; the rate-limit counter keys on the real IP.
- **[BLOCKER] `Host` / `X-Forwarded-Host` are attacker-controlled** — never build
  links (reset/verify/OAuth callback), cache keys, or auth decisions from them; use
  a trusted `APP_ORIGIN` constant. A poisoned `Host` sends password-reset links to
  the attacker's domain. (Link construction: `web-nextjs.md`.)
  ```ts
  // wrong
  const link = `https://${req.headers.host}/reset?token=${t}`
  // right
  const link = `${process.env.APP_ORIGIN}/reset?token=${t}`
  ```
  Verify: request with `Host: evil.com` / `X-Forwarded-Host: evil.com` → the emitted reset link still points at `APP_ORIGIN`.
- **[HARDEN] Admin and infra planes are not public** — the Traefik dashboard, the
  Coolify UI, and Postgres/Redis ports bind to the private network or sit behind
  auth; never expose them on `0.0.0.0`. (App diagnostics endpoints: `secrets-config.md`.)
  Verify: from outside the host, curl the Traefik dashboard and the Coolify port → connection refused / 401, never 200.
- **[HARDEN] One normalizing hop, no smuggling surface** — terminate TLS and parse
  requests at Traefik; don't stack a second proxy that interprets
  `Content-Length`/`Transfer-Encoding` differently, and keep a body-size limit at
  the edge (`dos-limits.md`). Divergent parsing between hops is request smuggling.
  Verify: edge and app agree on ambiguous `Content-Length`/`Transfer-Encoding`; an oversized body is rejected at the edge.
- **[HARDEN] HSTS and TLS terminate at the edge** — Traefik serves HTTPS only,
  redirects HTTP→HTTPS, and sets HSTS; the app-level HSTS header (`web-nextjs.md`)
  is a backstop, not the primary. Verify: `curl -I http://APP` → 308 to https; `curl -I https://APP` shows `Strict-Transport-Security`.
