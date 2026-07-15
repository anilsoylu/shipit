# Security: AI features / OpenRouter proxy

Part of `../security.md` (tags, threat model, and review method live there).

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
  private/link-local/metadata ranges (`ssrf.md`). Verify: grep the client bundle for the OpenRouter key → none; a fetch tool for `http://169.254.169.254/…` → refused.
- **[HARDEN] Cap output, budget, and indirect injection** — set `max_tokens`, a
  per-user spend cap, and a global kill-switch (`dos-limits.md`); treat RAG/upload/
  retrieved content as a prompt-injection source; strip PII from prompts. Verify: loop past the cap → 429 before spend continues.
- **[HARDEN] Sanitize what the proxy returns** — stream only the completion; never
  forward upstream error bodies/headers or echo the system prompt (leaks routing,
  cost, key metadata). Verify: force an upstream 401 → client sees a generic error, no provider detail.
