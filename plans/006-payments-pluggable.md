# Plan 006: Make payments pluggable — contract + template + reference playbooks

> **Executor instructions**: Follow step by step, verify each step, honor STOP
> conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**: `git diff --stat b48283b..HEAD -- payments.md skills.md rules.md`
> Compare `payments.md` against the excerpts below; on mismatch, STOP.

## Status

- **Priority**: P1
- **Effort**: M
- **Risk**: MED (store-compliance rules must not be weakened)
- **Depends on**: 005 (soft — appends registry rows), 002/003 (references auth + backend contracts for server-side verification)
- **Category**: docs / direction
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

`payments.md` is a good Stripe-vs-RevenueCat decision tree but hardcodes those
two providers with no swap contract and no path for Adapty/Superwall/Paddle/
Polar/Lemon Squeezy — all of which a project may legitimately choose. This plan
restructures it into router + **Payments Swap Contract** (the invariants any
payment integration must satisfy, including the store-IAP mandate as a hard
rule) + add-your-own template + reference playbooks, keeping RevenueCat (IAP
default) and Stripe (web default) loud.

## Current state

- `payments.md` — decision tree (RevenueCat vs Stripe vs both), Stripe pattern,
  RevenueCat pattern, store-review guardrails, references. Keep all its rules;
  reorganize them under the contract + playbooks.
- **Verified options**: `plans/research-findings.md` → "Payments and
  monetization" table (RevenueCat default IAP, Stripe default web,
  Adapty/Superwall/Paddle/Polar/Lemon Squeezy situational) with verified
  llms.txt (`revenuecat.com/docs/llms.txt` ✓, `docs.stripe.com/llms.txt` ✓) and
  the verified skills `revenuecat/ai-toolkit` (RevenueCat) and `stripe/ai`
  (Stripe), plus `superwall/skills` for Superwall. Adapty/Paddle/Polar/Lemon
  Squeezy have no verified skills.sh pack — llms.txt only.
- Existing security rules to preserve verbatim in the contract:
  `payments.md` store-review guardrails (no routing in-app digital purchases
  through Stripe to dodge IAP; server-authoritative entitlements; webhook
  signature verification; mobile never sends trusted amount/entitlement/role).

**Style**: match `payments.md`'s current voice; reuse the contract+template
shape from `plans/002-auth-pluggable.md` (Auth Swap Contract) as the structural
model.

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Verify llms.txt | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence | `grep -n "<pattern>" payments.md` | matches |
| Even fences | `grep -c '```' payments.md` | even |
| Scope | `git status --porcelain` | only in-scope |

## Scope

**In scope**: `payments.md` (restructure), optionally create
`payments-revenuecat.md` and `payments-stripe.md` if you extract the two
patterns into playbooks (recommended for symmetry), `skills.md` (append registry
rows), `rules.md` (doc-discipline line), `plans/README.md` (status).

**Out of scope**: all other docs. Do NOT weaken any store-compliance rule.

## The Payments Swap Contract (canonical — reuse verbatim)

1. In-app digital goods/subscriptions consumed inside the app MUST use the
   platform IAP (Apple/Google) — via RevenueCat or native billing. Never route
   these through Stripe/web to avoid the store fee.
2. Entitlements are server-authoritative. The server verifies receipts/webhooks;
   the mobile client never sends a trusted price, amount, currency, entitlement,
   plan, or role.
3. Webhooks verify the provider signature before mutating entitlement/order
   state; store the raw body when signature verification needs it.
4. Product/price IDs come from server config or DB, not mobile input.
5. Secrets (provider secret keys, webhook secrets) live in server env only;
   only publishable/public keys ship in the Expo bundle.
6. If backend access is gated by entitlement, the check is server-side.

## Steps

### Step 1: Restructure `payments.md`

New section order: intro (default RevenueCat for IAP, Stripe for web) →
`## Choosing a payment path` (the existing decision tree) → `## Payments Swap
Contract` (the 6 invariants; fold in the store-review guardrails) →
`## Add a payment provider` (template skeleton like auth.md/backend.md) →
`## Provider playbooks` (links / sections for RevenueCat, Stripe, and a
situational note for Adapty/Superwall/Paddle/Polar/Lemon Squeezy).

**Verify**: `grep -c "Payments Swap Contract" payments.md` → 1; store-IAP rule
present (`grep -in "in-app" payments.md` shows the mandate); `grep -c '```' payments.md` even.

### Step 2: Playbooks

Keep RevenueCat and Stripe as full reference playbooks (either inline sections
or extracted `payments-revenuecat.md` / `payments-stripe.md` — pick one and be
consistent). Each: client setup, server verification, webhooks, env vars,
llms.txt reference from `research-findings.md`, and a contract-compliance
checklist. Add a short situational block for Adapty/Superwall (paywall A/B),
Paddle/Polar (merchant-of-record), Lemon Squeezy — each with its `(verify)`
llms.txt from `research-findings.md` (curl before writing).

**Verify**: `grep -ci "revenuecat" payments.md` ≥ 3; `grep -ci "adapty\|paddle\|polar\|superwall" payments.md` ≥ 1.

### Step 3: Registry rows + doc discipline + index

- `skills.md` registry: append verified payment rows (RevenueCat llms.txt,
  Stripe skill+llms.txt, situational providers' verified llms.txt).
- `rules.md` Documentation Discipline already says "Read payments.md before
  payment work" — leave it (still accurate).
- Set plan 006 row in `plans/README.md` to DONE.

**Verify**: `grep -ci "revenuecat" skills.md` ≥ 1.

## Done criteria

- [ ] `grep -c "Payments Swap Contract" payments.md` → 1
- [ ] Store-IAP mandate present as a contract invariant (not deleted)
- [ ] RevenueCat + Stripe playbooks present; situational providers listed
- [ ] Registry rows appended in `skills.md`; only verified URLs/slugs (curl-checked)
- [ ] `grep -c '```' payments.md` even
- [ ] `git status --porcelain` only in-scope
- [ ] `plans/README.md` row 006 = DONE

## STOP conditions

- Restructuring would drop a store-compliance or server-authoritative rule — STOP.
- The `payments.md` excerpts don't match the live file (drift) — STOP.

## Maintenance notes

- The store-IAP mandate is the highest-risk rule; a reviewer should confirm it
  survived as a hard contract invariant, not softened prose.
- Situational providers change fast; keep them a short list with llms.txt, not
  full playbooks, to limit maintenance.
