# Plan 004: Merge workflow.md + dynamic-workflows-claude-code.md into one improved workflow doc

> **Executor instructions**: Follow step by step, run every verification, honor
> STOP conditions. Update this plan's row in `plans/README.md` when done.
>
> **Drift check (run first)**:
> `git diff --stat b48283b..HEAD -- workflow.md dynamic-workflows-claude-code.md AGENTS.md CLAUDE.md README.md`
> On any change vs. the excerpts below, STOP and reconcile.

## Status

- **Priority**: P1
- **Effort**: M
- **Risk**: LOW
- **Depends on**: none (references `stack-picker.md` via a forward link that plan 005 creates — see Step 3)
- **Category**: docs / dx
- **Planned at**: commit `b48283b`, 2026-07-01

## Why this matters

The pack has two overlapping workflow docs: `workflow.md` (the Expo ship
workflow: intake → scope → scaffold → build → release) and
`dynamic-workflows-claude-code.md` (a raw copy of the Claude Code "dynamic
workflows" announcement blog — marketing prose, a Bun anecdote, customer
quotes). A vibe-coder gets no value from a pasted press release, and two docs
named "workflow" compete. This plan merges them into one improved `workflow.md`
that keeps the ship phases and adds a practical "orchestrate it with Claude
Code" layer (when to fan out subagents, when to trigger a dynamic
workflow/ultracode, load skills, use plan mode/hooks/background tasks/
checkpoints), distilled from the blog and the community changelog — then deletes
the blog dump.

## Current state

- `workflow.md` — the Expo ship workflow (7 phases). Keep its structure.
- `dynamic-workflows-claude-code.md` — a verbatim blog announcement; to be
  deleted after its actionable content is distilled into `workflow.md`.
- A **verified, ready-to-use merged draft** already exists at
  `plans/drafts/workflow-merged.draft.md`. Its "Claude Code Orchestration
  Primer" and per-phase orchestration notes were built by reading both docs plus
  the two sources below and were confirmed against real Claude Code feature
  flags. **Use that draft as the body of the new `workflow.md`**, applying the
  one required change described in its HTML-comment header.
- Sources the draft was distilled from (both verified reachable in the research
  run):
  - https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
  - https://github.com/marckrenn/claude-code-changelog (README + `meta/cli-surface.md`)
- Index files that reference the docs and must be updated:
  - `AGENTS.md` read list (lists `docs/workflow.md` and `docs/dynamic-workflows-claude-code.md`).
  - `CLAUDE.md` (root bridge) "Primary instructions" list (same two entries).
  - `README.md` `## Files` list (lines describing `workflow.md` and `dynamic-workflows-claude-code.md`).

**Style**: terse shipit voice; imperative bullets; `## ` sections. The draft
already matches it.

## Commands you will need

| Purpose | Command | Expected |
|---------|---------|----------|
| Drift check | `git diff --stat b48283b..HEAD -- <files>` | no output |
| Presence | `grep -n "<pattern>" workflow.md` | matching lines |
| Even fences | `grep -c '```' workflow.md` | even number |
| Dead-link sweep | `grep -rn "dynamic-workflows-claude-code" .` | no matches after Step 4 |
| Scope | `git status --porcelain` | only in-scope files |

Markdown-only repo: no typecheck/test/lint.

## Scope

**In scope**: `workflow.md` (rewrite from the draft), delete
`dynamic-workflows-claude-code.md`, `AGENTS.md`, `CLAUDE.md`, `README.md`,
`plans/README.md` (status row).

**Out of scope**: every other doc; do not touch `plans/drafts/` (that is source
material, not shipped docs).

## Steps

### Step 1: Read the draft and its required change

Open `plans/drafts/workflow-merged.draft.md`. Read the HTML-comment header: it
mandates replacing the draft's bottom "Stack Options" table with a short pointer
to `stack-picker.md` + `skills.md` (the draft already contains the corrected
"## Choosing tools and loading their docs" pointer section — keep that, it is the
fix). Strip the HTML comment.

### Step 2: Write the new `workflow.md`

Replace `workflow.md`'s contents with the draft body (comment removed). Confirm
it contains: the "Claude Code Orchestration Primer", a "## 0. Lock The Stack"
step, phases 1–7 with orchestration notes, the "Choosing tools" pointer (NOT a
duplicated options table), and the two Claude Code source links in Official
References.

**Verify**:
- `grep -c "Claude Code Orchestration Primer" workflow.md` → 1
- `grep -c "ultracode" workflow.md` → ≥ 1
- `grep -c "hairyf/skills\|mcollina/skills\|sentry-for-ai" workflow.md` → 0 (no unverified slug table survived)
- `grep -c '```' workflow.md` even

### Step 3: Handle the stack-picker forward link

`workflow.md` references `stack-picker.md` (created by plan 005). If plan 005 has
already run, the link resolves. If not, that is fine — it is a forward reference;
leave it. Do NOT create `stack-picker.md` here (out of scope).

### Step 4: Delete the blog dump and fix references

- Delete `dynamic-workflows-claude-code.md`.
- `AGENTS.md`: remove the `docs/dynamic-workflows-claude-code.md` line.
- `CLAUDE.md`: remove the `docs/dynamic-workflows-claude-code.md` line from the
  Primary instructions list.
- `README.md`: remove the `dynamic-workflows-claude-code.md` bullet from `##
  Files`; update the `workflow.md` bullet to note it now includes Claude Code
  orchestration guidance.

**Verify**: `grep -rn "dynamic-workflows-claude-code" .` → no matches anywhere
(except inside `plans/` history if any — scope the grep to the repo root docs:
`grep -rn "dynamic-workflows-claude-code" *.md`).

### Step 5: Update the plan index

Set plan 004 row in `plans/README.md` to DONE.

## Done criteria

- [ ] `workflow.md` contains the Orchestration Primer + "## 0. Lock The Stack" + phases 1–7
- [ ] `dynamic-workflows-claude-code.md` deleted (`ls dynamic-workflows-claude-code.md` → not found)
- [ ] `grep -rn "dynamic-workflows-claude-code" *.md` → no matches
- [ ] No unverified skill-slug options table in `workflow.md` (grep in Step 2 → 0)
- [ ] `grep -c '```' workflow.md` even
- [ ] `git status --porcelain` lists only in-scope files
- [ ] `plans/README.md` row 004 = DONE

## STOP conditions

- `dynamic-workflows-claude-code.md`'s content differs materially from a Claude
  Code dynamic-workflows announcement (someone repurposed the file) — STOP.
- The draft file is missing — STOP and report (do not re-derive the merge from
  scratch without the verified sources).

## Maintenance notes

- The Orchestration Primer names Claude Code features (ultracode, dynamic
  workflows, hooks, checkpoints, background tasks) that evolve; re-check against
  the changelog on major updates. Keep feature claims tied to what the changelog
  confirms — do not add speculative features.
- One options source of truth: `skills.md` registry. If you ever feel tempted to
  paste an options table into `workflow.md` again, don't — link the registry.
