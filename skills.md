# Agent Skills Routing

Goal: make agents use the right specialized skill before touching important subsystems.

## Core Installed Skills

Use these when available:

- `postgresql-database-engineering`: schema design, indexes, migrations, query performance, backups, production Postgres.
- `nextjs-app-router-patterns`: App Router structure, server/client component boundaries, layouts, route handlers.
- `nextjs-data-fetching`: RSC data fetching, caching, revalidation, server actions.
- `nextjs-performance`: Next.js bundle, rendering, and Core Web Vitals optimization.
- `seo-audit` / `seo-geo`: web SEO passes — metadata, sitemap, structured data, geo/AI-search visibility.
- `web-design-guidelines`: web UI quality guardrails.
- `vercel-composition-patterns`: reusable React component APIs, compound components, context interfaces, avoiding boolean prop sprawl.
- `vercel-react-best-practices`: React performance, waterfalls, bundle size, rerenders, data fetching.
- `frontend-design`: distinctive production UI, non-generic visual direction.
- `ui-ux-pro-max`: mobile UI/UX, accessibility, touch targets, navigation, typography, color, interaction quality.
- `gpt-taste`: high-end marketing/web pages, motion-heavy pages, ASO screenshot landing-style composition.
- `design-taste-frontend`: senior frontend architecture, premium UI guardrails, animation and layout performance.

If a required skill is missing, install it before doing that category of work.

## Expo Skills To Install When Needed

Install with:

```sh
npx skills add https://github.com/expo/skills --skill <skill-name>
```

Or install the whole Expo skill pack:

```sh
npx skills add expo/skills
```

Core Expo set:

- `building-native-ui`: use for Expo Router, native tabs, modals, forms, native UI patterns.
- `expo-tailwind-setup`: use when setting up NativeWind/Tailwind in Expo.
- `native-data-fetching`: use for API calls, caching, auth tokens, error handling, offline behavior.
- `expo-dev-client`: use when native modules or custom dev clients are needed.
- `expo-deployment`: use for EAS builds, App Store, Play Store, preview, and production deployment.
- `expo-cicd-workflows`: use when `.eas/workflows` or CI/CD automation is requested.
- `upgrading-expo`: use for Expo SDK upgrades and dependency conflicts.

Conditional Expo skills:

- `expo-api-routes`: use only if the project uses Expo Router API routes or EAS Hosting. Default backend remains Fastify on Coolify.
- `expo-module`: use only for custom native modules/views.
- `use-dom`: use when a web-only React library must run inside Expo native.
- `expo-ui-swiftui`: use only for SDK-compatible SwiftUI-backed Expo UI work.
- `expo-ui-jetpack-compose`: use only for SDK-compatible Jetpack Compose-backed Expo UI work.
- `eas-update-insights`: use when inspecting EAS Update health, crash rates, launches, or rollout issues.
- `expo-brownfield`: use only for existing native iOS/Android apps adopting Expo.
- `add-app-clip`: use only when adding an iOS App Clip.

Known Expo skill names:

- `building-native-ui`
- `native-data-fetching`
- `upgrading-expo`
- `expo-tailwind-setup`
- `expo-cicd-workflows`
- `expo-deployment`
- `expo-dev-client`
- `expo-api-routes`
- `use-dom`
- `expo-module`
- `expo-ui-swiftui`
- `expo-ui-jetpack-compose`
- `eas-update-insights`
- `expo-brownfield`
- `add-app-clip`

Vercel React Native skill:

```sh
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-native-skills
```

Use it for React Native performance, list rendering, animations, navigation, native modules, and Expo architecture checks.

Vercel JSON render for React Native:

```sh
npx skills add https://github.com/vercel-labs/json-render --skill react-native
```

Use it only when the product needs JSON-defined native mobile UI trees, AI-generated UI specs, dynamic component catalogs, or runtime-rendered native screens. Do not add it to normal apps by default.

## Clerk Skills

Install with:

```sh
npx skills add clerk/skills
```

Or install a focused skill:

```sh
npx skills add https://github.com/clerk/skills --skill <skill-name>
```

Expo-focused defaults:

- `clerk`: router skill, start here when Clerk work is unclear.
- `clerk-setup`: new Clerk setup.
- `clerk-expo`: Clerk in Expo and React Native.
- `clerk-expo-patterns`: SecureStore token persistence, OAuth, callbacks, Expo-specific flows.
- `clerk-backend-api`: backend API exploration.
- `clerk-webhooks`: webhook verification and event handling.
- `clerk-custom-ui`: custom sign-in/up UI.
- `clerk-testing`: auth test flows.
- `clerk-billing`: only if Clerk Billing is explicitly chosen.

Known Clerk skill names:

- `clerk`
- `clerk-setup`
- `clerk-custom-ui`
- `clerk-backend-api`
- `clerk-expo`
- `clerk-expo-patterns`
- `clerk-webhooks`
- `clerk-testing`
- `clerk-billing`
- `clerk-orgs`
- `clerk-react-patterns`
- `clerk-nextjs-patterns`
- `clerk-react-router-patterns`
- `clerk-tanstack-patterns`
- `clerk-vue-patterns`
- `clerk-nuxt-patterns`
- `clerk-astro-patterns`
- `clerk-android`
- `clerk-swift`
- `clerk-chrome-extension-patterns`

## Supabase And Postgres Skills

Default app stack uses the user's Coolify/VDS Postgres, not Supabase as the app backend. Still use Supabase's Postgres skill for database quality.

Install:

```sh
npx skills add https://github.com/supabase/agent-skills --skill supabase-postgres-best-practices
```

Use together:

- `postgresql-database-engineering`: deep production database engineering.
- `supabase-postgres-best-practices`: Postgres performance and query rules from Supabase.

Use these before schema design, indexes, migrations, query optimization, RLS-like authorization modeling, pgBouncer, backups, or production database debugging.

## Redis Skills

Install:

```sh
npx skills add redis/agent-skills
```

Focused install:

```sh
npx skills add https://github.com/redis/agent-skills --skill redis-best-practices
npx skills add https://github.com/redis/agent-skills --skill redis-development
```

Use:

- `redis-best-practices`: caching, rate limits, queues, vector search, semantic caching, performance.
- `redis-development`: Redis implementation help when available.

Use these before adding AI quotas, rate limits, cache invalidation, background jobs, locks, leaderboards, pub/sub, or semantic cache.

## RevenueCat MCP

Use RevenueCat MCP when subscriptions or IAP are in scope and the agent has MCP access.

It can manage:

- Projects.
- Apps.
- Products.
- Offerings and packages.
- Paywalls.
- Analytics and experiments.

Reference: https://www.revenuecat.com/docs/tools/mcp

## RevenueCat Skills

Install RevenueCat skill packs as needed:

```sh
npx skills add revenuecat/ai-toolkit
npx skills add revenuecat/rc-claude-code-plugin
npx skills add revenuecat/revenuecat-skill
```

Use `revenuecat/ai-toolkit` for implementation work:

- `create-revenuecat-project`
- `integrate-revenuecat`
- `revenuecat`
- `revenuecat-paywall`
- `revenuecat-purchase-flow`
- `revenuecat-entitlements-gate`
- `revenuecat-identify-user`
- `revenuecat-customer-center`
- `revenuecat-testing-setup`
- `revenuecat-status`
- `revenuecat-troubleshoot`
- `revenuecat-charts`
- `revenuecat-migrate`

Use `rc-claude-code-plugin` for dashboard/API setup flows:

- `apikey`
- `create-app`
- `create-product`
- `status`

Use RevenueCat skills only when IAP, subscriptions, entitlements, paywalls, or RevenueCat analytics are in scope.

## Stripe Skills

Install the Stripe skill pack as needed:

```sh
npx skills add stripe/ai --skill stripe-best-practices
```

The `stripe/ai` pack ships these skills:

- `stripe-best-practices`
- `upgrade-stripe`
- `stripe-projects`
- `stripe-directory`
- `connect-recommend`

If the Stripe MCP is available in the environment, use it for Stripe API exploration and documentation search (`search_stripe_documentation`, `stripe_api_read`).

Use Stripe skills only when server-driven Stripe work is in scope: checkout, payment intents, subscriptions, Connect, or webhooks (see `payments.md`).

## LLM Docs Registry

Verified: 2026-07-01 (skill slugs API-resolved on skills.sh; llms.txt URLs
`curl`-checked 200). This is the single directory the stack-picker and every
domain doc link into.

When integrating any external service, first look for its `llms.txt` or
`llms-full.txt` (usually at `<docs-domain>/llms.txt`) and its official
agent skill on https://www.skills.sh/. Prefer these over training-data
recall — they reflect the current SDK. If neither exists, use the official
docs linked in the relevant playbook. Slugs and URLs drift — re-verify on a
cadence and refresh the date stamp above.

Skill column = `npx skills add <slug>`. `—` = no authoritative vendor pack; use
the llms.txt.

### Web core (Next.js / TanStack)

Verified 2026-07-07 (llms.txt URLs curl-checked 200; Tailwind has none).

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Next.js | — (use installed `nextjs-*` skills) | https://nextjs.org/docs/llms.txt | Default web framework: App Router, RSC, route handlers, ISR. |
| TanStack Start | — | https://tanstack.com/start/latest/llms.txt | Client-heavy web apps with type-safe routing/loaders (swap). |
| TanStack Router | — | https://tanstack.com/router/latest/llms.txt | Standalone type-safe SPA routing. |
| TanStack Virtual | — | https://tanstack.com/virtual/latest/llms.txt | Web list virtualization (FlashList's web counterpart). |
| Tailwind CSS | — | check https://tailwindcss.com/docs (no llms.txt) | Web utility styling; same vocabulary as NativeWind. |
| shadcn/ui | — | https://ui.shadcn.com/llms.txt | Default web UI primitives (RNR's web counterpart). |
| Coolify | — | https://coolify.io/docs/llms-full.txt | Default deploy for web + API + Postgres + Redis on the user's Hetzner VDS. |
| Vercel | — | https://vercel.com/docs/llms.txt | Web deploy swap: previews per PR, edge/serverless. |
| Motion (framer-motion) | — | https://motion.dev/llms.txt | Web animation (Reanimated's web counterpart). |
| nuqs | — | https://nuqs.dev/llms.txt | Type-safe URL query state for Next.js/React. |

### Mobile core (Expo / RN)

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Expo | `expo/skills` | https://docs.expo.dev/llms-full.txt | Expo SDK, config plugins, EAS, native APIs, Expo Router. |
| NativeWind | `expo/skills` (expo-tailwind-setup) | https://www.nativewind.dev/llms.txt | Tailwind styling for React Native / Expo. |
| Tamagui | `tamagui/tamagui` | https://tamagui.dev/llms.txt | Compiler-optimized universal design system (web+native). |
| Gluestack UI | `gluestack/agent-skills` | https://gluestack.io/llms-full.txt | Copy-paste universal components on Tailwind v4 / NativeWind. |
| Unistyles | — | https://www.unistyl.es/llms.txt | Fast StyleSheet engine (C++ core, no theme re-renders). |
| Reanimated + Gesture Handler | — | https://docs.swmansion.com/react-native-reanimated/llms.txt | UI-thread 60/120fps animation + native gestures. |
| React Navigation | — | https://reactnavigation.org/llms-full.txt | Imperative nav when not using Expo Router. |
| VisionCamera / MMKV | `margelo/react-native-skills` | https://docs.expo.dev/llms-full.txt | Advanced camera (frame processors) + fast synchronous KV. |

### Auth

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Clerk | `clerk/skills` | https://clerk.com/docs/llms-full.txt | Hosted Expo auth: sessions, OAuth, webhooks, token handling. |
| better-auth | `better-auth/skills` | https://www.better-auth.com/llms.txt | Self-hosted TS auth: Expo client plugin, JWT/JWKS, Drizzle adapter. |
| Supabase Auth | `supabase/agent-skills` | https://supabase.com/llms.txt | Supabase Auth client, JWT signing keys/JWKS, RN sessions. |
| Auth0 | `auth0/agent-skills` | check https://auth0.com/docs for llms.txt | Enterprise SSO/compliance; has a React Native skill. |
| WorkOS | `workos/skills` | check https://workos.com/docs for llms.txt | B2B SSO, SCIM, directory sync. |
| Firebase Auth | `firebase/agent-skills` | check https://firebase.google.com/docs (no llms.txt) | Google ecosystem / quick mobile auth. |

### Backend frameworks

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Fastify | `mcollina/skills` | https://fastify.dev/llms.txt | Default Node TS API: low boilerplate, rich plugins. |
| Hono | `yusukebe/hono-skill` | https://hono.dev/llms.txt | Portable TS API (Node/Bun/edge), JWK middleware. |
| Elysia | `elysiajs/skills` | check https://elysiajs.com for llms.txt | Bun-first high-perf TS API. |
| tRPC | `trpc/trpc` | https://trpc.io/llms.txt | End-to-end typesafe RPC between mobile and API. |
| Convex | `get-convex/agent-skills` | https://docs.convex.dev/llms.txt | Reactive BaaS (DB + functions + live queries). |
| Go (chi + golang-migrate) | — | check https://go-chi.io and https://github.com/golang-migrate/migrate docs | Go API router, migrations, pgx/sqlc, JWKS verify. |
| Rust (axum + sqlx) | — | check https://docs.rs/axum and https://github.com/launchbadge/sqlx docs | Rust API framework, sqlx migrations, JWT/JWKS. |
| Python (FastAPI + Alembic) | — | check https://fastapi.tiangolo.com and https://alembic.sqlalchemy.org docs | FastAPI, SQLAlchemy/Alembic, PyJWT JWKS verify. |

### Data & ORM

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Drizzle | — | https://orm.drizzle.team/llms.txt | Default TS ORM: schema-in-TS, queries, migrations. |
| Prisma | `prisma/skills` | https://www.prisma.io/docs/llms.txt | ORM alternative: generated client, Studio, broad DB support. |
| Kysely | — | https://kysely.dev/llms.txt | Type-safe raw-SQL query builder (not a full ORM). |
| Neon | `neondatabase/agent-skills` | https://neon.com/llms.txt | Default managed serverless Postgres with branching. |
| Supabase (Postgres/BaaS) | `supabase/agent-skills` | https://supabase.com/llms.txt | Postgres + auth + storage + realtime in one platform. |
| PlanetScale | `planetscale/database-skills` | https://planetscale.com/llms.txt | Scalable MySQL (Vitess) or managed Postgres with branching. |
| Turso / libSQL | — | https://docs.turso.tech/llms.txt | SQLite at the edge or embedded/synced with a cloud tier. |
| MongoDB | — | https://www.mongodb.com/llms.txt | Document/flexible-schema data; Atlas for managed. |
| Expo SQLite | `expo/skills` | https://docs.expo.dev/llms.txt | Default on-device DB (works with Drizzle). |
| PowerSync | — | https://docs.powersync.com/llms.txt | Bidirectional offline sync between on-device SQLite and Postgres. |

### Cache & background jobs

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Redis | `redis/agent-skills` | https://redis.io/llms.txt | Self-hosted cache/queue/rate-limit on Coolify/VDS. |
| Upstash | `upstash/skills` | https://upstash.com/docs/llms.txt | Serverless Redis / rate-limit / QStash queues. |
| Trigger.dev | `triggerdotdev/skills` | https://trigger.dev/docs/llms.txt | Default durable background jobs/workflows in TS. |
| Inngest | — | https://www.inngest.com/llms.txt | Event-driven durable functions (fan-out, step retries). |

### Payments

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| RevenueCat | `revenuecat/ai-toolkit` | https://www.revenuecat.com/docs/llms.txt | Default IAP/subscriptions in Expo (StoreKit + Play Billing). |
| Stripe | `stripe/ai` | https://docs.stripe.com/llms.txt | Web billing, SaaS subs, marketplaces, checkout, Connect. |
| Adapty | — | https://adapty.io/docs/llms.txt | RevenueCat alternative with heavy paywall A/B testing. |
| Superwall | `superwall/skills` | https://superwall.com/docs/llms.txt | Remote-configurable paywall design + A/B testing. |
| Paddle | — | https://developer.paddle.com/llms.txt | Merchant-of-record for web/desktop SaaS (owns tax/VAT). |
| Polar | — | https://polar.sh/docs/llms.txt | Developer-first open-source MoR for digital products. |

### Observability

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Sentry | `getsentry/skills` | https://docs.sentry.io/llms.txt | Default crash/error + performance across mobile and API. |
| PostHog | `posthog/skills` | https://posthog.com/llms.txt | Default analytics, feature flags, session replay, A/B. |
| Amplitude | — | https://amplitude.com/docs/llms.txt | Deep behavioural analytics, funnels, retention. |
| Mixpanel | — | https://docs.mixpanel.com/llms.txt | Event-centric product analytics, ad-hoc reporting. |
| Aptabase | — | https://aptabase.com/llms.txt | Privacy-first open-source analytics (runs in Expo Go). |
| Datadog | `datadog-labs/agent-skills` | https://docs.datadoghq.com/llms.txt | Full-stack backend APM, logs, metrics, mobile RUM. |
| Axiom | `axiomhq/skills` | https://axiom.co/docs/llms.txt | Cheap high-volume log/event storage with APL queries. |

### AI / LLM

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| OpenRouter | `openrouterteam/agent-skills` | https://openrouter.ai/docs/llms.txt | Default gateway: one key to Claude/GPT/Gemini/Llama + fallback. |
| Vercel AI SDK | `vercel/ai` | https://ai-sdk.dev/llms.txt | Default TS toolkit to wire providers into handlers with streaming. |
| Anthropic (Claude) | `anthropics/skills` | https://platform.claude.com/llms.txt | Claude directly: reasoning, tool use, prompt caching, long context. |
| OpenAI | `openai/skills` | https://developers.openai.com/api/docs/llms.txt | GPT models, Realtime/voice, Whisper, image gen. |
| Google Gemini | — | https://ai.google.dev/gemini-api/docs/llms.txt | Large context, native multimodal, free tier for prototyping. |
| Groq | — | https://console.groq.com/llms.txt | Very low latency on open models; OpenAI-compatible. |
| Replicate | — | https://replicate.com/llms.txt | Image/video/audio gen or any community model behind one API. |
| fal.ai | — | https://fal.ai/docs/llms.txt | Fast image/video/audio generation with streaming/queue. |
| ElevenLabs | — | https://elevenlabs.io/docs/llms.txt | High-quality TTS, voice cloning, STT, conversational voice. |
| Mastra | `mastra-ai/skills` | https://mastra.ai/llms.txt | TS agents with tools, workflows, memory, RAG on the backend. |
| Pinecone | `pinecone-io/skills` | https://docs.pinecone.io/llms.txt | Managed high-scale vector DB with hybrid search (RAG). |

### Mobile state & data-fetching

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| TanStack Query | — | https://tanstack.com/query/latest/llms.txt | Default remote-data: caching, refetch, mutations, offline. |
| Zustand | — | https://zustand.docs.pmnd.rs/llms.txt | Default client/global state: small store with hooks. |
| Zod | — | https://zod.dev/llms.txt | Default validation + types; pairs with react-hook-form. |
| SWR | — | https://swr.vercel.app/llms.txt | Lighter alternative to TanStack Query for read-heavy fetch. |

### Notifications, email, storage

| Service | Skill | llms.txt | Use for |
|---|---|---|---|
| Expo Push | `expo/skills` | https://docs.expo.dev/llms.txt | Default push for Expo (single API, Expo push service). |
| OneSignal | — | https://documentation.onesignal.com/llms.txt | Multichannel campaign/segmentation (push+in-app+email+SMS). |
| Knock | `knocklabs/skills` | https://docs.knock.app/llms.txt | Notification-as-infra: workflows, preferences, in-app inbox. |
| Novu | `novuhq/skills` | https://docs.novu.co/llms.txt | Open-source notification infra (self-hostable). |
| Resend | `resend/resend-skills` | https://resend.com/docs/llms.txt | Default transactional email: clean API, React Email. |
| Cloudflare R2 | `cloudflare/skills` | https://developers.cloudflare.com/r2/llms.txt | Default object storage: S3-compatible, zero egress. |

## Routing Rules

- UI work: use `ui-ux-pro-max` first, then `frontend-design` or `design-taste-frontend`.
- Marketing/screenshot/landing-style composition: use `gpt-taste` when motion or high-conversion visuals matter.
- React component architecture: use `vercel-composition-patterns`.
- React performance: use `vercel-react-best-practices`.
- Postgres: use `postgresql-database-engineering`.
- Postgres performance: also use `supabase-postgres-best-practices`.
- Redis: use `redis-best-practices` before production cache, rate limit, queue, or AI quota work.
- Clerk Expo auth: use `clerk`, `clerk-expo`, and `clerk-expo-patterns`.
- Clerk web auth: use `clerk`, `clerk-nextjs-patterns` (or `clerk-tanstack-patterns`).
- Next.js work: use `nextjs-app-router-patterns`, then `nextjs-data-fetching`; `nextjs-performance` before optimization passes.
- Web SEO: use `seo-audit` (and `seo-geo` for AI-search visibility) before public web launches.
- Expo styling: use `expo-tailwind-setup`.
- Expo API calls: use `native-data-fetching`.
- Native SDKs: use `expo-dev-client`.
- Release: use `expo-deployment`, then `expo-cicd-workflows` if automation is needed.
- IAP/subscriptions: use `payments.md`, RevenueCat docs, and RevenueCat MCP if available.
- Stripe (web/server billing): use `stripe/ai` skills and `payments.md`; use the Stripe MCP for API exploration if available.
- New/unfamiliar integration: check its `llms.txt` and skills.sh entry first (see LLM Docs Registry).

## skills.sh References

- Directory: https://www.skills.sh/
- Expo skills directory: https://www.skills.sh/expo/skills
- Clerk skills directory: https://www.skills.sh/clerk/skills
- Supabase skills directory: https://www.skills.sh/supabase/agent-skills
- Redis skills directory: https://www.skills.sh/redis/agent-skills
- RevenueCat directory: https://www.skills.sh/revenuecat
- Stripe directory: https://www.skills.sh/stripe
- Expo Tailwind: https://www.skills.sh/expo/skills/expo-tailwind-setup
- Expo CI/CD: https://www.skills.sh/expo/skills/expo-cicd-workflows
- Expo deployment: https://www.skills.sh/expo/skills/expo-deployment
- Expo dev client: https://www.skills.sh/expo/skills/expo-dev-client
- Expo API routes: https://www.skills.sh/expo/skills/expo-api-routes
- Building native UI: https://www.skills.sh/expo/skills/building-native-ui
- Native data fetching: https://www.skills.sh/expo/skills/native-data-fetching
- Upgrading Expo: https://www.skills.sh/expo/skills/upgrading-expo
- Expo module: https://www.skills.sh/expo/skills/expo-module
- use-dom: https://www.skills.sh/expo/skills/use-dom
- Expo UI SwiftUI: https://www.skills.sh/expo/skills/expo-ui-swiftui
- Expo UI Jetpack Compose: https://www.skills.sh/expo/skills/expo-ui-jetpack-compose
- EAS Update insights: https://www.skills.sh/expo/skills/eas-update-insights
- Expo brownfield: https://www.skills.sh/expo/skills/expo-brownfield
- Add App Clip: https://www.skills.sh/expo/skills/add-app-clip
- Vercel React Native: https://www.skills.sh/vercel-labs/agent-skills/vercel-react-native-skills
- Vercel JSON render React Native: https://www.skills.sh/vercel-labs/json-render/react-native
