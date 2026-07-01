# Plan 007: Make observability pluggable — new observability.md (contract + template + playbooks)

> **Executor instructions**: Follow step by step, verify each step, honor STOP
> conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**: `git diff --stat b48283b..HEAD -- stack.md rules.md skills.md AGENTS.md CLAUDE.md README.md`

## Status

- **Priority**: P2
- **Effort**: M
- **Risk**: LOW
- **Depends on**: 005 (soft — registry rows)
- **Category**: docs / direction
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

Crash and analytics tooling is hardcoded to Sentry + PostHog in `stack.md` and
`rules.md` with no alternatives, even though a project may need Firebase
(Google ecosystem), Amplitude/Mixpanel (deep behavioural), or Aptabase
(privacy-first, works in Expo Go). This plan creates `observability.md` as a
pluggable domain: router + **Observability Swap Contract** + template +
playbooks, keeping Sentry (crash) and PostHog (analytics) as the loud defaults.

## Current state

- No `observability.md` exists. Sentry + PostHog are referenced in `stack.md`
  (matrix rows: Crash | Sentry, Analytics | PostHog) and `rules.md` (defaults +
  verification gates: "Sentry and PostHog are wired if the app has reached a
  testable MVP").
- **Verified options**: `plans/research-findings.md` → "Analytics, crash and
  observability" table (PostHog default, Sentry default, Firebase/Amplitude/
  Mixpanel/Aptabase/OpenPanel/Bugsnag/LogRocket alternates, backend Datadog/
  Axiom/Better Stack/OTel). Verified: `posthog/skills` ✓ + `posthog.com/llms.txt` ✓;
  `getsentry/skills` ✓ + `docs.sentry.io/llms.txt` ✓.

**Style**: mirror the contract+template shape of `plans/002` / `plans/003`.

## The Observability Swap Contract (canonical — reuse verbatim)

1. Never send PII, tokens, secrets, or full auth headers in analytics/crash
   events; scrub before sending.
2. Initialize crash + analytics before first render so early errors are caught.
3. Upload source maps / symbolication artifacts so stack traces are readable.
4. Honor analytics opt-out / consent where required (GDPR/ATT).
5. Backend observability keys live in server env only.

## Scope

**In scope**: create `observability.md`; edit `stack.md` (matrix rows →
swappable pointer), `rules.md` (defaults + doc-discipline pointer, keep the
verification gate), `skills.md` (registry rows), `AGENTS.md`, `CLAUDE.md`,
`README.md` (Files list), `plans/README.md` (status).

**Out of scope**: all other docs.

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Verify llms.txt | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence | `grep -n "<pattern>" observability.md` | matches |
| Even fences | `grep -c '```' <file>` | even |
| Scope | `git status --porcelain` | only in-scope |

## Steps

### Step 1: Create `observability.md`

Sections: intro (defaults Sentry crash + PostHog analytics) → `## Choosing crash +
analytics` → `## Observability Swap Contract` (the 5 invariants) → `## Add a
provider` (template) → `## Playbooks`: Sentry (default crash), PostHog (default
analytics), then alternates (Firebase Crashlytics, Amplitude, Mixpanel, Aptabase,
OpenPanel, Bugsnag, LogRocket) and backend (Datadog, Axiom, Better Stack, OTel).
Pull each option's verified skill/llms from `research-findings.md`; curl any
`(verify)` URL before writing.

**Verify**: `ls observability.md`; `grep -c "Observability Swap Contract" observability.md` → 1; `grep -ci "sentry\|posthog" observability.md` ≥ 2; even fences.

### Step 2: Soften the hardcoded framing

- `stack.md` matrix: `| Crash | Sentry (default) | ... swap via observability.md |`
  and `| Analytics | PostHog (default) | ... swap via observability.md |`.
- `rules.md`: keep the "Sentry and PostHog are wired ..." verification gate but
  note "or the chosen observability.md providers"; add a Documentation
  Discipline line "Read observability.md before crash/analytics work."

**Verify**: `grep -c "observability.md" stack.md rules.md` → each ≥ 1; Sentry +
PostHog still named as defaults (`grep -ci "sentry" stack.md` ≥ 1).

### Step 3: Wire index + registry + plan status

- `AGENTS.md`, `CLAUDE.md`, `README.md`: add `observability.md`.
- `skills.md` registry: append observability rows.
- `plans/README.md`: row 007 = DONE.

**Verify**: `grep -c "observability.md" AGENTS.md CLAUDE.md README.md` → each ≥ 1.

## Done criteria

- [ ] `ls observability.md`; contract + template + Sentry/PostHog playbooks + alternates present
- [ ] Sentry + PostHog still the named defaults; not deleted from `stack.md`
- [ ] `grep -c "observability.md" stack.md rules.md AGENTS.md CLAUDE.md README.md` → each ≥ 1
- [ ] Registry rows appended; only verified URLs/slugs
- [ ] Even fences; `git status` only in-scope; `plans/README.md` row 007 = DONE

## STOP conditions

- The `stack.md` matrix rows don't match (drift) — STOP.
- Firebase llms.txt confirmed 404 in research — do NOT invent one; use the skill
  or "check docs".

## Maintenance notes

- Keep alternates as short playbook stubs + llms.txt, not full guides, to limit
  the maintenance surface.
- The contract's PII-scrubbing rule is the security-load-bearing line; reviewers
  scrutinize it.
