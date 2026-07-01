# Expo Ship Workflow

Use this after the user has chosen the app idea. Do not spend time discovering apps unless explicitly asked.

This doc has two layers:
- WHAT to build and in what order (the ship workflow, phases 1-7).
- HOW to drive it with Claude Code (the orchestration notes in each phase).

## Claude Code Orchestration Primer

Read once, then apply per phase. Every capability below is a real Claude Code feature (confirmed against the changelog feature flags and the dynamic-workflows announcement).

- Plan mode: draft the slice, routes, schema, and env before writing code. Never scaffold from a vague prompt. Plan mode is for verification steps too, not just building.
- Skills: load the exact skill for the tool you touch BEFORE coding it (see `skills.md` and the per-domain options tables). Skills carry current, opinionated setup steps so the agent stops guessing SDK versions.
- Subagents: fan out independent work (research a provider, scaffold `api` while `mobile` scaffolds, write states for 3 screens). One task per subagent, focused. Set a cheaper subagent model for bulk grunt work when appropriate.
- Dynamic workflows / ultracode: for big, parallel, checked-twice jobs, either ask Claude to "create a workflow" or switch on ultracode (effort menu; it sets effort to xhigh and lets Claude decide when to spin up a workflow). Workflows run tens to hundreds of parallel subagents, verify findings with adversarial agents before folding them in, and save progress so an interrupted run resumes instead of restarting. They cost meaningfully more tokens; the first run asks you to confirm. Turn on auto mode when using them.
- Background tasks: keep dev server, Metro/Expo, API, and watchers running detached across turns instead of blocking on them.
- Checkpoints: rely on file checkpointing to roll back a bad slice cleanly instead of hand-reverting.
- Hooks: wire typecheck/lint/format to run automatically on stop or edit so the done-definition stays enforced without you asking.
- MCP: prefer provider MCP servers when they exist and are in scope (e.g. RevenueCat, Stripe) over hand-rolled API calls.

When to escalate: a single agent handles one slice. Reach for a dynamic workflow only for codebase-wide passes (security/perf audit, dead-code hunt, a migration touching many files, or a plan you want stress-tested from every angle). Building one MVP slice does NOT need a workflow.

## 0. Lock The Stack

Before scaffolding, run `stack-picker.md`: accept the default Expo Ship Stack or answer its 6 questions to choose swaps. Confirm the filled Stack Matrix and install the chosen skills. This makes the stack decision once, up front, instead of mid-build churn.

## 1. Intake

Extract or ask for only the missing high-impact details:

- Product promise: what the app does in one sentence.
- Target user and top job-to-be-done.
- Core loop: open app, do what, get what result.
- Monetization: free, subscription, one-time, credits, bookings, services, marketplace, or none.
- Required platforms: iOS, Android, web, or internal test only.
- Must-have external services: auth, payments, AI provider, maps, push, email, storage, admin.
- Reference apps or screenshots if available.
- Store packaging target: first launch, update, ASO refresh, or internal test.

If details are missing, choose the smallest credible MVP and state assumptions. Do this in plan mode.

## 2. Scope Cut

Define a 1-day MVP:

- 3 to 5 screens maximum.
- 1 primary user path.
- 1 auth path if login is required.
- 1 backend resource if data persistence is required.
- 1 monetization path only if it is core to validation.
- No settings maze, no admin panel, no generic dashboard unless needed.

Write the slice before coding: routes, API endpoints, data model, env vars, and acceptance checks. Keep this plan; it is the success criteria you loop against.

Orchestration:
- Do this in plan mode. Get the plan approved before touching files.
- If the stack is undecided, fan out a research subagent per undecided domain (auth, db, payments) to pick from the registry in `skills.md`, then converge.

## 3. Scaffold

Default structure:

```txt
apps/
  mobile/
  api/
packages/
  config/
  types/
docs/
```

Mobile:

- Create Expo app with TypeScript and Expo Router.
- Add NativeWind and React Native Reusables (default UI stack).
- Add `.env.example` with only public Expo vars prefixed as required.
- Add app scheme for auth callbacks and deep links.

API:

- Create the backend service from the chosen playbook in `backend.md` (default Fastify TypeScript; Hono/Go/Rust/Python are valid swaps).
- Add health route.
- Add auth middleware per `auth.md` (default Clerk).
- Add Drizzle, Postgres connection, migrations folder.
- Add Redis client only when rate limits, queues, sessions, or caching need it.
- Add Dockerfile and Coolify-ready env list.

Orchestration:
- Load the relevant setup skills first (e.g. `expo/skills` expo-dev-client + expo-deployment, your chosen auth/db skill).
- Run `mobile` and `api` scaffolds as parallel subagents; they are independent.
- Start Expo and the API as background tasks and keep them alive across the build.

Follow the default stack unless the user overrides it. `stack-picker.md` and the LLM Docs Registry in `skills.md` list every verified per-domain option with its agent skill and llms.txt so a swap stays smooth.

## 4. Build The Vertical Slice

Build in this order:

1. App navigation shell with final route names.
2. Real UI for the main screen, not placeholder cards.
3. Auth gate if needed.
4. API endpoint for the main action.
5. Drizzle schema and migration if data persists.
6. Mobile API client and typed request/response.
7. Loading, empty, error, and success states.
8. PostHog events for the core loop.
9. Sentry capture around app/API failures.
10. App Store packaging notes if the slice affects screenshots, onboarding, pricing, or review.

Stop broadening scope until the first slice works end to end.

Orchestration:
- Keep the main agent on the critical path (nav -> main UI -> endpoint -> wire). Fan out subagents for parallelizable leaf work: per-screen loading/empty/error states, analytics event wiring, test data.
- Point the analytics/observability skill at steps 8-9 so instrumentation matches current SDKs.
- Let hooks run typecheck on each stop; fix failures before moving on (autonomous, no hand-holding).
- Use a checkpoint before risky refactors so you can roll back the slice, not the repo.

## 5. Optional Modules

Payments:

- Read `payments.md`.
- Use RevenueCat MCP if RevenueCat setup is in scope and MCP access exists.
- Use Stripe only through API-created intents, sessions, customers, and webhooks.
- Use RevenueCat/IAP when the mobile app sells digital subscriptions or digital content consumed in-app.

AI features:

- Read `ai-features.md`.
- Add provider calls only in `apps/api`.
- Add Redis-based rate limits and user quotas before exposing expensive generation.

Push, email, maps, storage:

- Add only after the core loop needs them.
- Keep credentials server-side unless the provider requires public client keys.

ASO and App Store packaging:

- Read `app-store.md` and `aso.md`.
- Prepare metadata before screenshots.
- Capture screenshots from the current build.
- Use app-store screenshot tooling only after UI is stable.
- Use product page optimization after the app has traffic.

Orchestration:
- Load the module's skill before wiring it (see the registry in `skills.md`).
- Prefer the provider MCP over hand-rolled calls when in scope.

## 6. Release Path

For local MVP:

- Run typecheck.
- Run mobile start.
- Run API start.
- Smoke test the core route on iOS or Android.

For native modules:

- Use Expo development builds.
- Do not depend on Expo Go once native SDKs are required.

For distribution:

- Configure EAS build profiles for development, preview, and production.
- Use internal distribution for tester builds.
- Use EAS Submit for store binaries when ready.
- Use EAS Update for JS-only fixes that comply with store policy.
- Complete App Store review and ASO checklist before submission.

Orchestration:
- Run EAS builds as background tasks; they are long. Come back to the result.
- For a pre-submission hardening pass (auth checks, input validation, secret exposure, unsafe patterns across the whole codebase), trigger a dynamic workflow / ultracode: it audits in parallel and independently verifies each finding so the report is real, not noise. This is the one place a workflow earns its token cost on an MVP.

## 7. Done Definition

A task is done only when:

- The main user path works on device or simulator.
- Auth, API, DB, and payments/AI are wired only where needed.
- Secrets are not exposed in mobile code.
- Env vars are documented.
- Typecheck passes.
- Known gaps are listed plainly.
- Next ship step is obvious.

Verify, do not assume. Diff behavior against the plan from step 2. For high-stakes correctness, let a workflow give independent attempts with adversarial agents trying to break the result before you accept it.

## Choosing tools and loading their docs

Do NOT restate the options here. Pick the stack in `stack-picker.md`, and load
each chosen tool's skill + `llms.txt` from the single LLM Docs Registry in
`skills.md`. That is the one source of truth for per-domain options, so it never
drifts against this doc.

## Official References

- Expo create project: https://docs.expo.dev/get-started/create-a-project/
- Expo Router: https://docs.expo.dev/router/introduction/
- Development builds: https://docs.expo.dev/develop/development-builds/introduction/
- EAS Build: https://docs.expo.dev/build/introduction/
- EAS Submit: https://docs.expo.dev/submit/introduction/
- EAS Update: https://docs.expo.dev/eas-update/introduction/
- Dynamic workflows announcement: https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
- Claude Code changelog (community): https://github.com/marckrenn/claude-code-changelog
