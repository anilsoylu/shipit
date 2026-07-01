# Plan 009: Make the data layer pluggable — new data.md (contract + template + playbooks)

> **Executor instructions**: Follow step by step, verify each step, honor STOP
> conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**: `git diff --stat b48283b..HEAD -- backend.md stack.md rules.md skills.md AGENTS.md CLAUDE.md README.md`

## Status

- **Priority**: P2
- **Effort**: L
- **Risk**: MED (must not duplicate/contradict backend.md's DB guidance)
- **Depends on**: 005 (soft — registry rows), 003 (soft — pairs with backend playbooks)
- **Category**: docs / direction
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

The data layer (ORM, managed Postgres host, on-device/offline store, cache,
background jobs) is scattered: `backend.md` covers Drizzle/Postgres/Redis but
there is no menu for Prisma, Neon/Supabase/Turso hosts, on-device stores
(Expo SQLite/op-sqlite/WatermelonDB/PowerSync), or job runners (Trigger.dev/
QStash). This plan creates `data.md` as a pluggable domain: router + **Data Swap
Contract** + template + playbooks, keeping Postgres + Drizzle the default and
linking (not duplicating) `backend.md`'s shared Database/Redis guidance.

## Current state

- `backend.md` has `## Database` and `## Redis` sections (language-agnostic
  principles). `data.md` must LINK these, not restate them.
- No `data.md` exists.
- **Verified options**: `plans/research-findings.md` → "Database and ORM / data
  layer" (20 options). Verified: `prisma/skills` ✓ + `prisma.io/docs/llms.txt` ✓;
  `neondatabase/agent-skills` ✓ + `neon.com/llms.txt` ✓; `supabase/agent-skills` ✓;
  `redis/agent-skills` ✓; `upstash/skills` ✓; `docs.turso.tech/llms.txt` ✓;
  `expo/skills` ✓ + `docs.expo.dev/llms.txt` ✓. Drizzle has no verified vendor
  skill (llms.txt only, `(verify)`).

**Style**: contract+template shape from plans 002/003; terse.

## The Data Swap Contract (canonical — reuse verbatim)

1. Schema changes go through committed migrations, run before production deploy.
2. Index foreign keys, lookup fields, and uniqueness constraints; use
   transactions for multi-table writes.
3. DB/Redis secrets live in server env only; never in the Expo bundle.
4. On-device secrets go in `expo-secure-store`, never in an on-device DB or MMKV.
5. The on-device DB is a cache/offline copy; the server DB is the source of
   truth. Reconcile via sync, don't trust device state for authorization.

## Scope

**In scope**: create `data.md`; edit `backend.md` (add a pointer from Database/
Redis sections to `data.md` for host/ORM/on-device/jobs options — do NOT move
its rules), `stack.md` (DB/ORM matrix rows → swappable pointer), `rules.md`
(doc-discipline line), `skills.md` (registry rows), `AGENTS.md`, `CLAUDE.md`,
`README.md`, `plans/README.md` (status).

**Out of scope**: RAG/vector store SETUP belongs here but AI usage stays in
`ai-features.md` (plan 008) — link, don't duplicate. All other docs.

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Verify llms.txt/slug | `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | `200` |
| Presence | `grep -n "<pattern>" data.md` | matches |
| Even fences | `grep -c '```' <file>` | even |
| Scope | `git status --porcelain` | only in-scope |

## Steps

### Step 1: Create `data.md`

Sections: intro (default Postgres + Drizzle; managed host Neon or VDS Postgres) →
`## Choosing ORM + DB host + on-device store` → `## Data Swap Contract` (the 5
invariants) → `## Add a data option` (template) → `## Playbooks`:
- ORM: Drizzle (default), Prisma (strong-alt), Kysely (situational).
- Managed host: Neon (default), Supabase, Turso, PlanetScale.
- On-device: Expo SQLite (default), op-sqlite, WatermelonDB, PowerSync,
  Legend-State.
- Cache: Redis (self-host), Upstash (serverless).
- Jobs: Trigger.dev, QStash, Inngest, BullMQ.
Pull verified skill/llms from `research-findings.md`; curl `(verify)` items.
Add: "For server-side Database/Redis rules see `backend.md`'s Database and Redis
sections — this doc only adds host/ORM/on-device/jobs options."

**Verify**: `ls data.md`; `grep -c "Data Swap Contract" data.md` → 1; `grep -ci "drizzle\|neon\|prisma" data.md` ≥ 3; `grep -c "backend.md" data.md` ≥ 1 (links, not duplicates); even fences.

### Step 2: Soften matrix + wire

- `stack.md` matrix: mark DB and ORM rows swappable with a `data.md` pointer;
  keep Postgres + Drizzle as named defaults.
- `backend.md`: add one line in the Database section pointing to `data.md` for
  host/ORM/on-device/jobs options.
- `rules.md` Documentation Discipline: add "Read data.md before ORM, DB-host,
  on-device/offline, cache, or background-job choices."
- `AGENTS.md`, `CLAUDE.md`, `README.md`: add `data.md`.
- `skills.md` registry: append data rows.
- `plans/README.md`: row 009 = DONE.

**Verify**: `grep -c "data.md" stack.md backend.md rules.md AGENTS.md CLAUDE.md README.md` → each ≥ 1; Postgres + Drizzle still named defaults.

## Done criteria

- [ ] `ls data.md`; contract + template + ORM/host/on-device/cache/jobs playbooks present
- [ ] `data.md` links `backend.md` for server DB/Redis rules (no duplication)
- [ ] Postgres + Drizzle still the named defaults in `stack.md`
- [ ] `grep -c "data.md" stack.md backend.md rules.md AGENTS.md CLAUDE.md README.md` → each ≥ 1
- [ ] Registry rows appended; only verified URLs/slugs
- [ ] Even fences; `git status` only in-scope; `plans/README.md` row 009 = DONE

## STOP conditions

- `data.md` would restate/contradict `backend.md`'s Database/Redis rules instead
  of linking them — STOP and link instead.
- `backend.md` Database section doesn't match the plan-003-era shape (drift) — STOP.

## Maintenance notes

- Boundary: `backend.md` owns server-side DB/Redis *rules*; `data.md` owns the
  *menu of options*. Keep that split to avoid two drifting sources.
- On-device sync options (WatermelonDB/PowerSync) are heavy; keep them
  situational stubs, not full guides.
