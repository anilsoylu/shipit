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
- **[BLOCKER] Resolve once, pin the IP, reject non-global-unicast** — a check that
  inspects the hostname *string* is bypassed by alternate IP encodings
  (`2130706433`, `0177.0.0.1`, `[::ffff:169.254.169.254]`, `0.0.0.0`, `[::]`) and by
  DNS rebinding (the record flips to a private IP between check and fetch), so
  validate the *resolved address* and pin the connection to it.
  ```ts
  // wrong: validated the name, then fetch() re-resolves — rebinding wins, encodings slip past
  if (allowed(new URL(url).hostname)) await fetch(url)
  // right: pin to the resolved IP; reject anything that isn't a public (global-unicast)
  // address; TLS/SNI still use the original hostname, so HTTPS keeps working
  import { Agent } from "undici"
  const dispatcher = new Agent({ connect: { lookup: (host, opts, cb) =>
    dns.lookup(host, opts, (e, addr, fam) =>
      e ? cb(e, addr, fam)
        : isGlobalUnicast(addr) ? cb(null, addr, fam)
        : cb(new Error("blocked SSRF target"), addr, fam)) } })
  await fetch(url, { dispatcher, redirect: "manual" })   // re-validate on every redirect hop
  ```
  Verify: `2130706433`, `0177.0.0.1`, `[::ffff:a9fe:a9fe]`, `0.0.0.0`, `::`, `::1`, RFC1918, and IPv6 ULA/link-local hosts are each refused — before and after a redirect.

Surfaces that must route through the check above:

- **Webhook / callback URLs** the user configures — validate on save *and* at fetch time (DNS can change).
- **AI / RAG fetch tools** — model- or user-driven URL fetches; see `ai-openrouter.md`.
- **Next image optimizer** — `images.remotePatterns` lists exact hosts, never `**`; no user-controlled host in a server `fetch`. See `web-nextjs.md`. Verify: grep `remotePatterns` for `**`.
- **XML/SVG parsing** — external entities are a second SSRF vector; see the XXE rule in `injection.md`.
- **Cloud metadata** — beyond `169.254.169.254`, GCP exposes `metadata.google.internal` (the name resolves into link-local, so the resolved-IP check catches it) and honors a `Metadata-Flavor` header; on AWS prefer IMDSv2 (token-required). Any host resolving into a private/link-local range is refused by the rule above.
