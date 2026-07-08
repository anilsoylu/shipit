# Ship Docs Pack

Reusable agent instructions for building mobile (Expo) and web (Next.js /
TanStack Start) products fast.

## Install Into A New Project

```sh
git clone --depth 1 https://github.com/anilsoylu/shipit.git docs
rm -rf docs/.git
mv docs/AGENTS.md AGENTS.md
mv docs/CLAUDE.md CLAUDE.md
```

Then start the agent with the app idea and tell it to follow `docs/rules.md`.
Recommended starting point after install: run `docs/stack-picker.md` to lock the
stack in seconds.

## Files

- `stack-picker.md`: interactive front door — fast-path defaults + 7 questions (platforms first) to pick swaps.
- `rules.md`: non-negotiable agent rules and default decisions.
- `workflow.md`: execution order from app brief to release, plus Claude Code orchestration guidance.
- `stack.md`: default mobile, web, API, DB, auth, metrics, payment, and AI stack.
- `web.md`: web frontend playbook — Next.js default, TanStack Start swap, SEO essentials, web deploy.
- `backend.md`: backend swap contract, shared DB/Redis/Coolify/security guidance, and playbook router.
- `backend/<stack>.md`: reference playbooks — fastify (default), hono, go, rust, python.
- `data.md`: data swap contract + ORM/DB-host/on-device/cache/jobs option menu.
- `auth.md`: auth provider selection, swap contract, and per-provider playbooks (Clerk default, better-auth, Supabase, custom JWT).
- `payments.md`: payments swap contract, RevenueCat/Stripe playbooks, situational providers.
- `observability.md`: crash + analytics swap contract, Sentry/PostHog playbooks, alternates.
- `ai-features.md`: optional backend-only AI provider playbook.
- `app-store.md`: App Store review and packaging checklist.
- `aso.md`: ASO, metadata, screenshot, and product page workflow.
- `mobile-libs.md`: curated Expo/RN client library menu (state, data, forms, lists, animation, native).
- `skills.md`: required and optional agent skill routing.
- `ship-checklist.md`: pre-release verification checklist.
- `AGENTS.md`: Codex bridge file.
- `CLAUDE.md`: Claude bridge file.

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
