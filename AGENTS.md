# Agent Entry

This repo is meant to be cloned into new projects as `docs/`.

If this file is copied to the project root, read:

- `docs/stack-picker.md`
- `docs/rules.md`
- `docs/stack.md`
- `docs/workflow.md`
- `docs/web.md`
- `docs/backend.md` (links to `backend/fastify.md`, `backend/hono.md`, `backend/go.md`, `backend/rust.md`, `backend/python.md`)
- `docs/data.md`
- `docs/auth.md` (links to `auth/clerk.md`, `auth/betterauth.md`, `auth/supabase.md`, `auth/custom-jwt.md`)
- `docs/security.md` (links to `security/access-control.md`, `security/injection.md`, `security/ssrf.md`, `security/auth-sessions.md`, `security/crypto-transport.md`, `security/web-nextjs.md`, `security/edge-proxy.md`, `security/mobile-expo.md`, `security/webhooks-payments.md`, `security/storage-r2.md`, `security/ai-openrouter.md`, `security/secrets-config.md`, `security/logging-privacy.md`, `security/dependencies.md`, `security/dos-limits.md`, `security/incident-response.md`, `security/per-framework.md`)
- `docs/payments.md`
- `docs/storekit-paywall-debug.md`
- `docs/observability.md`
- `docs/ai-features.md`
- `docs/app-store.md`
- `docs/app-store-submission-runbook.md`
- `docs/aso.md`
- `docs/mobile-libs.md`
- `docs/skills.md`
- `docs/ship-checklist.md`

If this file remains inside `docs/`, read the sibling files with the same names.

Follow `rules.md` first. Use `workflow.md` for execution order. Use `skills.md` before touching specialized subsystems. Use the specialized files only when that subsystem is touched. Mobile-only docs (`app-store*.md`, `aso.md`, `mobile-libs.md`) apply only when mobile is in scope; `web.md` applies only when web is in scope.
