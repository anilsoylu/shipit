# Plan 001: Expand skill routing and add an extensible LLM docs registry

> **Executor instructions**: Follow this plan step by step. Run every
> verification command and confirm the expected result before moving to the
> next step. If anything in the "STOP conditions" section occurs, stop and
> report — do not improvise. When done, update the status row for this plan in
> `plans/README.md`.
>
> **Drift check (run first)**: `git diff --stat b48283b..HEAD -- skills.md`
> If `skills.md` changed since this plan was written, compare the "Current
> state" excerpts below against the live file before proceeding; on a mismatch,
> treat it as a STOP condition.

## Status

- **Priority**: P2
- **Effort**: M
- **Risk**: LOW
- **Depends on**: none
- **Category**: docs / dx
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

This repo is a docs pack that tells coding agents which specialized skill and
which authoritative docs to load before touching a subsystem. It currently
routes to Clerk, Expo, Redis, Supabase/Postgres, and RevenueCat skills, but has
**no Stripe skill routing** even though Stripe is a first-class payment path in
`payments.md`, and it references an `llms.txt` file for Clerk only. Agents
integrating Stripe, Drizzle, Sentry, PostHog, NativeWind, or any new service
have no pointer to that vendor's official machine-readable docs, so they work
from stale training data. This plan adds Stripe skill routing and a single,
**extensible** "LLM Docs Registry" — a table of `llms.txt` / `llms-full.txt`
endpoints plus a standing rule to look one up for any new integration — so the
pack scales to services it doesn't yet list by name.

## Current state

- `skills.md` — the only file this plan modifies. Relevant anchors:
  - Lines 176–224: the `## RevenueCat MCP` and `## RevenueCat Skills` sections
    (Stripe's new section will be added immediately after RevenueCat, since
    both are payments).
  - Lines 226–232: the existing `## Clerk Docs` section — the only `llms.txt`
    reference in the whole pack:
    ```markdown
    ## Clerk Docs

    Before implementing Clerk auth, use the official LLM docs:

    - https://clerk.com/docs/llms-full.txt

    Use this for current Expo SDK patterns, backend request verification,
    webhooks, auth flows, and token handling.
    ```
  - Lines 234–248: the `## Routing Rules` bullet list (a Stripe routing bullet
    will be appended here).
  - Lines 250–275: the `## skills.sh References` link list (Stripe directory
    links will be appended here).

- **Style conventions to match** (this file is the exemplar — copy its shape):
  - Each integration gets a `## Section`, a one-line purpose, a fenced `sh`
    install block using `npx skills add <owner>/<repo>` or
    `npx skills add https://github.com/<owner>/<repo> --skill <name>`, a bulleted
    list of individual skill names, and a "use when" sentence.
  - See the `## RevenueCat Skills` block at `skills.md:191-224` for the exact
    pattern to mirror for Stripe.

- **Do not invent skill package names or URLs.** Every `npx skills add` target
  and every `llms.txt` URL you add must be verified to exist before you write it
  (see Step 0). The Stripe skills live under the Stripe org on skills.sh
  (`https://www.skills.sh/stripe`); confirm the exact repo slug and skill names
  there.

## Commands you will need

| Purpose | Command | Expected on success |
|---------|---------|---------------------|
| Drift check | `git diff --stat b48283b..HEAD -- skills.md` | no output (unchanged) |
| Verify a URL is live | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence check | `grep -n "<pattern>" skills.md` | matching line(s) |
| Files unchanged outside scope | `git status --porcelain` | only `skills.md` and `plans/README.md` listed |

This is a markdown-only repo: there is no typecheck, test, or lint step.

## Suggested executor toolkit

- The Stripe plugin/MCP may be available in your environment (`search_stripe_documentation`,
  `stripe-best-practices` skill). If so, use it in Step 0 to confirm the current
  Stripe skill package name and the Stripe `llms.txt` URL rather than guessing.
- skills.sh directory root: https://www.skills.sh/ — browse the Stripe org to
  confirm slugs.

## Scope

**In scope** (the only files you should modify):
- `skills.md`
- `plans/README.md` (status row only)

**Out of scope** (do NOT touch):
- Every other doc. `payments.md` already documents the Stripe *pattern*; this
  plan only adds skill/docs *routing*, not payment logic.
- Do not add `better-auth`, Supabase-auth, or backend-language rows to the LLM
  Docs Registry — plans 002 and 003 append those in their own domains. Add only
  the base rows listed in Step 3.

## Git workflow

- Branch: `advisor/001-skills-llms-registry`
- Single commit is fine; message style matches repo history (short imperative,
  e.g. `Add Stripe skills and LLM docs registry to skills.md`).
- Do NOT push or open a PR unless the operator instructed it.

## Steps

### Step 0: Verify every external reference before writing it

For each URL and skill package you intend to add (Steps 1–3), confirm it exists:

- Skill packages: check `https://www.skills.sh/stripe` for the Stripe skill repo
  slug and skill names. Record the exact `npx skills add ...` command that the
  site shows.
- `llms.txt` URLs: run
  `curl -sS -o /dev/null -w "%{http_code}\n" <url>` for each candidate. Only
  include URLs that return `200`.

Confirmed-good anchor (already in the repo, no need to re-verify but a useful
sanity check that `curl` works): `https://clerk.com/docs/llms-full.txt`.

Candidate URLs to check (include only those returning 200; for any that
404, write the cell as `check <vendor docs site> for llms.txt` instead of a
dead link — never fabricate):

- Stripe: `https://docs.stripe.com/llms.txt`
- Expo: `https://docs.expo.dev/llms.txt`
- Drizzle: `https://orm.drizzle.team/llms.txt`
- NativeWind: `https://www.nativewind.dev/llms.txt`
- RevenueCat: `https://www.revenuecat.com/docs/llms.txt`
- PostHog: `https://posthog.com/llms.txt`
- Sentry: `https://docs.sentry.io/llms.txt`
- Fastify: `https://fastify.dev/llms.txt`

**Verify**: you have a written list of (name → verified URL or "check docs")
and the exact Stripe `npx skills add` command. If `https://www.skills.sh/stripe`
does not resolve or shows no Stripe skill, STOP and report — do not invent a
package name.

### Step 1: Add the Stripe Skills section

Insert a new `## Stripe Skills` section immediately **after** the
`## RevenueCat Skills` section (which currently ends near `skills.md:224`,
before `## Clerk Docs`). Mirror the RevenueCat block's structure. Fill the
install command and skill names with the values verified in Step 0. Include a
"Use Stripe skills only when …" sentence scoped to server-driven Stripe work
(checkout, payment intents, subscriptions, Connect, webhooks) consistent with
`payments.md`.

If the Stripe plugin/MCP is available in the environment, add one line noting it
can be used for Stripe API exploration and documentation search.

**Verify**: `grep -n "## Stripe Skills" skills.md` → one match, positioned
between `## RevenueCat Skills` and `## Clerk Docs`
(`grep -n "^## " skills.md` shows the ordering).

### Step 2: Generalize the Clerk Docs section into "LLM Docs Registry"

Rename the `## Clerk Docs` section (`skills.md:226-232`) to
`## LLM Docs Registry` and expand it. Keep the Clerk line. Add:

1. A **standing rule** at the top of the section, verbatim:
   > When integrating any external service, first look for its `llms.txt` or
   > `llms-full.txt` (usually at `<docs-domain>/llms.txt`) and its official
   > agent skill on https://www.skills.sh/. Prefer these over training-data
   > recall — they reflect the current SDK. If neither exists, use the official
   > docs linked in the relevant playbook.

2. A **registry table** with columns `Service | llms.txt / docs | Use for`,
   one row per integration, filled from Step 0's verified list. Include at
   minimum: Clerk (existing, `https://clerk.com/docs/llms-full.txt`), Stripe,
   Expo, Drizzle, NativeWind, RevenueCat, PostHog, Sentry, Fastify. Use the
   verified URL, or `check <vendor> docs for llms.txt` where Step 0 found none.

**Verify**:
- `grep -n "## LLM Docs Registry" skills.md` → one match
- `grep -n "## Clerk Docs" skills.md` → no matches (renamed)
- `grep -c "llms" skills.md` → count increased vs. the original (was 3 lines
  mentioning `llms`; must be higher now)

### Step 3: Add routing + directory links

- In `## Routing Rules` (`skills.md:234-248`), append two bullets:
  - one routing Stripe work to the Stripe skill + `payments.md`
  - one stating the standing rule: "New/unfamiliar integration: check its
    `llms.txt` and skills.sh entry first (see LLM Docs Registry)."
- In `## skills.sh References` (`skills.md:250-275`), append the Stripe
  directory link(s) verified in Step 0 (e.g. `https://www.skills.sh/stripe`).

**Verify**: `grep -in "stripe" skills.md` → at least 4 matches spanning the new
section, routing rule, and references list.

### Step 4: Update the plan index

Set this plan's row in `plans/README.md` to `DONE`.

**Verify**: `grep -n "001" plans/README.md` shows `DONE`.

## Test plan

No automated tests exist (markdown repo). Verification is the grep/curl gates in
each step plus:

- Render `skills.md` (any markdown previewer) and confirm the new table renders
  as a table and no fenced code block is left unclosed:
  `grep -c '```' skills.md` → an **even** number.

## Done criteria

ALL must hold:

- [ ] `grep -n "## Stripe Skills" skills.md` → exactly one match
- [ ] `grep -n "## LLM Docs Registry" skills.md` → exactly one match; `## Clerk Docs` gone
- [ ] Every URL added returns `200` via `curl` (or is written as a "check docs"
      pointer, never a dead link)
- [ ] `grep -c '```' skills.md` returns an even number (no broken code fences)
- [ ] `git status --porcelain` lists only `skills.md` and `plans/README.md`
- [ ] `plans/README.md` row 001 = DONE

## STOP conditions

Stop and report back (do not improvise) if:

- `https://www.skills.sh/stripe` does not resolve or shows no Stripe skill —
  do not fabricate a package name or install command.
- The `## Clerk Docs` section text does not match the excerpt above (file
  drifted).
- More than half of the candidate `llms.txt` URLs 404 — report and ask whether
  to ship the registry with mostly "check docs" pointers.

## Maintenance notes

- The LLM Docs Registry is the extension point: adding a new integration later
  means adding one row, not restructuring. Plans 002 (auth providers) and 003
  (backend languages) will append their own rows here.
- `llms.txt` URLs drift; the "check docs" fallback keeps the table honest when a
  vendor moves theirs. A reviewer should spot-check a couple of URLs per PR.
- Skill package slugs on skills.sh can change; the "verify before writing" rule
  in Step 0 is the guardrail — keep it in any future edit.
