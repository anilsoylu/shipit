# Expo Ship Workflow

Use this workflow after the user has chosen the app idea. Do not spend time discovering apps unless explicitly asked.

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

If details are missing, choose the smallest credible MVP and state assumptions.

## 2. Scope Cut

Define a 1-day MVP:

- 3 to 5 screens maximum.
- 1 primary user path.
- 1 auth path if login is required.
- 1 backend resource if data persistence is required.
- 1 monetization path only if it is core to validation.
- No settings maze, no admin panel, no generic dashboard unless needed.

Write the slice before coding: routes, API endpoints, data model, env vars, and acceptance checks.

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
- Add NativeWind and React Native Reusables.
- Add `.env.example` with only public Expo vars prefixed as required.
- Add app scheme for auth callbacks and deep links.

API:

- Create Fastify TypeScript service.
- Add health route.
- Add Clerk auth middleware.
- Add Drizzle, Postgres connection, migrations folder.
- Add Redis client only when rate limits, queues, sessions, or caching need it.
- Add Dockerfile and Coolify-ready env list.

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

## 7. Done Definition

A task is done only when:

- The main user path works on device or simulator.
- Auth, API, DB, and payments/AI are wired only where needed.
- Secrets are not exposed in mobile code.
- Env vars are documented.
- Typecheck passes.
- Known gaps are listed plainly.
- Next ship step is obvious.

## Official References

- Expo create project: https://docs.expo.dev/get-started/create-a-project/
- Expo Router: https://docs.expo.dev/router/introduction/
- Development builds: https://docs.expo.dev/develop/development-builds/introduction/
- EAS Build: https://docs.expo.dev/build/introduction/
- EAS Submit: https://docs.expo.dev/submit/introduction/
- EAS Update: https://docs.expo.dev/eas-update/introduction/
