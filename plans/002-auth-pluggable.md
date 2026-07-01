# Plan 002: Make auth pluggable — contract + template + reference playbooks

> **Executor instructions**: Follow this plan step by step. Run every
> verification command and confirm the expected result before moving on. If a
> "STOP conditions" item occurs, stop and report — do not improvise. When done,
> update this plan's row in `plans/README.md`.
>
> **Drift check (run first)**:
> `git diff --stat b48283b..HEAD -- stack.md rules.md backend.md workflow.md AGENTS.md CLAUDE.md README.md skills.md`
> If any of these changed since this plan was written, compare the "Current
> state" excerpts against the live files before proceeding; on a mismatch, treat
> it as a STOP condition.

## Status

- **Priority**: P1
- **Effort**: L
- **Risk**: MED (edits many index files; risk is dead cross-links, not runtime)
- **Depends on**: 001 (soft — appends rows to the LLM Docs Registry it creates)
- **Category**: docs / direction / dx
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

The pack hardcodes Clerk as the only auth path across `stack.md`, `backend.md`,
`rules.md`, and `workflow.md`, and `rules.md` even says "Do not re-litigate
these defaults," which actively steers agents away from a different provider a
project may require (better-auth, Supabase Auth, or a custom JWT server). There
is no guidance for swapping auth, so an agent asked to use better-auth has
nothing to follow. This plan keeps Clerk as the loud default but adds an **auth
swap contract** (the security invariants any provider must satisfy), an
**"add your own provider" template**, and **reference playbooks** for Clerk
(default), better-auth, Supabase Auth, and custom JWT. The contract is the
important part: it is what keeps a non-Clerk backend as safe as the default.

## Current state

Files and the exact lines this plan edits (verify each before editing):

- `stack.md`
  - Line 19 (Stack Matrix, Auth row): `| Auth | Clerk | Fast Expo auth and polished session handling. |`
  - Lines 44–46, 52–53 (Environment Rules): list `CLERK_SECRET_KEY`,
    `CLERK_WEBHOOK_SECRET`, and "Clerk publishable key is public."
- `backend.md`
  - Lines 8–12 (architecture diagram) include `-> Clerk backend verification`.
  - Lines 47–54, the `## Auth` section (Clerk-specific) — this content is
    **moved** into the new `auth-clerk.md`:
    ```markdown
    ## Auth

    - Clerk owns identity.
    - Mobile uses Clerk Expo SDK.
    - API verifies incoming authenticated requests.
    - API maps Clerk user ID to internal user records when the product needs app-specific profile data.
    - Webhooks must verify signatures before mutating user records.
    ```
- `rules.md`
  - Line 15: `- Auth: Clerk for user identity.`
  - Line 34: `- Verify Clerk auth on the API before protected reads/writes.`
  - Lines 60–70 (Documentation Discipline) — add an `auth.md` read rule.
  - Line 78 (Verification Gates) references "Protected API routes reject
    unauthenticated requests" — keep, it is provider-agnostic already.
- `workflow.md`
  - Line 58: `- Add Clerk auth middleware.`
- `AGENTS.md` — the read list (lines 6–17) and does not yet list `auth.md`.
- `CLAUDE.md` (repo root, the bridge file) — its "Primary instructions" list
  (the sibling-paths list near the top) does not list `auth.md`.
- `README.md` — the `## Files` list (lines 17–29) does not list the auth docs.
- `skills.md` — the `## LLM Docs Registry` section created by plan 001 (rename of
  the old `## Clerk Docs` at `skills.md:226-232`). Append better-auth and
  Supabase-auth rows. If plan 001 has NOT run yet (section still named
  `## Clerk Docs`), add the rows under that section instead and note it.

**Style conventions to match**: every playbook in this repo is terse, uses
`## Section` headers, bullet rules, fenced `txt`/`sh` blocks, and ends with an
`## Official References` link list. Use `backend.md` and `payments.md` as the
structural exemplars — match their voice (imperative, no fluff).

## The auth swap contract (canonical text — reuse verbatim)

Every auth playbook must satisfy these invariants. Write them once in `auth.md`
and reference them from each provider playbook:

1. The Expo client obtains a session/token from the provider's client SDK and
   stores tokens only in secure storage (`expo-secure-store`), never
   AsyncStorage or plaintext.
2. Every protected API request carries the provider's bearer token or session.
3. The API verifies the token **server-side on every protected request** — via
   the provider's JWKS (for JWTs), the provider's server SDK verifier, or a
   token-introspection endpoint. An unverified token is never trusted.
4. The API derives actor identity (user id) **only** from the verified token —
   never from a `userId` in the request body, query, or header.
5. The API maps the provider user id to an internal user record via an
   idempotent upsert when app-specific profile data is needed.
6. Auth webhooks (user created/updated/deleted, session events) verify the
   provider's signature before mutating user records.
7. Only publishable/public keys ship in the Expo bundle. Secret keys, JWKS
   signing secrets, and webhook secrets stay in API server env only.
8. `GET /health` never requires auth.

## The "add your own provider" template (canonical skeleton)

Put this skeleton in `auth.md`. Each `auth-<provider>.md` is this skeleton
filled in:

```markdown
# Auth Playbook: <Provider>

Use <Provider> when: <one or two selection criteria>.

## Client (Expo)
- Package(s) and install command.
- Provider wrapper/provider component and where it mounts.
- Sign-up / sign-in / sign-out flow.
- How the client retrieves the token for API calls.
- Secure token storage (expo-secure-store).

## Server verification
- How the API validates the token (JWKS URL / server SDK / introspection).
- Middleware/plugin shape (framework-agnostic description).

## User mapping
- Provider user id -> internal user record (idempotent upsert).

## Webhooks
- Events to handle and how signatures are verified.

## Env vars
- Public (bundle-safe): ...
- Secret (server only): ...

## Contract compliance
- Restate the 8 contract invariants and confirm this provider meets each.

## Official References
- Provider docs, llms.txt, and skill (from the LLM Docs Registry).
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

- Verify every SDK detail against the provider's `llms.txt` / official docs
  before writing it — SDK APIs drift and this repo has no compiler to catch a
  wrong import.
  - Clerk Expo: https://clerk.com/docs/llms-full.txt (confirmed live)
  - better-auth: check `https://www.better-auth.com/llms.txt` and
    https://www.better-auth.com/docs; its Expo integration and secure-store
    plugin specifics MUST come from there, not memory.
  - Supabase Auth: https://supabase.com/docs (and its skills on skills.sh).
- If a Clerk skill set is installed, `skills.md` already routes it; reuse those
  names in `auth-clerk.md`'s references.

## Scope

**In scope**:
- Create: `auth.md`, `auth-clerk.md`, `auth-betterauth.md`, `auth-supabase.md`,
  `auth-custom-jwt.md`
- Edit: `backend.md` (auth section → pointer, diagram label), `stack.md`
  (matrix row, env note), `rules.md` (2 default/boundary lines + doc-discipline
  line), `workflow.md` (line 58), `AGENTS.md`, `CLAUDE.md`, `README.md`,
  `skills.md` (registry rows)
- Edit: `plans/README.md` (status row)

**Out of scope** (do NOT touch):
- `payments.md`, `ai-features.md`, `app-store.md`, `aso.md`,
  `ship-checklist.md`, `dynamic-workflows-claude-code.md`.
- Backend runtime/language docs — plan 003 owns those. Do NOT create
  `backend-*.md` here. Where an auth playbook needs to show server verification,
  describe it framework-agnostically and link to `backend.md`, do not embed a
  Fastify-only or Go-only handler.
- Do NOT weaken any security rule while "softening" the Clerk defaults. Turning
  "verify Clerk auth" into "verify auth" is fine; deleting the verify
  requirement is a STOP condition.

## Git workflow

- Branch: `advisor/002-auth-pluggable`
- Commit per logical unit (new playbooks, then index wiring) or one commit;
  message style: short imperative (e.g. `Make auth pluggable: add auth contract and provider playbooks`).
- Do NOT push or open a PR unless instructed.

## Steps

### Step 1: Create `auth.md` (router + contract + template)

Sections, in order:
- Intro: "Default auth is Clerk. Auth is swappable — pick a provider, then
  follow its playbook. Whatever you pick must satisfy the Auth Swap Contract
  below."
- `## Choosing a provider`: a short decision list — Clerk (fastest Expo auth,
  hosted, default), better-auth (self-hosted, own your data, TS backend),
  Supabase Auth (already using Supabase, or want Postgres-native auth), custom
  JWT (full control, you own all security — discouraged unless required).
- `## Auth Swap Contract`: the 8 invariants verbatim from this plan.
- `## Add a new provider`: the template skeleton verbatim from this plan, plus:
  "To support a provider not listed here, copy the skeleton into
  `auth-<provider>.md`, satisfy every contract invariant, add its row to the LLM
  Docs Registry in `skills.md`, and link it from this file and the index files."
- `## Provider playbooks`: links to the four `auth-*.md` files.

**Verify**: `ls auth.md` exists; `grep -c "Auth Swap Contract" auth.md` ≥ 1;
`grep -c '```' auth.md` is even.

### Step 2: Create `auth-clerk.md` by extracting Clerk content

Fill the template for Clerk. Move the Clerk `## Auth` bullets from
`backend.md:47-54` here and expand them into the template sections: Clerk Expo
SDK on the client, `getToken()` for API calls, server verification via Clerk's
backend SDK / JWKS, user mapping, webhooks (signature-verified), env vars
(`CLERK_PUBLISHABLE_KEY` public; `CLERK_SECRET_KEY`, `CLERK_WEBHOOK_SECRET`
secret). Reference `https://clerk.com/docs/llms-full.txt` and the Clerk skills
already in `skills.md`. End with the contract-compliance checklist.

**Verify**: `ls auth-clerk.md`; `grep -c "CLERK_SECRET_KEY" auth-clerk.md` ≥ 1.

### Step 3: Create `auth-betterauth.md`

Fill the template for better-auth. **All SDK specifics (Expo client plugin,
secure-store integration, server setup, session verification, Drizzle adapter,
env `BETTER_AUTH_SECRET` / `BETTER_AUTH_URL`) must be taken from better-auth's
official docs/llms.txt verified in this step**, not from memory. Note that
better-auth runs in the API process (TS), so it pairs naturally with the Fastify
default and the Hono option from plan 003; for non-TS backends, note that the
API must verify better-auth's session/JWT via its documented mechanism.

**Verify**: `ls auth-betterauth.md`; `grep -ci "better-auth\|better auth" auth-betterauth.md` ≥ 3.

### Step 4: Create `auth-supabase.md` and `auth-custom-jwt.md`

- `auth-supabase.md`: Supabase Auth client in Expo, JWT verification server-side
  via Supabase JWKS/JWT secret (the API is separate from Supabase, so it
  verifies the JWT — do not rely on RLS as the only gate for the custom API),
  env `EXPO_PUBLIC_SUPABASE_URL` + anon key (public), `SUPABASE_JWT_SECRET`
  (server). Verify specifics against Supabase docs.
- `auth-custom-jwt.md`: the "you own security" path — password hashing with a
  memory-hard algorithm (argon2/bcrypt), short-lived access JWT + refresh token
  rotation, secure-store on device, verify signature server-side. Add an
  explicit warning: prefer a managed provider unless a hard requirement forces
  custom auth.

**Verify**: both files exist; each has a `## Contract compliance` section
(`grep -l "Contract compliance" auth-supabase.md auth-custom-jwt.md` lists both).

### Step 5: Soften the hardcoded Clerk lines (without weakening security)

Make these exact edits:

- `stack.md:19`: change the Auth row to
  `| Auth | Clerk (default) | Fast Expo auth; swappable — see auth.md for better-auth, Supabase, custom JWT. |`
- `stack.md` Environment Rules: after the Clerk key lines, add one line:
  `- Auth env vars are provider-specific; see auth.md for the chosen provider's public vs secret keys.`
- `backend.md:8-12` diagram: change `-> Clerk backend verification` to
  `-> Auth provider verification (default Clerk; see auth.md)`.
- `backend.md:47-54`: replace the `## Auth` body with a pointer:
  "Auth is pluggable. Default is Clerk. Pick a provider and follow its playbook;
  every provider must satisfy the Auth Swap Contract. See `auth.md`." Keep the
  provider-agnostic API rules (verify server-side, don't trust client userId)
  either here or ensure they live in the contract.
- `rules.md:15`: `- Auth: Clerk by default; pluggable per auth.md.`
- `rules.md:34`: `- Verify auth server-side on the API before protected reads/writes (default Clerk; see auth.md).`
- `rules.md` Decision Rules: add a bullet:
  `- If the user has not specified auth, use Clerk. If they request another provider, follow auth.md and its swap contract.`
- `rules.md` Documentation Discipline: add
  `- Read auth.md before auth provider, login, session, token, or webhook work.`
- `workflow.md:58`: `- Add auth middleware per auth.md (default Clerk).`

**Verify**:
- `grep -rn "Clerk owns identity" backend.md` → no matches (section replaced)
- `grep -rin "verify" rules.md` still shows an auth-verification requirement
  (the security rule survived)
- `grep -c "auth.md" rules.md stack.md backend.md workflow.md` → each ≥ 1

### Step 6: Wire the new docs into the index files

- `AGENTS.md` read list: add `- docs/auth.md` (with a sub-note that it links to
  `auth-clerk.md`, `auth-betterauth.md`, `auth-supabase.md`,
  `auth-custom-jwt.md`). Match the existing `docs/<name>` format.
- `CLAUDE.md` (root bridge file) "Primary instructions" list: add
  `- docs/auth.md` in the same style; keep the "If this file is inside docs/,
  use sibling paths" note applicable.
- `README.md` `## Files` list: add
  `- auth.md: auth provider selection, swap contract, and per-provider playbooks (Clerk default, better-auth, Supabase, custom JWT).`

**Verify**: `grep -c "auth.md" AGENTS.md CLAUDE.md README.md` → each ≥ 1.

### Step 7: Append registry rows and update the plan index

- In `skills.md` `## LLM Docs Registry` (from plan 001), append rows for
  better-auth and Supabase Auth with their verified `llms.txt`/docs URLs
  (`curl` them first; use a "check docs" pointer if 404). If plan 001 has not
  run and the section is still `## Clerk Docs`, add the two links there and note
  it in your report.
- Set plan 002's row in `plans/README.md` to `DONE`.

**Verify**: `grep -ci "better-auth" skills.md` ≥ 1; `plans/README.md` row 002 =
DONE.

## Test plan

No automated tests. Verification = the grep/curl/ls gates above, plus:
- Every new/edited markdown file has an even count of ` ``` ` fences.
- Render `auth.md` and confirm the contract list and template render cleanly.
- Cross-link check: `grep -rl "auth.md" .` shows the index files, and every
  `auth-*.md` referenced in `auth.md` exists (`ls auth-*.md` → 4 files).

## Done criteria

ALL must hold:

- [ ] `ls auth.md auth-clerk.md auth-betterauth.md auth-supabase.md auth-custom-jwt.md` → all present
- [ ] `grep -c "Auth Swap Contract" auth.md` ≥ 1 and each `auth-*.md` has a `## Contract compliance` section
- [ ] `grep -rn "Clerk owns identity" .` → no matches (Clerk section moved, not duplicated)
- [ ] Security preserved: `grep -in "verify" rules.md backend.md auth.md` shows a server-side auth-verification requirement
- [ ] `grep -c "auth.md" AGENTS.md CLAUDE.md README.md rules.md stack.md backend.md workflow.md` → each ≥ 1
- [ ] All added URLs return 200 or are "check docs" pointers (no dead links)
- [ ] `grep -c '```' <each edited/created md>` even
- [ ] `git status --porcelain` lists only in-scope files
- [ ] `plans/README.md` row 002 = DONE

## STOP conditions

Stop and report (do not improvise) if:

- The Clerk `## Auth` block in `backend.md` does not match the excerpt (drift).
- You cannot verify better-auth's Expo integration from its official docs/llms
  (do not invent the client plugin API or import paths).
- A "softening" edit would require deleting a security requirement rather than
  making it provider-neutral.
- Wiring `auth.md` into an index file reveals the index format has changed from
  the excerpts (e.g. `AGENTS.md` no longer uses the `docs/<name>` list).

## Maintenance notes

- The Auth Swap Contract is the durable core; provider playbooks are the
  volatile layer. When a provider's SDK changes, only its `auth-<provider>.md`
  needs updating — the contract and template stay put. Reviewers should scope
  version-risk to the individual playbooks.
- Adding a new provider later = copy the template into `auth-<provider>.md`,
  satisfy the contract, add a registry row, link from `auth.md` and the index
  files. No restructuring.
- Watch that plan 003's backend playbooks reference this contract (not a
  Clerk-specific flow) for their auth-verification sections.
