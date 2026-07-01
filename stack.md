# Default Stack

Use this stack unless the user explicitly overrides it. For the interactive front
door — a fast-path default plus 6 questions to pick swaps — start with
`stack-picker.md`.

## Product Shape

- Expo mobile app plus a backend API (default Fastify; swappable — see backend.md).
- Monorepo for speed and shared types.
- Coolify/VDS owns deployable backend infrastructure.
- External services are wrappers around the core product, not the architecture.

## Stack Matrix

These are the defaults — swap any row via `stack-picker.md`. For client-library
choices (state, data-fetching, forms, lists, animation, native capability), see
`mobile-libs.md`.

| Area | Default | Why |
| --- | --- | --- |
| Mobile | Expo + Expo Router + TypeScript | Fast native iteration, file-based routes, EAS path. |
| Styling | NativeWind | Tailwind-speed styling for React Native. |
| UI kit | React Native Reusables | shadcn-like primitives without inventing every control. |
| Auth | Clerk (default) | Fast Expo auth; swappable — see auth.md for better-auth, Supabase, custom JWT. |
| API | Fastify standalone (default) | Low boilerplate, fast; swappable to Hono/Go/Rust/Python — see backend.md. |
| DB | Postgres (default) | Default durable data store on the user's VDS/Coolify; host/swap via data.md. |
| ORM | Drizzle (default) | Typed schema, migration control; swap to Prisma/Kysely via data.md. |
| Cache | Redis | Rate limits, queues, OTP/session helpers, AI quotas. |
| Deploy | Coolify | User-owned VDS deployment target. |
| Crash | Sentry (default) | Mobile and API error visibility; swap via observability.md. |
| Analytics | PostHog (default) | Funnels, events, retention, feature usage; swap via observability.md. |
| Payments | Stripe or RevenueCat | Depends on product and store rules. |
| AI | OpenRouter/provider proxy | Optional, server-side only. |

## Package Rules

- Put mobile-only code in `apps/mobile`.
- Put server-only code in `apps/api`.
- Put shared runtime-safe types in `packages/types`.
- Do not import server code into mobile.
- Do not import mobile code into API.
- Shared packages must not contain secrets or server clients.

## Environment Rules

Mobile public vars:

- Use Expo public env naming only for values safe to ship in the app bundle.
- Clerk publishable key is public.
- API base URL can be public.
- Never expose secret keys, DB URLs, Redis URLs, webhook secrets, provider keys, or Stripe secret keys.

API secret vars:

- `DATABASE_URL`
- `REDIS_URL`
- `CLERK_SECRET_KEY`
- `CLERK_WEBHOOK_SECRET` when needed
- Auth env vars are provider-specific; see auth.md for the chosen provider's public vs secret keys.
- `STRIPE_SECRET_KEY` when Stripe is in scope
- `STRIPE_WEBHOOK_SECRET` when Stripe is in scope
- `OPENROUTER_API_KEY` or provider-specific keys when AI is in scope
- `SENTRY_DSN`
- `POSTHOG_KEY`

## Integration Triggers

Add integrations only when the product needs them:

- Add Stripe for physical goods, services, booking deposits, paid web/SaaS, or marketplace flows.
- Add RevenueCat for mobile digital subscriptions and in-app digital entitlements.
- Add OpenRouter or other model providers only for product-facing AI features.
- Add Redis when rate limiting, jobs, queues, token buckets, or AI credits are needed.
- Add storage only when user-generated files are part of the core loop.

## Official References

- Expo docs: https://docs.expo.dev/
- NativeWind: https://www.nativewind.dev/docs/getting-started/installation
- React Native Reusables: https://rnr-docs.vercel.app/
- Clerk Expo: https://clerk.com/docs/quickstarts/expo
- Fastify docs: https://fastify.dev/docs/latest/
- Drizzle docs: https://orm.drizzle.team/docs/overview
- Coolify docs: https://coolify.io/docs
