# Stack Picker

Fastest way to lock your stack. Answer the questions, take the defaults unless a
line below tells you to swap. Every choice points to its skill and `llms.txt` in
`skills.md` (the LLM Docs Registry) so the agent loads authoritative docs before
writing code.

## Fast path (recommended)

Accept every default and you get the **Expo Ship Stack** (mobile):

> Expo + Expo Router + TypeScript, NativeWind + React Native Reusables, Clerk
> auth, Fastify TypeScript API on Coolify/VDS, Postgres + Drizzle on your
> Coolify/VDS (Neon swap), Redis on the same box, RevenueCat (in-app subs) /
> Stripe (web), Sentry + PostHog, OpenRouter + Vercel AI SDK for any AI.

For web, the **Web Ship Stack** shares everything below the frontend:

> Next.js (App Router) + TypeScript, Tailwind CSS + shadcn/ui, deployed on
> your Coolify/VDS next to the API/Postgres/Redis (Vercel swap); same
> auth/backend/data/payments/observability/AI defaults as above. TanStack
> Start is the supported framework swap (`web.md`).

If that fits, tell the agent "use the default Expo Ship Stack" and/or "use the
default Web Ship Stack" and skip to Scaffold in `workflow.md`. Otherwise answer
the 7 questions.

## The 7 questions

**0. Which platforms?**
- Mobile (iOS/Android) → Expo. `mobile-libs.md`, `app-store.md`, `aso.md` apply.
- Web → **Next.js App Router** (default) or **TanStack Start** (client-heavy
  apps, type-safe routing). See `web.md`; SEO section replaces `aso.md`.
- Both → Expo + Next.js sharing one `apps/api` and `packages/types`.
- Web-only and small → Next.js fullstack (route handlers as backend) is
  allowed; see the product-shape rules in `web.md`.

**1. What are you charging for?**
- Nothing yet / free → skip payments.
- In-app digital subscription or unlock (iOS/Android) → **RevenueCat** (default).
- Web checkout, SaaS, physical goods, marketplace → **Stripe**.
- Both → RevenueCat in-app, Stripe on web.
  Store rules: Apple/Google require their IAP for in-app digital goods — Stripe
  is for web/physical only. See `payments.md`.

**2. Who owns identity?**
- Just want it to work, hosted → **Clerk** (default).
- Self-host, own the user table, TS backend → **better-auth**.
- Already on Supabase / want Postgres-native → **Supabase Auth**.
- Hard requirement forces it → custom JWT (discouraged). See `auth.md`.

**3. What language is the backend?**
- TypeScript, richest ecosystem → **Fastify** (default).
- TS but edge/portable runtimes → **Hono**.
- Go / Rust / Python(FastAPI) → matching `backend-<stack>.md`. See `backend.md`.

**4. Where does data live?**
- ORM → **Drizzle** (default) or Prisma.
- Postgres host → **your Coolify/VDS Postgres** (default, Hetzner); **Neon**
  (serverless) or **Supabase** (BaaS) as swaps.
- On-device / offline? None (server-only, default) → skip; local cache →
  **Expo SQLite**; offline-first sync → **WatermelonDB** or **PowerSync**.
- Need cache/jobs? → **Redis** (self-host) or **Upstash** (serverless);
  background jobs → **Trigger.dev** or **QStash**. See `data.md`.

**5. What do you need to see in production?**
- Crashes/errors → **Sentry** (default). Analytics → **PostHog** (default).
- In Google ecosystem → Firebase (Crashlytics + Analytics).
- Deep behavioural analytics → Amplitude / Mixpanel.
- Privacy-first / open-source (runs in Expo Go) → Aptabase. See `observability.md`.

**6. Any AI features?**
- No → skip.
- Yes → **OpenRouter** gateway + **Vercel AI SDK** + Claude model family
  (default). One key, many models, easy fallback. Direct provider only if you
  need it (Anthropic/OpenAI). Media → Replicate/fal/ElevenLabs. See
  `ai-features.md`. Keys stay server-side; the app calls your API, never a
  provider directly.

## Your stack (fill from the answers)

| Area | Choice | Swap |
| --- | --- | --- |
| Mobile | Expo + Expo Router + TS | bare RN only if forced |
| Web | Next.js App Router + TS | TanStack Start (`web.md`) |
| Styling/UI (mobile) | NativeWind + React Native Reusables | Tamagui, Gluestack |
| Styling/UI (web) | Tailwind CSS + shadcn/ui | Tamagui universal (`web.md`) |
| Web deploy | Coolify/VDS | Vercel (`web.md`) |
| Auth | Clerk | better-auth, Supabase, custom JWT (`auth.md`) |
| Backend | Fastify TS | Hono, Go, Rust, Python (`backend.md`) |
| ORM | Drizzle | Prisma (`data.md`) |
| DB host | Coolify/VDS Postgres | Neon, Supabase, Turso (`data.md`) |
| On-device | none / Expo SQLite | WatermelonDB, PowerSync (`data.md`) |
| Cache/jobs | Redis / Upstash | Trigger.dev, QStash (`data.md`) |
| Payments | RevenueCat / Stripe | Adapty, Superwall, Paddle (`payments.md`) |
| Crash | Sentry | Firebase Crashlytics (`observability.md`) |
| Analytics | PostHog | Amplitude, Mixpanel, Aptabase (`observability.md`) |
| AI | OpenRouter + Vercel AI SDK | direct provider (`ai-features.md`) |
| Client libs | TanStack Query, Zustand, RHF+Zod, MMKV, FlashList v2 | SWR, tRPC, Jotai (`mobile-libs.md`) |

## Load the docs before you build

For every non-default choice, the agent must load that option's skill and
`llms.txt` first — do not code from memory. The pointers live in the
**LLM Docs Registry** in `skills.md`. Quick anchors (slugs API-resolved and
`curl`-verified 2026-07-01):

| Choice | Skill (`npx skills add ...`) | llms.txt |
| --- | --- | --- |
| Expo | `expo/skills` | https://docs.expo.dev/llms-full.txt |
| NativeWind | `expo/skills` (expo-tailwind-setup) | https://www.nativewind.dev/llms.txt |
| Next.js | — (use installed `nextjs-*` skills) | https://nextjs.org/docs/llms.txt |
| TanStack Start | — | https://tanstack.com/start/latest/llms.txt |
| shadcn/ui | — | https://ui.shadcn.com/llms.txt |
| Tailwind CSS | — | check https://tailwindcss.com/docs (no llms.txt) |
| Vercel | — | https://vercel.com/docs/llms.txt |
| Coolify | — | https://coolify.io/docs/llms-full.txt |
| Clerk | `clerk/skills` | https://clerk.com/llms.txt |
| better-auth | `better-auth/skills` | https://www.better-auth.com/llms.txt |
| Supabase | `supabase/agent-skills` | https://supabase.com/llms.txt |
| Drizzle | — (no vendor pack; use llms.txt) | https://orm.drizzle.team/llms.txt |
| Prisma | `prisma/skills` | https://www.prisma.io/docs/llms.txt |
| Neon | `neondatabase/agent-skills` | https://neon.com/llms.txt |
| Redis | `redis/agent-skills` | https://redis.io/llms.txt |
| Upstash | `upstash/skills` | https://upstash.com/docs/llms.txt |
| Trigger.dev | `triggerdotdev/skills` | https://trigger.dev/docs/llms.txt |
| RevenueCat | `revenuecat/ai-toolkit` | https://www.revenuecat.com/docs/llms.txt |
| Stripe | `stripe/ai` | https://docs.stripe.com/llms.txt |
| Sentry | `getsentry/skills` | https://docs.sentry.io/llms.txt |
| PostHog | `posthog/skills` | https://posthog.com/llms.txt |
| OpenRouter | `openrouterteam/agent-skills` | https://openrouter.ai/docs/llms.txt |
| Vercel AI SDK | `vercel/ai` | https://ai-sdk.dev/llms.txt |
| Resend (email) | `resend/resend-skills` | https://resend.com/docs/llms.txt |
| Cloudflare R2 | `cloudflare/skills` | https://developers.cloudflare.com/r2/llms.txt |

Standing rule: for any service not in this table, look for `<docs-domain>/llms.txt`
and its skill on https://www.skills.sh/ before implementing (see the full
registry in `skills.md`). Verify each URL/slug once before relying on it — they
drift.

## After picking

1. Confirm the filled Stack Matrix back to the user.
2. Install the skills for every chosen option.
3. Proceed to Scaffold in `workflow.md`.
