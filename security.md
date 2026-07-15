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

**This file is a router.** It keeps the shared governance — the threat-model
ritual, the trust-boundary map, the severity legend, and the pre-ship review
pass — and delegates each attack class to a focused file under `security/`. The
review pass runs one pass per attack class; when hunting a class, load only that
class's file.

Every rule under `security/` is tagged **[BLOCKER]** (exploitable, data-loss, or
privilege escalation — fix before ship) or **[HARDEN]** (defense-in-depth — do
it, but it alone won't block a launch). Snippets show `// wrong` vs `// right`;
each rule ends with a `Verify:` you can actually run.

## When to read

- Before building any feature that touches auth, money, PII, uploads, or AI.
- At the `ship-checklist.md` **Security** gate before submission.
- When a security review is requested on existing code.

## Attack-class index

Load the file for the class you're building or hunting.

| File | Read when |
| --- | --- |
| `security/access-control.md` | authorization, ownership, IDOR, mass-assignment, tenant scoping |
| `security/injection.md` | SQL / command injection, path traversal, SSTI, unsafe deserialization, XXE, prototype pollution |
| `security/ssrf.md` | any server-side fetch of a user-influenced URL (webhooks, AI/RAG, image optimizer) |
| `security/auth-sessions.md` | token verification, cookies, session invalidation, auth rate-limiting, CSRF |
| `security/crypto-transport.md` | reset tokens / OTPs / session ids, password hashing, TLS, symmetric crypto |
| `security/web-nextjs.md` | Server Actions, CORS, CSP/headers, per-user caching, XSS/DOM sinks, redirects |
| `security/edge-proxy.md` | Traefik/Coolify edge — forwarded-header trust, host-header, admin plane, smuggling |
| `security/mobile-expo.md` | Expo bundle secrets, SecureStore, deep links, release hardening |
| `security/webhooks-payments.md` | Stripe/RevenueCat webhooks, entitlement grants, refund/dispute, race / double-spend |
| `security/storage-r2.md` | R2 signed URLs, upload constraints, serving user files |
| `security/ai-openrouter.md` | AI proxy, tool-call authz, model-output sinks, cost caps |
| `security/secrets-config.md` | committed secrets, diagnostics endpoints, env injection, error hygiene |
| `security/logging-privacy.md` | Sentry/PostHog scrubbing, logger redaction, PII minimization, audit trail |
| `security/dependencies.md` | dependency audit gate, lockfile, SRI, pinned actions/images |
| `security/dos-limits.md` | ReDoS, body/pagination limits, zip bombs, cost amplification |
| `security/per-framework.md` | **reference, not a hunt class** — the four security primitives per backend |

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
- **internet ↔ edge** (Traefik/Coolify): the reverse proxy is the only public
  hop. Forwarded headers are trusted only from the proxy; admin planes and infra
  ports never face the internet. See `security/edge-proxy.md`.

## Security review pass (before ship)

The `ship-checklist.md` Security gate. Method:

1. **Recon** — map architecture, trust boundaries, and input surfaces (fan out
   parallel read-only readers on a repo of any size).
2. **Hunt** — one pass per attack class (access control, injection, auth, SSRF,
   secrets, config, AI), each routed to its file under `security/`.
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
