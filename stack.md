# Default Stack

Use this stack unless the user explicitly overrides it. For the interactive front
door — a fast-path default plus 7 questions to pick swaps — start with
`stack-picker.md`.

## Product Shape

- Expo mobile app and/or Next.js web app plus a backend API (default Fastify;
  swappable — see backend.md). Web-only products may run Next.js fullstack —
  see the product-shape rules in `web.md`.
- Monorepo for speed and shared types.
- Coolify/VDS (Hetzner) owns deployable infrastructure: API, Postgres, Redis,
  and the web frontend. Vercel is the web-deploy swap (`web.md`).
- External services are wrappers around the core product, not the architecture.

## Stack Matrix

These are the defaults — swap any row via `stack-picker.md`. For client-library
choices (state, data-fetching, forms, lists, animation, native capability), see
`mobile-libs.md`.

| Area | Default | Why |
| --- | --- | --- |
| Mobile | Expo + Expo Router + TypeScript | Fast native iteration, file-based routes, EAS path. |
| Web | Next.js (App Router) + TypeScript | RSC/ISR for SEO, largest ecosystem; TanStack Start swap via web.md. |
| Styling | NativeWind (mobile) / Tailwind CSS (web) | Same utility vocabulary on both platforms. |
| UI kit | React Native Reusables (mobile) / shadcn/ui (web) | shadcn-like primitives without inventing every control. |
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
- Put web-only code in `apps/web`.
- Put server-only code in `apps/api`.
- Put shared runtime-safe types in `packages/types`.
- Do not import server code into mobile or into web client components.
- Do not import client code into API.
- Shared packages must not contain secrets or server clients.

## Environment Rules

Mobile public vars:

- Use Expo public env naming only for values safe to ship in the app bundle.
- Clerk publishable key is public.
- API base URL can be public.
- Never expose secret keys, DB URLs, Redis URLs, webhook secrets, provider keys, or Stripe secret keys.

Web public vars:

- Use `NEXT_PUBLIC_` naming only for browser-safe values (API base URL,
  publishable keys, analytics keys). Everything else is server-only env —
  never imported into client components. See `web.md`.

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
- Next.js docs: https://nextjs.org/docs
- Tailwind CSS: https://tailwindcss.com/docs
- shadcn/ui: https://ui.shadcn.com/docs
- NativeWind: https://www.nativewind.dev/docs/getting-started/installation
- React Native Reusables: https://rnr-docs.vercel.app/
- Clerk Expo: https://clerk.com/docs/quickstarts/expo
- Fastify docs: https://fastify.dev/docs/latest/
- Drizzle docs: https://orm.drizzle.team/docs/overview
- Coolify docs: https://coolify.io/docs
