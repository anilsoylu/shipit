# Plan 003: Make backend pluggable — contract + template + reference playbooks

> **Executor instructions**: Follow this plan step by step. Run every
> verification command and confirm the expected result before moving on. If a
> "STOP conditions" item occurs, stop and report — do not improvise. When done,
> update this plan's row in `plans/README.md`.
>
> **Drift check (run first)**:
> `git diff --stat b48283b..HEAD -- backend.md stack.md rules.md workflow.md AGENTS.md CLAUDE.md README.md skills.md`
> Also confirm plan 002 has landed: `ls auth.md` must exist (this plan links to
> the Auth Swap Contract it defines). If `auth.md` is missing, STOP — run 002
> first. If any listed file changed since this plan was written, compare the
> "Current state" excerpts against the live files; on a mismatch, STOP.

## Status

- **Priority**: P1
- **Effort**: L
- **Risk**: MED (many new docs with version-specific, language-specific claims)
- **Depends on**: 002 (hard — references `auth.md`'s Auth Swap Contract)
- **Category**: docs / direction / dx
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

`backend.md` is a single Fastify+TypeScript+Drizzle playbook, and `stack.md`,
`rules.md`, and `workflow.md` all hardcode "Fastify TypeScript service." If a
project wants Hono, Go, Rust, or Python (FastAPI) — a legitimate, common
choice — the entire backend doc becomes useless and the agent has no guidance
that keeps the API as secure and deployable as the default. Enumerating every
language is a losing game (the next request is Elixir, then Java). This plan
converts the backend doc into a **language-agnostic Backend Swap Contract** (the
invariants any API must satisfy), an **"add your own backend" template** (so an
agent can produce a correct playbook for *any* stack on demand), and complete
**reference playbooks** for Fastify (default), Hono, Go, Rust, and Python. The
contract is what makes a Go or Python backend obey the same auth, migration, and
deploy rules as the Fastify default.

## Current state

Files and exact lines edited (verify before editing):

- `backend.md`
  - Line 3: `Default backend: standalone Fastify TypeScript API on Coolify/VDS.`
  - Lines 6–14 (`## Architecture` diagram) — keep, but relabel to be
    provider/language-agnostic where it names Clerk/Fastify.
  - Lines 16–36 (`## API Shape`, the `src/` TS layout) — **Fastify-specific**;
    move into `backend-fastify.md`.
  - Lines 38–45 (Rules under API Shape: `/health` no auth, don't trust client
    userId, validate bodies, typed DTOs) — **language-agnostic**; these become
    part of the Backend Swap Contract.
  - Lines 47–54 (`## Auth`) — already handled by plan 002 (replaced with an
    `auth.md` pointer). Do not re-edit; if plan 002 left it as a pointer, leave
    it.
  - Lines 56–74 (`## Database`, `## Redis`) — **mostly language-agnostic
    principles**; keep as shared guidance in `backend.md`, and let each playbook
    name the concrete tool (Drizzle vs sqlc vs SQLAlchemy).
  - Lines 76–103 (`## Coolify Deploy`, `## Security Defaults`) —
    **language-agnostic**; keep in `backend.md` as part of the contract.
- `stack.md`
  - Line 7 (Product Shape): `- Expo mobile app plus Fastify API.`
  - Line 20 (Stack Matrix, API row): `| API | Fastify standalone | Low boilerplate, fast endpoints, good plugin model. |`
- `rules.md`
  - Line 14: `- API: standalone Fastify TypeScript service.`
  - Line 50: `- If the user has not specified backend details, use Fastify, Drizzle, Postgres, Redis, Clerk.`
  - Documentation Discipline (lines 60–70) — the `backend.md` read rule exists;
    extend it to mention picking a backend playbook.
- `workflow.md`
  - Lines 53–62 (Scaffold → API): `- Create Fastify TypeScript service.` and
    the Drizzle/Clerk lines under it.
- `AGENTS.md`, `CLAUDE.md`, `README.md` — index/Files lists (already list
  `backend.md`; add the new `backend-*.md` playbooks).
- `skills.md` `## LLM Docs Registry` (from plan 001) — append rows for Hono,
  Go, Rust, Python/FastAPI, and their migration tools.

**Style conventions**: match `backend.md`'s current voice — terse imperative
rules, `txt` fenced layouts, `## Official References` link lists. Keep each
playbook self-contained but short; do not turn them into tutorials.

## The Backend Swap Contract (canonical text — reuse verbatim)

Every backend, in any language, must satisfy these. Write once in `backend.md`:

1. `GET /health` responds without auth.
2. Protected routes verify the auth token **server-side on every request**, per
   the Auth Swap Contract in `auth.md` (JWKS / provider SDK / introspection).
3. Never accept `userId`, ownership, price, credits, entitlement, or role from
   client input; derive identity from the verified token, other trusted values
   from server config or DB.
4. Validate request bodies/params before any DB write.
5. Return typed DTOs, not raw DB rows, when fields are sensitive.
6. Schema changes go through committed migrations, run before production deploy;
   use transactions for multi-table writes; index foreign keys, lookup fields,
   and uniqueness constraints.
7. Webhooks verify the provider signature before mutating state; store the raw
   body when signature verification needs it.
8. Secrets (DB URL, Redis URL, provider keys, webhook secrets) live in server
   env only, never in the Expo bundle.
9. Deployable on Coolify/VDS: a Dockerfile (or Coolify-supported build), the
   `/health` route, an explicit env var list, a documented migration command,
   and logs visible in Coolify.
10. HTTPS in production; CORS scoped to known clients (not `*`); rate limits
    before expensive public endpoints; never log tokens, cookies, payment
    secrets, provider keys, or full auth headers.

## The "add your own backend" template (canonical skeleton)

Put this in `backend.md`. Each `backend-<stack>.md` is this filled in:

```markdown
# Backend Playbook: <Stack>

Use <Stack> when: <selection criteria>.

## Project layout
- Directory/module structure (equivalent to the Fastify src/ layout).

## Libraries
- HTTP framework, DB access/ORM, migration tool, validation, Redis client.

## Auth verification
- How this stack verifies the token per auth.md's Auth Swap Contract
  (e.g. JWKS validation library, provider SDK).

## Database & migrations
- ORM/driver and the exact migration command (goes in the deploy runbook).

## Health, webhooks, security
- Health route, webhook signature verification, CORS/rate-limit approach.

## Coolify deploy
- Dockerfile notes specific to this stack; build/run commands; env list.

## Env vars
- Secret vs public.

## Contract compliance
- Restate the 10 Backend Swap Contract invariants; confirm each is met.

## Official References
- Framework, ORM, migration, and JWKS-verify docs + llms.txt (from the registry).
```

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Drift check | `git diff --stat b48283b..HEAD -- <files>` | no output |
| Verify a URL | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence check | `grep -rn "<pattern>" .` | matching lines |
| Even code fences | `grep -c '```' <file>` | even number |
| Scope check | `git status --porcelain` | only in-scope files |

Markdown-only repo: no typecheck/test/lint.

## Suggested executor toolkit

- Verify each stack's library and migration-tool names against current docs
  before writing them; do not rely on memory for versions or import paths.
  Candidate authoritative sources to `curl`/read (check first, use "check docs"
  if unavailable): Hono (`https://hono.dev/`), Go chi/echo + `golang-migrate`,
  Rust axum + `sqlx`, Python FastAPI (`https://fastapi.tiangolo.com/`) +
  Alembic. Add each stack's `llms.txt` to the registry if it has one.
- The auth-verification section of each playbook must align with the provider
  chosen in `auth.md` — for non-TS backends the norm is JWKS-based JWT
  verification; name a maintained JWKS library for that language.

## Scope

**In scope**:
- Create: `backend-fastify.md`, `backend-hono.md`, `backend-go.md`,
  `backend-rust.md`, `backend-python.md`
- Restructure: `backend.md` (into router + contract + template + shared
  Database/Redis/Coolify/Security guidance; move the TS `src/` layout out)
- Edit: `stack.md` (Product Shape + API matrix row), `rules.md` (2 lines +
  doc-discipline), `workflow.md` (Scaffold API block), `AGENTS.md`, `CLAUDE.md`,
  `README.md`, `skills.md` (registry rows)
- Edit: `plans/README.md` (status row)

**Out of scope** (do NOT touch):
- `auth.md` and `auth-*.md` (plan 002 owns them) — link to `auth.md`, don't
  edit it. If `backend.md`'s `## Auth` is already a pointer from plan 002, leave
  it.
- `payments.md`, `ai-features.md`, `app-store.md`, `aso.md`,
  `ship-checklist.md`, `dynamic-workflows-claude-code.md`.
- Do NOT drop any existing security/deploy rule during restructuring. Every rule
  currently in `backend.md` must survive either in the contract or a playbook.
  Removing one is a STOP condition.

## Git workflow

- Branch: `advisor/003-backend-pluggable`
- Commit per logical unit (contract restructure, then playbooks, then wiring) or
  one commit; message: short imperative (e.g.
  `Make backend pluggable: add backend contract and Fastify/Hono/Go/Rust/Python playbooks`).
- Do NOT push or open a PR unless instructed.

## Steps

### Step 1: Restructure `backend.md` into router + contract + template

New section order:
- Intro: "Default backend is standalone Fastify TypeScript on Coolify/VDS. The
  backend is pluggable — pick a stack, then follow its playbook. Whatever you
  pick must satisfy the Backend Swap Contract below."
- `## Architecture`: keep the existing diagram (lines 6–14) but relabel
  provider/framework-specific nodes to be generic (e.g. "API" instead of
  Fastify-only framing; "Auth provider verification (see auth.md)").
- `## Choosing a backend`: short decision list — Fastify (TS, default, richest
  plugin ecosystem), Hono (TS, portable/edge runtimes), Go (performance, single
  binary, strong concurrency), Rust (max performance/safety), Python/FastAPI
  (Python ecosystem, ML-adjacent work).
- `## Backend Swap Contract`: the 10 invariants verbatim from this plan (fold in
  the current lines 38–45 rules).
- `## Shared guidance`: keep the language-agnostic Database, Redis, Coolify
  Deploy, and Security Defaults content (current lines 56–103), noting each
  playbook names the concrete tool.
- `## Add a new backend`: the template skeleton verbatim, plus: "To support a
  stack not listed, copy the skeleton into `backend-<stack>.md`, satisfy every
  contract invariant, add its rows to the LLM Docs Registry in `skills.md`, and
  link it from this file and the index files."
- `## Backend playbooks`: links to the five `backend-*.md` files.

**Verify**:
- `grep -c "Backend Swap Contract" backend.md` ≥ 1
- `grep -c "Coolify" backend.md` ≥ 1 (deploy guidance preserved)
- `grep -c "src/" backend.md` → 0 (the TS layout moved out)
- `grep -c '```' backend.md` even

### Step 2: Create `backend-fastify.md` (extract the default)

Fill the template for Fastify. Move the `src/` layout and plugin structure from
the old `backend.md:16-36` here. Keep it the canonical default: Fastify + Drizzle
+ Postgres + Redis, Clerk verification via `auth.md`. This is the reference other
playbooks mirror.

**Verify**: `ls backend-fastify.md`; `grep -c "src/" backend-fastify.md` ≥ 1
(layout landed here); `grep -c "Contract compliance" backend-fastify.md` ≥ 1.

### Step 3: Create `backend-hono.md`

Fill the template for Hono (TS). Emphasize runtime portability (Node/Bun/edge),
Drizzle for DB, zod validation, JWKS/provider verification for auth. Note it
pairs well with better-auth (TS) from plan 002. Verify Hono specifics against
`https://hono.dev/`.

**Verify**: `ls backend-hono.md`; `grep -ci "hono" backend-hono.md` ≥ 3.

### Step 4: Create `backend-go.md`, `backend-rust.md`, `backend-python.md`

Each fills the full template with **verified** stack-specific choices:
- `backend-go.md`: an HTTP router (e.g. chi or echo), `pgx`/`sqlc` for
  Postgres, `golang-migrate` for migrations, a JWKS library for token
  verification, single-binary Dockerfile.
- `backend-rust.md`: axum, `sqlx` (with `sqlx migrate`), a `jsonwebtoken`/JWKS
  approach for auth, multi-stage Dockerfile.
- `backend-python.md`: FastAPI, SQLAlchemy/SQLModel + Alembic migrations,
  Pydantic validation, PyJWT/python-jose for JWKS verification, uvicorn/gunicorn
  in the Dockerfile.

Every one must include the `## Contract compliance` checklist mapping to the 10
invariants, and its auth-verification section must reference `auth.md`.

**Verify**: `ls backend-go.md backend-rust.md backend-python.md`;
`grep -l "Contract compliance" backend-go.md backend-rust.md backend-python.md`
lists all three; `grep -l "auth.md" backend-go.md backend-rust.md backend-python.md`
lists all three.

### Step 5: Soften the hardcoded Fastify lines

- `stack.md:7`: `- Expo mobile app plus a backend API (default Fastify; swappable — see backend.md).`
- `stack.md:20`: `| API | Fastify standalone (default) | Low boilerplate, fast; swappable to Hono/Go/Rust/Python — see backend.md. |`
- `rules.md:14`: `- API: standalone backend service; default Fastify TypeScript, pluggable per backend.md.`
- `rules.md:50`: `- If the user has not specified backend details, use Fastify, Drizzle, Postgres, Redis. If they request another stack, follow backend.md and its swap contract.`
- `rules.md` Documentation Discipline: extend the backend line to:
  `- Read backend.md before API, DB, auth, Redis, Docker, or Coolify work, and pick the matching backend-<stack>.md playbook.`
- `workflow.md:54`: change `- Create Fastify TypeScript service.` to
  `- Create the backend service using the chosen backend-<stack>.md playbook (default Fastify TypeScript).`
  Keep the health-route / Drizzle / Redis / Dockerfile bullets but note they are
  the Fastify-default specifics; other stacks follow their own playbook.

**Verify**:
- `grep -c "backend.md" stack.md rules.md workflow.md` → each ≥ 1
- `grep -in "fastify" stack.md rules.md` still shows Fastify as the named
  default (default preserved, not erased)

### Step 6: Wire the new docs into the index files

- `AGENTS.md`: under the `docs/backend.md` entry, add a note that it links to
  `backend-fastify.md`, `backend-hono.md`, `backend-go.md`, `backend-rust.md`,
  `backend-python.md`. Match the existing list format.
- `CLAUDE.md` (root bridge): same note in its Primary instructions list.
- `README.md` `## Files`: extend the `backend.md` line and add:
  `- backend-<stack>.md: reference playbooks — fastify (default), hono, go, rust, python.`

**Verify**: `grep -c "backend-fastify.md" AGENTS.md CLAUDE.md README.md` → total
≥ 3 (present in each or noted); `ls backend-*.md` → 5 files.

### Step 7: Append registry rows and update the plan index

- In `skills.md` `## LLM Docs Registry`, append verified rows for Hono, Go
  (chi/echo + golang-migrate), Rust (axum + sqlx), Python (FastAPI + Alembic).
  `curl` each URL; use "check docs" for any 404.
- Set plan 003's row in `plans/README.md` to `DONE`.

**Verify**: `grep -ci "fastapi\|hono\|axum" skills.md` ≥ 2; `plans/README.md`
row 003 = DONE.

## Test plan

No automated tests. Verification = the grep/curl/ls gates above, plus:
- Every new/edited markdown file has an even ` ``` ` fence count.
- Cross-link integrity: every `backend-*.md` linked from `backend.md` exists
  (`ls backend-*.md` → 5); `backend.md` links to `auth.md` for auth.
- Restructure safety: diff the old `backend.md` rules against the new contract
  and confirm no security/deploy rule was dropped
  (`git show b48283b:backend.md` vs. the new `backend.md` + playbooks — every
  bullet accounted for).

## Done criteria

ALL must hold:

- [ ] `ls backend.md backend-fastify.md backend-hono.md backend-go.md backend-rust.md backend-python.md` → all present
- [ ] `grep -c "Backend Swap Contract" backend.md` ≥ 1; each `backend-*.md` has a `## Contract compliance` section
- [ ] `grep -c "src/" backend.md` → 0 (TS layout moved to `backend-fastify.md`)
- [ ] No dropped rules: `grep -c "Coolify" backend.md` ≥ 1 and every non-TS-specific rule from the old `backend.md` exists in the contract or a playbook
- [ ] `grep -c "backend.md" stack.md rules.md workflow.md` → each ≥ 1; Fastify still named as default
- [ ] Each non-TS playbook references `auth.md` for token verification
- [ ] All added URLs return 200 or are "check docs" pointers
- [ ] `grep -c '```' <each edited/created md>` even
- [ ] `git status --porcelain` lists only in-scope files
- [ ] `plans/README.md` row 003 = DONE

## STOP conditions

Stop and report (do not improvise) if:

- `auth.md` does not exist (plan 002 not run) — do not fabricate the Auth Swap
  Contract; run 002 first.
- The `backend.md` excerpts don't match the live file (drift).
- You cannot verify a stack's framework/migration tool from current docs — do
  not invent import paths or CLI commands; mark that detail "check docs" and
  report.
- Restructuring would drop a security or deploy rule that has no new home.

## Maintenance notes

- The Backend Swap Contract + template are the durable core; the five reference
  playbooks are the volatile layer (library versions, CLI commands drift).
  Reviewers should concentrate version-risk scrutiny on the playbooks, not the
  contract.
- Adding a new stack later = copy the template into `backend-<stack>.md`,
  satisfy the contract, add registry rows, link from `backend.md` and the index.
  No restructuring — this is the whole point of the contract-first design.
- The auth-verification section of each backend playbook is coupled to
  `auth.md`; if the Auth Swap Contract changes, re-check all five playbooks.
- High maintenance surface acknowledged in `plans/README.md`: five backend docs
  can rot silently (no CI). Consider a periodic doc-freshness check or trimming
  rarely used playbooks if they fall out of date.
