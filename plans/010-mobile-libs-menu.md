# Plan 010: Add mobile-libs.md — curated Expo/RN client library menu

> **Executor instructions**: Follow step by step, verify each step, honor STOP
> conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**: `git diff --stat b48283b..HEAD -- stack.md skills.md AGENTS.md CLAUDE.md README.md`

## Status

- **Priority**: P3
- **Effort**: M
- **Risk**: LOW (advisory menu, not security-load-bearing)
- **Depends on**: 005 (soft — registry rows; stack-picker links this)
- **Category**: docs / dx
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

The most common source of agent thrash on a mobile build is guessing which
client library to use for state, data-fetching, forms, storage, lists,
animation, and native capability. `mobile-libs.md` gives a decisive default menu
with strong-alt/situational swaps and per-lib skill + `llms.txt` pointers, so the
agent stops guessing and loads authoritative docs. This is a **menu, not a
contract domain** — advisory only.

## Current state

- No `mobile-libs.md` exists. `stack.md` names NativeWind + React Native
  Reusables but nothing about state/data/forms/lists/animation libraries.
- **Verified options**: `plans/research-findings.md` → "Mobile state,
  data-fetching and storage" (14) + "Mobile UI, styling and native capability"
  (14). Verified: `expo/skills` ✓ (expo-secure-store, Expo Router, native
  modules); `zustand.docs.pmnd.rs/llms.txt` ✓; `trpc.io/llms.txt` ✓;
  `docs.expo.dev/llms.txt` ✓. Many library skills are `(verify)` and several are
  suspect aggregator slugs (`agents-inc/skills`, `jezweb/claude-skills`) —
  research-findings.md flags these; prefer the vendor's own skill or drop it.

**Style**: a curated table doc; terse; no swap contract (advisory).

## Scope

**In scope**: create `mobile-libs.md`; edit `stack.md` (pointer),
`skills.md` (registry rows), `AGENTS.md`, `CLAUDE.md`, `README.md`,
`plans/README.md` (status). If plan 005 has run, also link it from
`stack-picker.md`'s "Client libs" row (that row already exists in the picker
draft — just confirm it points here).

**Out of scope**: contract-domain docs. Do NOT invent a "contract" for this
domain — it is intentionally advisory.

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Verify llms.txt/slug | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence | `grep -n "<pattern>" mobile-libs.md` | matches |
| Even fences | `grep -c '```' <file>` | even |
| Scope | `git status --porcelain` | only in-scope |

## Steps

### Step 1: Create `mobile-libs.md`

Sections:
- `## Defaults` table: TanStack Query (server-state), Zustand (client state),
  react-hook-form + Zod (forms), expo-secure-store (secrets), react-native-mmkv
  (fast KV), FlashList v2 (lists), Reanimated + Gesture Handler (animation/
  gesture), Expo Router (navigation). Columns: Concern | Default | Skill |
  llms.txt.
- `## When to swap`: SWR, tRPC, Jotai, Redux Toolkit, TanStack Form, Valibot,
  Legend-State, op-sqlite, expo-sqlite — one line each.
- `## Animation & graphics`: Reanimated/Gesture Handler default; Skia/Moti/Lottie
  situational.
- `## Native capability`: expo-dev-client, expo-camera, expo-image,
  expo-notifications (default primitives); VisionCamera (advanced camera).
- A standing note: "expo-secure-store for secrets; never MMKV/AsyncStorage/
  on-device DB for tokens." (This one hard rule is worth stating even in an
  advisory doc.)

Pull verified skill/llms from `research-findings.md`. For any `(verify)` or
suspect aggregator slug, curl it; if it is not the official vendor skill, drop
the skill and keep only the llms.txt.

**Verify**: `ls mobile-libs.md`; `grep -ci "tanstack\|zustand\|flashlist\|reanimated" mobile-libs.md` ≥ 3; `grep -c "expo-secure-store" mobile-libs.md` ≥ 1; even fences.

### Step 2: Wire index + registry + plan status

- `stack.md`: add a pointer to `mobile-libs.md` for client-library choices.
- `AGENTS.md`, `CLAUDE.md`, `README.md`: add `mobile-libs.md`.
- `skills.md` registry: append the verified mobile-lib rows.
- `plans/README.md`: row 010 = DONE.

**Verify**: `grep -c "mobile-libs.md" stack.md AGENTS.md CLAUDE.md README.md` → each ≥ 1.

## Done criteria

- [ ] `ls mobile-libs.md`; Defaults table + swap notes + animation + native-capability sections present
- [ ] The expo-secure-store-for-secrets note is present
- [ ] Suspect aggregator slugs verified or dropped (no unverified slug shipped)
- [ ] `grep -c "mobile-libs.md" stack.md AGENTS.md CLAUDE.md README.md` → each ≥ 1
- [ ] Registry rows appended; even fences; `git status` only in-scope
- [ ] `plans/README.md` row 010 = DONE

## STOP conditions

- You cannot verify a library's official skill and are unsure of the llms.txt —
  ship the llms.txt as "check docs" and drop the skill rather than guess.

## Maintenance notes

- Advisory doc: it will drift as RN libraries churn. Keep entries to one line +
  pointer; do not write tutorials here (the skill/llms.txt carry the detail).
- This is P3 because it is not security-load-bearing; ship it last.
