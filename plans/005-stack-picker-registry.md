# Plan 005: Add stack-picker.md and enrich the LLM Docs Registry into a full multi-domain directory

> **Executor instructions**: Follow step by step, run every verification, honor
> STOP conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**:
> `git diff --stat b48283b..HEAD -- skills.md stack.md rules.md README.md AGENTS.md CLAUDE.md`
> Also confirm plan 001 has run: `grep -c "LLM Docs Registry" skills.md` → ≥ 1.
> If 001 has NOT run, do it first (this plan extends the registry it creates).

## Status

- **Priority**: P1
- **Effort**: L
- **Risk**: MED (many verified URLs/slugs to transcribe correctly)
- **Depends on**: 001 (hard — extends the LLM Docs Registry), 004 (soft — `workflow.md` Step 0 links the picker)
- **Category**: docs / dx / direction
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

This is the comfort centerpiece. A vibe-coder wants to start building without a
research detour. `stack-picker.md` gives a one-sentence "accept all defaults"
fast path plus 6 default-first questions, so choosing a stack takes seconds and
every choice auto-points to its agent skill + `llms.txt`. The enriched LLM Docs
Registry in `skills.md` becomes the single machine-readable directory the picker
and every domain doc link into, so the agent loads authoritative docs instead of
stale training data.

## Current state

- **Verified data source**: `plans/research-findings.md` — the workflow-verified
  option tables (ground-truth verified skills.sh packs + llms.txt at the top,
  per-domain tables below with `✓` / `(verify)` / `—` trust flags). **This is
  your data. Do not invent slugs or URLs; copy from here, and for any `(verify)`
  item run the verification in Step 0.**
- **Ready draft**: `plans/drafts/stack-picker.draft.md` — a complete draft of
  `stack-picker.md`. Its HTML-comment header lists the exact skill-slug
  corrections required (e.g. Stripe = `stripe/ai` not `stripe/agent-toolkit`;
  Sentry = `getsentry/skills`; RevenueCat has no skill). Use it as the body,
  apply those corrections against the ground truth.
- `skills.md` — has `## LLM Docs Registry` from plan 001 (base rows). Expand it.
- Index files to wire the picker into: `stack.md` (intro pointer + matrix rows
  marked swappable), `rules.md` (decision rule: unspecified stack → run picker),
  `README.md` (Files list + "start here"), `AGENTS.md` and `CLAUDE.md` (read
  lists, picker near top).

**Style**: match `skills.md`'s existing section/table voice; match the terse doc
voice for `stack-picker.md` (the draft already does).

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Verify a skills.sh slug | `curl -sS -o /dev/null -w "%{http_code}\n" https://www.skills.sh/<owner>/<repo>` | `200` (real) |
| Control test (must 404) | `curl -sS -o /dev/null -w "%{http_code}\n" https://www.skills.sh/this/does-not-exist` | `404` |
| Verify an llms.txt | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence | `grep -n "<pattern>" skills.md` | matching lines |
| Even fences | `grep -c '```' <file>` | even |
| Scope | `git status --porcelain` | only in-scope files |

## Scope

**In scope**: create `stack-picker.md`; edit `skills.md`, `stack.md`, `rules.md`,
`README.md`, `AGENTS.md`, `CLAUDE.md`, `plans/README.md`.

**Out of scope**: `payments.md`, `observability.md`, `ai-features.md`, `data.md`,
`mobile-libs.md` — plans 006–010 own those. `plans/drafts/` and
`plans/research-findings.md` are read-only inputs. Do NOT edit `workflow.md`
(plan 004 owns it).

## Steps

### Step 0: Re-verify before writing (the whole point of this batch)

Run the control test (a known-bad slug must return 404 — proves your detection
works). Then, for every `(verify)`-flagged skill slug and llms.txt you intend to
write into `stack-picker.md` or the registry, `curl` it. Rules:
- Skill slug not 200, or not the official vendor's skill → drop the skill, keep
  only the llms.txt.
- llms.txt not 200 / not real markdown → write `check <vendor> docs for llms.txt`
  instead of a dead link.
- `✓`-flagged items in `research-findings.md` are already verified — no re-curl
  needed.

**Verify**: you have a reconciled list of (choice → verified skill | verified
llms.txt | "check docs"). If the control test does NOT return 404, STOP (your
verification is unreliable).

### Step 1: Create `stack-picker.md`

Use `plans/drafts/stack-picker.draft.md` as the body. Strip its HTML comment.
Apply the slug corrections it names, using the reconciled list from Step 0 for
the "Load the docs before you build" table. Keep the fast path, the 6 questions,
the filled Stack Matrix, and the swaps.

**Verify**:
- `ls stack-picker.md`
- `grep -c "Fast path" stack-picker.md` → ≥ 1
- `grep -c "stripe/agent-toolkit\|getsentry/sentry-for-ai" stack-picker.md` → 0 (corrections applied)
- `grep -c '```' stack-picker.md` even

### Step 2: Enrich the LLM Docs Registry in `skills.md`

Expand `## LLM Docs Registry` into the authoritative directory. For each domain
in `research-findings.md` (data, payments, observability, ai, mobile-state,
mobile-ui, infra), add rows: `Service | skill (npx cmd) | llms.txt | use-for`.
Only write verified values (Step 0). Prefer one row per option; group by domain
with sub-headings. Add a `Verified: 2026-07-01` note at the top of the registry
so future readers know the as-of date (answers the freshness open-question).

**Verify**:
- `grep -c "Verified: 2026-07-01" skills.md` → 1
- `grep -ci "revenuecat\|posthog\|drizzle\|trigger" skills.md` → several new rows
- `grep -c '```' skills.md` even

### Step 3: Wire the picker into the index files

- `stack.md`: add an intro line pointing to `stack-picker.md` as the interactive
  front door; keep the Stack Matrix but mark it "defaults — swap via
  stack-picker.md". (Coexist; do NOT delete the matrix.)
- `rules.md` Decision Rules: add
  `- If the user has not specified the stack, run stack-picker.md and take its defaults.`
- `README.md`: add `stack-picker.md` to `## Files` and name it the recommended
  starting point after install.
- `AGENTS.md` and `CLAUDE.md`: add `docs/stack-picker.md` near the top of the
  read/Primary-instructions lists.

**Verify**: `grep -c "stack-picker.md" stack.md rules.md README.md AGENTS.md CLAUDE.md` → each ≥ 1.

### Step 4: Update the plan index

Set plan 005 row in `plans/README.md` to DONE.

## Done criteria

- [ ] `ls stack-picker.md` present; fast path + 6 questions + matrix present
- [ ] Control-test 404 confirmed; every written slug/URL verified 200 or written as "check docs"
- [ ] No corrected-away slugs remain (`grep` in Step 1 → 0)
- [ ] `skills.md` registry expanded with per-domain rows + `Verified: 2026-07-01`
- [ ] `grep -c "stack-picker.md" stack.md rules.md README.md AGENTS.md CLAUDE.md` → each ≥ 1
- [ ] `grep -c '```' <each edited/created md>` even
- [ ] `git status --porcelain` only in-scope files
- [ ] `plans/README.md` row 005 = DONE

## STOP conditions

- Control test does not return 404 (verification broken) — STOP.
- `plans/research-findings.md` or the draft is missing — STOP; do not re-research
  from memory.
- More than half the `(verify)` URLs in a domain 404 — report before shipping a
  registry full of "check docs" pointers.

## Maintenance notes

- Registry granularity chosen: full per-option table in `skills.md`; the picker
  carries only a curated anchor subset (per the synthesis recommendation).
- The `Verified: <date>` stamp is the freshness signal in lieu of CI. Re-verify
  on a cadence; slugs and llms.txt URLs drift.
- Plans 006–010 append their domain rows here; keep the section structure stable
  so they slot in cleanly.
