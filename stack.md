# Default Stack

Use this stack unless the user explicitly overrides it.

## Product Shape

- Expo mobile app plus Fastify API.
- Monorepo for speed and shared types.
- Coolify/VDS owns deployable backend infrastructure.
- External services are wrappers around the core product, not the architecture.

## Stack Matrix

| Area | Default | Why |
| --- | --- | --- |
| Mobile | Expo + Expo Router + TypeScript | Fast native iteration, file-based routes, EAS path. |
| Styling | NativeWind | Tailwind-speed styling for React Native. |
| UI kit | React Native Reusables | shadcn-like primitives without inventing every control. |
| Auth | Clerk | Fast Expo auth and polished session handling. |
| API | Fastify standalone | Low boilerplate, fast endpoints, good plugin model. |
| DB | Postgres | Default durable data store on the user's VDS/Coolify. |
| ORM | Drizzle | Typed schema, migration control, less runtime magic. |
| Cache | Redis | Rate limits, queues, OTP/session helpers, AI quotas. |
| Deploy | Coolify | User-owned VDS deployment target. |
| Crash | Sentry | Mobile and API error visibility. |
| Analytics | PostHog | Funnels, events, retention, feature usage. |
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
