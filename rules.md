# Agent Rules

Use this file as the highest-priority project instruction after the user's prompt.

## Mission

Ship production-ready mobile (Expo) and web (Next.js) products fast. The user brings the app idea. Your job is to turn it into a working product on the platforms in scope with the default stack, minimal debate, and verified builds.

## Defaults

- Mobile: Expo, Expo Router, TypeScript.
- Web: Next.js App Router, TypeScript; TanStack Start swap per web.md.
- UI: NativeWind plus React Native Reusables (mobile); Tailwind CSS plus shadcn/ui (web).
- Repo: monorepo with `apps/mobile` and/or `apps/web`, `apps/api`, and optional `packages/*`.
- API: standalone backend service; default Fastify TypeScript, pluggable per backend.md.
- Auth: Clerk by default; pluggable per auth.md.
- Data: Postgres with Drizzle migrations.
- Cache, queues, rate limits: Redis on the user's Coolify/VDS.
- Hosting: Coolify/VDS (Hetzner) for the web app, API, Postgres, Redis, and optional workers. Vercel only as an explicit web swap.
- Metrics: Sentry for crashes, PostHog for product analytics.
- Payments: decision tree in `payments.md`.
- AI features: optional backend-only provider proxy in `ai-features.md`.
- App Store review: checklist in `app-store.md`.
- ASO and screenshots: workflow in `aso.md`.
- Skills: routing and install rules in `skills.md`.

Do not re-litigate these defaults unless the user explicitly overrides one.

## Hard Boundaries

- Never put server secrets in the Expo client or the web client bundle (`NEXT_PUBLIC_*`, client components).
- Never connect the Expo app or web client code directly to Postgres, Redis, Stripe secret keys, OpenRouter keys, or provider API keys.
- Clients call `apps/api`; API talks to infrastructure and external services. Web-only exception: Next.js server-side code may be the backend per web.md, under the same contract.
- Never trust user IDs, prices, credits, limits, entitlements, roles, or ownership values from any client.
- Verify auth server-side on the API before protected reads/writes (default Clerk; see auth.md).
- Use Drizzle migrations for schema changes. Do not hand-edit production schema.
- Add payment and AI modules only when the app actually needs them.

## Work Style

- Build vertical slices, not broad empty scaffolds.
- One working path beats ten half-built screens.
- Prefer simple code and explicit types over clever abstractions.
- Keep files small enough for agents to edit safely.
- Use platform-native behavior where it is cheaper than custom UI.
- Use proven libraries for auth, payments, routing, storage, crash reporting, analytics, and native modules.
- Do not create landing pages inside the mobile app unless the product needs them.

## Decision Rules

- If the user has not specified the stack, run stack-picker.md and take its defaults.
- If platforms are unspecified, ask; if unanswered, infer from the product (store-distributed → mobile, SEO/dashboard/SaaS → web) and state the assumption.
- If the product is web-only and small, Next.js fullstack (route handlers as backend) is allowed per web.md; keep a standalone API when mobile is or will be in scope.
- If the user has not specified backend details, use Fastify, Drizzle, Postgres, Redis. If they request another stack, follow backend.md and its swap contract.
- If the user has not specified auth, use Clerk. If they request another provider, follow auth.md and its swap contract.
- If the product has digital in-app subscriptions or digital content consumed in-app, use RevenueCat/IAP first.
- If the product sells physical goods, appointments, offline services, deposits, or B2B SaaS accessed outside the app, use Stripe through the API.
- If the product has AI features, use a backend proxy with provider keys in server env only.
- If a native module is needed, switch to Expo development builds.
- If a choice affects speed and quality, choose the fastest option that does not create a security, store review, or migration trap.
- If a specialized skill exists for the subsystem, use it before implementation.
- If the skill is missing and listed in `skills.md`, install it before implementation.

## Documentation Discipline

- Read `stack.md` before choosing packages.
- Read `workflow.md` before implementation.
- Read `web.md` before Next.js/TanStack Start, web UI, SEO, or web deployment work.
- Read `backend.md` before API, DB, auth, Redis, Docker, or Coolify work, and pick the matching backend/<stack>.md playbook.
- Read `auth.md` before auth provider, login, session, token, or webhook work.
- Read `security.md` before auth, payments, uploads, or AI work, any feature touching PII or money, and the pre-ship security review.
- Read `payments.md` before payment work.
- Read `ai-features.md` before provider/model work.
- Read `observability.md` before crash/analytics work.
- Read `data.md` before ORM, DB-host, on-device/offline, cache, or background-job choices.
- Read `app-store.md` before release, review, metadata, screenshots, login/demo account, privacy, or compliance work.
- Read `aso.md` before title, subtitle, keywords, description, icon, screenshots, previews, localization, or product page work.
- Read `skills.md` before UI, Expo, React, Postgres, RevenueCat, Clerk, deployment, or ASO work.
- Read `ship-checklist.md` before calling the task done.
- For exact install commands, verify current official docs for the touched integration.

## Verification Gates

Before claiming done:

- Typecheck passes.
- Every in-scope app starts locally (mobile, web, API).
- Protected API routes reject unauthenticated requests.
- Web public pages have SEO metadata if web is in scope (see web.md).
- At least one authenticated happy path works end to end if auth is in scope.
- Migrations are generated and documented if schema changed.
- Env vars are listed in `.env.example`, with no real secrets.
- Sentry and PostHog (or the chosen observability.md providers) are wired if the app has reached a testable MVP.
- EAS build path is documented once native dependencies are present.
- App Store review checklist is complete before store submission.
- Screenshots and metadata match the submitted build.
- ASO assets are generated or explicitly deferred.

## Official References

- Expo docs: https://docs.expo.dev/
- Next.js docs: https://nextjs.org/docs
- shadcn/ui: https://ui.shadcn.com/docs
- Expo Router: https://docs.expo.dev/router/introduction/
- EAS Build: https://docs.expo.dev/build/introduction/
- EAS Submit: https://docs.expo.dev/submit/introduction/
- NativeWind: https://www.nativewind.dev/docs/getting-started/installation
- React Native Reusables: https://rnr-docs.vercel.app/
- Clerk Expo: https://clerk.com/docs/quickstarts/expo
- Fastify: https://fastify.dev/docs/latest/
- Drizzle: https://orm.drizzle.team/docs/overview
- Coolify: https://coolify.io/docs
- Stripe React Native: https://docs.stripe.com/sdks/react-native
- RevenueCat React Native: https://www.revenuecat.com/docs/getting-started/installation/reactnative
- RevenueCat MCP: https://www.revenuecat.com/docs/tools/mcp
- OpenRouter: https://openrouter.ai/docs/quickstart
- Sentry Expo: https://docs.expo.dev/guides/using-sentry/
- PostHog React Native: https://posthog.com/docs/libraries/react-native
- Apple App Review Guidelines: https://developer.apple.com/app-store/review/guidelines/
