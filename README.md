# Expo Ship Docs Pack

Reusable agent instructions for building Expo apps fast.

## Install Into A New Project

```sh
git clone --depth 1 <this-repo-url> docs
rm -rf docs/.git
cp docs/AGENTS.md AGENTS.md
cp docs/CLAUDE.md CLAUDE.md
```

Then start the agent with the app idea and tell it to follow `docs/rules.md`.

## Files

- `rules.md`: non-negotiable agent rules and default decisions.
- `workflow.md`: execution order from app brief to release.
- `stack.md`: default Expo, API, DB, auth, metrics, payment, and AI stack.
- `backend.md`: Fastify, Clerk, Drizzle, Postgres, Redis, Coolify playbook.
- `payments.md`: Stripe vs RevenueCat decision tree.
- `ai-features.md`: optional backend-only AI provider playbook.
- `app-store.md`: App Store review and packaging checklist.
- `aso.md`: ASO, metadata, screenshot, and product page workflow.
- `skills.md`: required and optional agent skill routing.
- `ship-checklist.md`: pre-release verification checklist.
- `AGENTS.md`: Codex bridge file.
- `CLAUDE.md`: Claude bridge file.
