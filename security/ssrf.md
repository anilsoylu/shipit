# Security: SSRF (server-side request forgery)

Part of `../security.md` (tags, threat model, and review method live there).

Any time the server fetches a URL the client can influence — webhook/callback
targets, AI/RAG document fetches, image/URL tools, the Next image optimizer — the
attacker can aim that fetch at internal services and the cloud metadata endpoint.
This is the one rule; the surfaces that must apply it are listed below.

- **[BLOCKER] Validate every server-fetched URL, before and after redirects** —
  restrict to http/https, block private/link-local/metadata ranges (`10/8`,
  `127/8`, `169.254.0.0/16` — Hetzner metadata lives there too), cap redirects,
  and re-validate the host *after* each redirect and DNS resolution (rebinding).
  ```ts
  // wrong: fetch a user-supplied URL directly
  await fetch(req.body.url)
  // right: scheme + resolved-IP allowlist, checked again on every hop
  const u = new URL(req.body.url)
  if (!/^https?:$/.test(u.protocol)) throw forbidden()
  if (isPrivateOrLinkLocal(await resolve(u.hostname))) throw forbidden()   // 10/8, 127/8, 169.254/16, ::1, fc00::/7
  await fetch(u, { redirect: "manual" })   // follow manually, re-validating each Location
  ```
  Verify: point a server-fetch feature at `http://169.254.169.254/…` and at an internal IP → both refused, before and after a redirect.

Surfaces that must route through the check above:

- **Webhook / callback URLs** the user configures — validate on save *and* at fetch time (DNS can change).
- **AI / RAG fetch tools** — model- or user-driven URL fetches; see `ai-openrouter.md`.
- **Next image optimizer** — `images.remotePatterns` lists exact hosts, never `**`; no user-controlled host in a server `fetch`. See `web-nextjs.md`. Verify: grep `remotePatterns` for `**`.
- **XML/SVG parsing** — external entities are a second SSRF vector; see the XXE rule in `injection.md`.
