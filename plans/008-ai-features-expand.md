# Plan 008: Expand ai-features.md — contract + template + gateway/provider/media/agent/vector playbooks

> **Executor instructions**: Follow step by step, verify each step, honor STOP
> conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**: `git diff --stat b48283b..HEAD -- ai-features.md skills.md`
> Compare `ai-features.md` against the excerpts below; on mismatch, STOP.

## Status

- **Priority**: P2
- **Effort**: M
- **Risk**: LOW (keys-server-side rule must not be weakened)
- **Depends on**: 005 (soft — registry rows), 003 (soft — AI calls live in the backend)
- **Category**: docs / direction
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

`ai-features.md` is a solid backend-only AI playbook but names only OpenRouter +
generic "direct provider." Projects increasingly need a named menu: gateways
(OpenRouter, Vercel AI SDK, Portkey, LiteLLM, Helicone), direct providers
(Anthropic/OpenAI/Gemini/Groq/Together/Fireworks), media (Replicate/fal/
ElevenLabs), agent frameworks (Mastra/LangGraph), and RAG stores (pgvector/
Upstash Vector/Pinecone/Turso). This plan restructures it into router + **AI
Swap Contract** + template + playbooks, reinforcing the keys-server-side-only
invariant, with OpenRouter + Vercel AI SDK the default and Claude the default
model family.

## Current state

- `ai-features.md` — default rule (all provider calls in `apps/api`; never ship
  keys in the Expo app), when-to-add, API pattern (`POST /ai/<job>`), provider
  strategy (`modelProvider`, `model`, limits), cost controls, UX rules. Keep all
  of this; the security rules become the contract.
- **Verified options**: `plans/research-findings.md` → "AI / LLM integration"
  table (OpenRouter default + `openrouter.ai/docs/llms.txt` ✓; Vercel AI SDK;
  Anthropic/OpenAI/Gemini/Groq/Together/Fireworks; Replicate/fal/ElevenLabs;
  Mastra/LangGraph; pgvector/Upstash Vector/Pinecone/Turso). Most skills are
  `(verify)` — curl before writing; keep llms.txt where verified.

**Style**: keep `ai-features.md`'s voice; contract+template shape from plans 002/003.

## The AI Swap Contract (canonical — reuse verbatim)

1. Provider API keys live in server env ONLY (`apps/api`); never in the Expo
   bundle. The mobile app calls your API, never a provider directly.
2. Every AI endpoint verifies the user, then checks plan/credits/quota and a
   Redis rate limit before calling the provider.
3. Validate input length/type and cap max input; enforce request timeouts.
4. Stream responses through your API when the UX needs streaming.
5. Log cost metadata without sensitive prompt/PII data unless explicitly needed;
   feature-flag expensive endpoints.

## Scope

**In scope**: `ai-features.md` (restructure), `skills.md` (registry rows),
`plans/README.md` (status). Optionally note `data.md` (plan 009) as the home for
pgvector/RAG store details to avoid duplication — link, don't duplicate.

**Out of scope**: all other docs. Do NOT weaken the keys-server-side rule.

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Verify llms.txt | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence | `grep -n "<pattern>" ai-features.md` | matches |
| Even fences | `grep -c '```' ai-features.md` | even |
| Scope | `git status --porcelain` | only in-scope |

## Steps

### Step 1: Restructure `ai-features.md`

Order: intro (default OpenRouter gateway + Vercel AI SDK + Claude family) →
`## When to add` (existing) → `## AI Swap Contract` (the 5 invariants; fold in
the existing cost-controls + default-rule) → `## API pattern` (existing
`POST /ai/<job>`) → `## Add a provider` (template) → `## Playbooks`: gateways
(OpenRouter default, Vercel AI SDK, Portkey, LiteLLM, Helicone), direct
(Anthropic/OpenAI/Gemini/Groq/Together/Fireworks), media (Replicate/fal/
ElevenLabs), agents (Mastra/LangGraph), RAG (pgvector default + Upstash Vector/
Pinecone/Turso — link to `data.md` for store setup). Pull verified skill/llms
from `research-findings.md`; curl `(verify)` items first.

**Verify**: `grep -c "AI Swap Contract" ai-features.md` → 1; `grep -in "server" ai-features.md` shows the keys-server-side rule; `grep -ci "openrouter\|vercel ai\|claude" ai-features.md` ≥ 2; even fences.

### Step 2: Registry rows + plan status

- `skills.md` registry: append AI rows (verified only).
- `plans/README.md`: row 008 = DONE.

**Verify**: `grep -ci "openrouter" skills.md` ≥ 1.

## Done criteria

- [ ] `grep -c "AI Swap Contract" ai-features.md` → 1
- [ ] Keys-server-side-only invariant present (not weakened)
- [ ] Gateway/direct/media/agent/RAG playbooks present; OpenRouter+Vercel AI SDK default; Claude noted as default model family
- [ ] Registry rows appended; only verified URLs/slugs
- [ ] Even fences; `git status` only in-scope; `plans/README.md` row 008 = DONE

## STOP conditions

- Restructuring would move a provider key to the client or drop the
  server-side-keys rule — STOP.
- `ai-features.md` excerpts don't match (drift) — STOP.

## Maintenance notes

- RAG store details live in `data.md` (plan 009); keep only a pointer here to
  avoid two sources drifting.
- Default model family = Claude; keep that explicit as provider menus expand.
