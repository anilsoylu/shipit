# Security: Denial of service & resource limits

Part of `../security.md` (tags, threat model, and review method live there).

Unbounded work is the theme: one request must not exhaust memory, the event loop,
disk, or the bill for everyone else.

- **[HARDEN] Bound every unit of work — body, page, and payload size** — set a
  request body limit per route (Fastify `bodyLimit`, Next route config), cap
  pagination `limit` server-side (clamp, don't trust the client), and bound array/
  string sizes at the Zod boundary. Large-upload routes stream to storage, they
  don't buffer in memory.
  ```ts
  const app = Fastify({ bodyLimit: 1_048_576 })         // 1 MB; raise per-route only where needed
  const Query = z.object({ limit: z.number().int().min(1).max(100).default(20) }).strict()
  ```
  Verify: POST a 10 MB JSON body → 413; `?limit=100000` → clamped to the max, not a full-table scan.
- **[HARDEN] No catastrophic regex on user input (ReDoS)** — Node runs regex on the
  single event-loop thread; a pathological pattern (nested quantifiers `(a+)+`,
  `(.*)*`) on attacker input stalls *every* request. Never compile a user-supplied
  string into a `RegExp`; keep patterns linear (or use `re2`) and bound input length
  before matching.
  ```ts
  // wrong
  new RegExp(req.query.q)                   // user controls the pattern
  /^(\w+\s?)+$/.test(input)                 // catastrophic backtracking on a long crafted string
  // right
  import RE2 from "re2"
  new RE2(FIXED_PATTERN).test(input.slice(0, 256))   // linear engine + bounded length
  ```
  Verify: feed a ~50-char adversarial string to the validator → completes in <10ms (no event-loop stall); grep `new RegExp(` fed by `req.`.
- **[HARDEN] Cap archive extraction (zip bomb)** — beyond zip-slip (`injection.md`),
  bound total uncompressed size and entry count before/while extracting; a small
  archive can inflate to gigabytes and fill the disk. Verify: extract a crafted highly-compressed archive → aborts at the size/entry cap.
- **[HARDEN] Cap cost-amplifying downstreams per user** — expensive external calls
  (OpenRouter tokens, R2 egress, email/SMS sends, image processing) get a per-user
  rate/spend cap and a global kill-switch in Redis; one user must not run up the
  bill or exhaust a provider quota. (AI specifics: `ai-openrouter.md`.) Verify: loop an expensive endpoint past the cap → 429 before further spend, and the global switch halts all callers.
