# Verified Stack Research (workflow-sourced)

> Produced by the `shipit-stack-research` dynamic workflow (run `wf_270a0984-502`) against commit `b48283b`.
> Every skills.sh slug and llms.txt URL was fetched during the run. `✓` = confirmed live in this run; `(verify)` = re-check with `curl` before shipping (came from a research agent whose field was not in the ground-truth verified set); `—` = none found.
> Ground-truth verified sets are listed first; per-domain tables follow. Auth/Backend provider tables are omitted here (covered by plans 002/003) — their verified skills/llms are in the sets below.
>
> **Executor rule when consuming this file:** treat `✓` values as safe to write. For any `(verify)` skill slug, `curl` the `https://www.skills.sh/<slug>` page (200 + a slug-specific title) before writing it; if it is not the official vendor skill or 404s, drop the skill and keep only the llms.txt. For any `(verify)` llms.txt, `curl` for a `200` returning real markdown before writing it; on 404 write `check <vendor> docs`. A control-test a known-404 slug to confirm your detection works.

## Verified skills.sh packs (ground truth — 13)

| Pack | Install | Notes |
|---|---|---|
| `expo/skills` | `npx skills add expo/skills` | Any Expo/React Native app — EAS deploy, dev-client, native UI, API routes, expo-tailwind-setup (NativeWind), CI/CD. |
| `clerk/skills` | `npx skills add clerk/skills` | Clerk auth; use `--skill clerk-expo-patterns` for React Native, plus clerk-setup/billing/orgs/webhooks. |
| `getsentry/skills` | `npx skills add getsentry/skills` | Crash/error monitoring + CI review workflows. Note: pack is mostly general DX (code-review, find-bugs), not RN-SDK-specific. |
| `PostHog/skills` | `npx skills add PostHog/skills` | Product analytics, feature flags, error tracking, LLM analytics. Large catalog — target a subset with `--skill`. |
| `stripe/ai` | `npx skills add stripe/ai --skill stripe-best-practices` | Stripe payments/billing on the backend. 5 skills: stripe-best-practices, upgrade-stripe, stripe-projects, stripe-directory, connect-recommend. |
| `supabase/agent-skills` | `npx skills add supabase/agent-skills --skill supabase` | All-in-one Postgres+auth+storage BaaS. Skills: supabase, supabase-postgres-best-practices, skill-creator. |
| `prisma/skills` | `npx skills add prisma/skills` | TS ORM alternative to Drizzle. |
| `neondatabase/agent-skills` | `npx skills add neondatabase/agent-skills --skill neon-postgres` | Serverless/branchable managed Postgres. |
| `redis/agent-skills` | `npx skills add redis/agent-skills` | Caching, rate limiting, sessions, search, vector search. |
| `upstash/skills` | `npx skills add upstash/skills` | Serverless Redis, rate limiting, queues (QStash), workflows. |
| `better-auth/skills` | `npx skills add better-auth/skills` | Self-hosted, code-owned auth on a TS backend (Clerk alternative). |
| `triggerdotdev/skills` | `npx skills add triggerdotdev/skills` | Durable background jobs, scheduled tasks, long-running AI workflows in TS. |
| `cloudflare/skills` | `npx skills add cloudflare/skills` | Workers/edge deploy, Durable Objects, R2, Turnstile. |

## Authoritative skill slugs (skills.sh search API, install-ranked)

Resolved directly from `https://www.skills.sh/api/search?q=<tool>` (install counts in brackets). **This section supersedes the `(verify)` flags in the per-domain Skill columns below** — use these slugs. Correction to an earlier claim: RevenueCat DOES have a skill (`revenuecat/ai-toolkit`); the earlier "no skill" was a wrong slug guess.

| Tool | Authoritative slug | Key skillIds | Vendor-owned? |
|---|---|---|---|
| Expo | `expo/skills` | expo-tailwind-setup, expo-deployment, expo-dev-client, expo-cicd-workflows | yes |
| Clerk | `clerk/skills` | clerk-expo-patterns, clerk-setup, clerk-webhooks | yes |
| better-auth | `better-auth/skills` | better-auth-best-practices, better-auth-security-best-practices, email-and-password-best-practices | yes |
| Supabase | `supabase/agent-skills` | supabase, supabase-postgres-best-practices | yes |
| Auth0 | `auth0/agent-skills` | auth0-react-native, auth0-quickstart, auth0-nextjs, auth0-react | yes |
| WorkOS | `workos/skills` | workos, workos-authkit-base, workos-authkit-react | yes |
| Firebase | `firebase/agent-skills` | firebase-auth-basics | yes |
| Fastify | `mcollina/skills` | fastify-best-practices | author (Fastify creator) |
| Hono | `yusukebe/hono-skill` | hono | author (Hono creator) |
| Elysia | `elysiajs/skills` | elysiajs | yes |
| NestJS | `kadajett/agent-nestjs-skills` | nestjs-best-practices | community |
| tRPC | `trpc/trpc` | trpc-router, client-setup, error-handling | yes (low installs) |
| Encore | `encoredev/skills` | encore-service, encore-api, encore-auth | yes |
| Convex | `get-convex/agent-skills` | convex, convex-quickstart, convex-setup-auth | yes |
| Appwrite | `appwrite/agent-skills` | appwrite-typescript, appwrite-cli | yes |
| Pocketbase | `greendesertsnow/pocketbase-skills` | pocketbase-best-practices | community |
| Deno | `denoland/skills` | deno-expert, deno-deploy | yes |
| Prisma | `prisma/skills` | prisma-database-setup, prisma-client-api, prisma-postgres | yes |
| Drizzle | `bobmatnyc/claude-mpm-skills` | drizzle-orm, drizzle-migrations | community (no vendor pack) |
| Kysely | `mindrally/skills` | kysely | community |
| PlanetScale | `planetscale/database-skills` | mysql, postgres, vitess | yes |
| Neon | `neondatabase/agent-skills` | neon-postgres | yes |
| Redis | `redis/agent-skills` | redis-development | yes |
| Upstash | `upstash/skills` | (upstash) | yes |
| Trigger.dev | `triggerdotdev/skills` | trigger-tasks, trigger-setup | yes |
| Cloudflare | `cloudflare/skills` | (workers/r2) | yes |
| RevenueCat | `revenuecat/ai-toolkit` | revenuecat-purchase-flow, revenuecat-paywall, revenuecat-testing-setup | yes |
| Stripe | `stripe/ai` | stripe-best-practices, stripe-projects, upgrade-stripe | yes |
| Superwall | `superwall/skills` | superwall-expo-quickstart, superwall-ios-quickstart | yes |
| Sentry (SDK setup) | `getsentry/sentry-for-ai` | sentry-sdk-setup, sentry-node-sdk, sentry-workflow | yes |
| Sentry (general DX) | `getsentry/skills` | code-review, find-bugs, security-review | yes |
| PostHog | `posthog/skills` (+ `posthog/ai-plugin` for instrument-*) | posthog-debugger; instrument-product-analytics | yes |
| Datadog | `datadog-labs/agent-skills` | dd-apm, dd-logs | yes |
| Axiom | `axiomhq/skills` | axiom-sre, query-metrics, axiom-alerting | yes (NOT charleswiltgen/axiom, which is Apple docs) |
| OpenRouter | `openrouterteam/agent-skills` | openrouter-typescript-sdk, create-agent | yes |
| Vercel AI SDK | `vercel/ai` | ai-sdk | yes |
| Anthropic | `anthropics/skills` | skill-creator, frontend-design | yes |
| OpenAI | `openai/skills` | openai-docs, security-best-practices | yes |
| Mastra | `mastra-ai/skills` | mastra, mastra-best-practices | yes |
| Pinecone | `pinecone-io/skills` | pinecone-docs, pinecone-cli | yes |
| Tamagui | `tamagui/tamagui` | tamagui | yes (low installs) |
| Gluestack | `gluestack/agent-skills` | gluestack-ui-v5, gluestack-ui-v4 | yes |
| Resend | `resend/resend-skills` | resend, resend-cli, send-email | yes |
| Knock | `knocklabs/skills` | knock-cli, notification-best-practices, in-app-ui | yes |
| Novu | `novuhq/skills` | novu-inbox-integration, novu-trigger-notification | yes |
| VisionCamera / MMKV | `margelo/react-native-skills` | react-native-vision-camera, react-native-mmkv | yes (Margelo) |

**Confirmed NO authoritative skills.sh pack** (use llms.txt only): NativeWind (use `expo/skills` → `expo-tailwind-setup`), Adapty, Paddle, Polar, Lemon Squeezy, Amplitude, Mixpanel, Aptabase, OpenPanel, Bugsnag, LogRocket, Better Stack, OpenTelemetry, OneSignal, Pusher, Inngest, Ably, MongoDB, Turso, Gemini/Groq/Together/Fireworks/Replicate/fal/ElevenLabs (no vendor pack found — re-check per-provider), TanStack Query, Zustand, react-hook-form, Zod, Valibot, SWR, Jotai, Redux Toolkit, FlashList, Reanimated, Gesture Handler, Skia, Moti, Lottie, React Navigation, Unistyles, UploadThing, PartyKit, BullMQ. (QStash → use `upstash/skills`; Expo Router → use `expo/skills`.)

## Verified llms.txt / docs endpoints (ground truth)

| URL | Live |
|---|---|
| https://claude.com/blog/introducing-dynamic-workflows-in-claude-code | 200 ✓ |
| https://raw.githubusercontent.com/marckrenn/claude-code-changelog/main/README.md | 200 ✓ |
| https://raw.githubusercontent.com/marckrenn/claude-code-changelog/main/meta/cli-surface.md | 200 ✓ |
| https://www.skills.sh/api/search | 200 ✓ |
| https://www.skills.sh/expo/skills/expo-deployment | 200 ✓ |
| https://docs.expo.dev/llms.txt (and /llms-full.txt) | 200 ✓ |
| https://reactnative.dev/llms.txt | 200 ✓ |
| https://www.nativewind.dev/llms.txt | 200 ✓ |
| https://clerk.com/llms.txt | 200 ✓ |
| https://www.better-auth.com/llms.txt | 200 ✓ |
| https://supabase.com/llms.txt | 200 ✓ |
| https://docs.convex.dev/llms.txt | 200 ✓ |
| https://orm.drizzle.team/llms.txt | 200 ✓ |
| https://www.prisma.io/docs/llms.txt | 200 ✓ |
| https://neon.com/llms.txt | 200 ✓ |
| https://docs.turso.tech/llms.txt | 200 ✓ |
| https://fastify.dev/llms.txt | 200 ✓ |
| https://hono.dev/llms.txt (and /llms-full.txt) | 200 ✓ |
| https://trpc.io/llms.txt | 200 ✓ |
| https://tanstack.com/llms.txt | 200 ✓ |
| https://zustand.docs.pmnd.rs/llms.txt | 200 ✓ |
| https://www.revenuecat.com/docs/llms.txt | 200 ✓ |
| https://docs.stripe.com/llms.txt | 200 ✓ |
| https://docs.sentry.io/llms.txt | 200 ✓ |
| https://posthog.com/llms.txt | 200 ✓ |
| https://openrouter.ai/docs/llms.txt (and /llms-full.txt) | 200 ✓ |
| https://firebase.google.com/llms.txt | 404 ✗ (Firebase has no llms.txt; check for a skill instead) |
| https://tailwindcss.com/llms.txt | 404 ✗ (use NativeWind's, which is live) |

## Database and ORM / data layer

Default TypeScript ORM stays Drizzle over Postgres (both have verified llms.txt, serverless-ready, RN-usable). Managed Postgres host: Neon or Supabase (both ship official skills). On-device: Expo SQLite default; offline-sync only when it's a core loop.

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| Drizzle ORM | default | `bobmatnyc/claude-mpm-skills/drizzle-orm` (verify) | https://orm.drizzle.team/llms-full.txt (verify) | Default TS ORM: type-safe schema-in-TS, thin SQL-like queries, serverless-ready. |
| Prisma ORM | strong-alt | `prisma/skills` ✓ | https://www.prisma.io/docs/llms.txt ✓ | Best-in-class DX, generated client, Prisma Studio, broad DB support. |
| Kysely | situational | (verify) | https://kysely.dev/llms.txt (verify) | Type-safe raw-SQL query builder (not a full ORM) — complex joins, zero magic. |
| TypeORM | avoid | (verify) | (verify) | Legacy decorator ORM; only for existing TypeORM/NestJS codebases. |
| Neon (managed Postgres) | default | `neondatabase/agent-skills` ✓ | https://neon.com/llms.txt ✓ | Default managed serverless Postgres: instant provisioning, branching. |
| Supabase (Postgres + BaaS) | strong-alt | `supabase/agent-skills` ✓ | https://supabase.com/llms-full.txt (verify) | Postgres + auth + storage + realtime in one platform. |
| PlanetScale (MySQL & Postgres) | situational | (verify) | https://planetscale.com/llms.txt (verify) | Horizontally-scalable MySQL (Vitess) or managed Postgres with branching. |
| Turso / libSQL | situational | (verify) | https://docs.turso.tech/llms.txt ✓ | SQLite at the edge or embedded/synced with a cloud tier. |
| Upstash (serverless Redis/Vector/QStash) | strong-alt | `upstash/skills` ✓ | https://upstash.com/docs/llms.txt (verify) | Serverless Redis for cache/rate-limit/queues when not running your own. |
| Redis (self-hosted / Redis Cloud) | situational | `redis/agent-skills` ✓ | https://redis.io/llms.txt (verify) | Current default cache/queue in shipit when running Redis on Coolify/VDS. |
| MongoDB | situational | (verify) | https://www.mongodb.com/llms.txt (verify) | Document/flexible-schema data; Atlas for managed. |
| Expo SQLite | default | `expo/skills` ✓ | https://docs.expo.dev/llms.txt ✓ | Default on-device DB for Expo (works with Drizzle). |
| op-sqlite | situational | — | — | Fastest RN SQLite (JSI) with SQLCipher/libSQL extensions. |
| WatermelonDB | situational | (verify) | — | Large offline-first RN apps needing a reactive, lazily-loaded store + sync. |
| PowerSync | situational | (verify) | https://docs.powersync.com/llms.txt (verify) | Robust bidirectional offline sync between on-device SQLite and Postgres/MySQL/Mongo. |
| Legend-State (sync) | situational | — | — | Fast observable state with built-in local persistence + sync plugins. |
| sqlc (Go) | situational | — | — | Go backends: type-safe Go from raw SQL (no ORM). |
| SQLAlchemy + Alembic (Python) | situational | (verify) | — | Python backends: standard ORM/core + migration tool. |
| Rust ORMs (sqlx / SeaORM / Diesel) | situational | — | — | Rust backends: sqlx (checked raw SQL), SeaORM (async), Diesel (mature sync). |
| Go ORMs (GORM / ent / pgx) | situational | — | — | Go backends: GORM (batteries), ent (schema-as-code), pgx (high-perf driver). |

## Payments and monetization

Default IAP/subscriptions: RevenueCat (wraps StoreKit + Play Billing, official Expo plugin, MCP, verified llms.txt). Web/hybrid billing: Stripe. Store-IAP mandate for in-app digital goods stays a hard rule.

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| RevenueCat | default | `revenuecat/ai-toolkit` ✓ | https://www.revenuecat.com/docs/llms.txt ✓ | Default for any IAP/subscription in Expo; wraps StoreKit + Play Billing. |
| Stripe | default (web) | `stripe/ai` ✓ | https://docs.stripe.com/llms.txt ✓ | Web billing, SaaS subs, marketplaces, any purchase outside the native app. |
| Adapty | strong-alt | — | https://adapty.io/docs/llms.txt (verify) | RevenueCat alternative with heavier paywall A/B testing + subscription analytics. |
| Superwall | situational | (verify) | https://superwall.com/docs/llms.txt (verify) | Remote-configurable paywall design + aggressive paywall A/B testing. |
| Native StoreKit (Apple) | situational | — | — | Only to avoid a third-party fee/dependency and hand-roll receipt validation. |
| Google Play Billing (native) | situational | (verify) | — | Native Android IAP without a third-party layer. |
| Paddle | situational | (verify) | https://developer.paddle.com/llms.txt (verify) | Merchant-of-record for web/desktop SaaS (owns global tax/VAT). |
| Polar | situational | (verify) | https://polar.sh/docs/llms.txt (verify) | Developer-first open-source Merchant of Record for digital products/subs. |
| Lemon Squeezy | situational | — | — | Simple MoR checkout for indie/web SaaS and license keys. |

## Analytics, crash and observability

Keep shipit defaults: PostHog (analytics) + Sentry (crash). Both have real skills, verified llms.txt, first-class Expo SDKs.

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| PostHog | default | `posthog/skills` ✓ | https://posthog.com/llms.txt ✓ | Default: product + web analytics, session replay, feature flags, A/B, error tracking. |
| Sentry | default | `getsentry/skills` ✓ | https://docs.sentry.io/llms.txt ✓ | Default crash/error + performance tracing across mobile and API. |
| Firebase (Analytics + Crashlytics) | strong-alt | `firebase/skills` (verify) | — | Already on Firebase/GCP; free analytics + battle-tested native crash reporting. |
| Amplitude | strong-alt | — | https://amplitude.com/docs/llms.txt (verify) | Deep behavioural analytics, funnels, retention, experimentation. |
| Mixpanel | strong-alt | — | https://docs.mixpanel.com/llms.txt (verify) | Event-centric product analytics with strong ad-hoc reporting. |
| Aptabase | situational | — | https://aptabase.com/llms.txt (verify) | Simple, GDPR-friendly, open-source analytics (works in Expo Go). |
| OpenPanel | situational | — | https://openpanel.dev/llms.txt (verify) | Open-source Mixpanel alternative, web + product, optional self-host. |
| Bugsnag (Insight Hub) | situational | — | — | Dedicated stability monitoring + JS-only Expo path. |
| LogRocket | situational | — | https://docs.logrocket.com/llms.txt (verify) | Pixel-perfect mobile session replay + Redux state capture. |
| Datadog | strong-alt | (verify) | https://docs.datadoghq.com/llms.txt (verify) | Full-stack backend APM, logs, metrics + optional mobile RUM. |
| Axiom | situational | (verify) | https://axiom.co/docs/llms.txt (verify) | Cheap high-volume log/event storage with APL queries for the backend. |
| Better Stack (Logtail + Uptime) | situational | — | — | Hosted log management + uptime/status page for the backend. |
| OpenTelemetry | situational | — | https://opentelemetry.io/llms.txt (verify) | Vendor-neutral backend instrumentation so you can swap vendors. |

## AI / LLM integration (backend for mobile apps)

LLM calls live server-side (never ship keys in the client). Default gateway OpenRouter + Vercel AI SDK integration; Anthropic Claude the default model family. RAG default: pgvector on the existing Postgres.

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| OpenRouter | default | `OpenRouterTeam/skills` (verify) | https://openrouter.ai/docs/llms.txt ✓ | Default gateway: one key to Claude/GPT/Gemini/Groq/Llama with fallback. |
| Vercel AI SDK | default | `vercel/ai` (verify) | https://ai-sdk.dev/llms.txt (verify) | Default TS toolkit to wire any provider into Fastify handlers with streaming. |
| Anthropic (Claude API) | strong-alt | `anthropics/skills` (verify) | https://platform.claude.com/llms.txt (verify) | Claude directly: best reasoning, tool use, prompt caching, long context. |
| OpenAI | strong-alt | `openai/skills` (verify) | https://developers.openai.com/api/docs/llms.txt (verify) | GPT models, Realtime/voice, Whisper, image gen. |
| Google Gemini | situational | — | https://ai.google.dev/gemini-api/docs/llms.txt (verify) | Large context, native multimodal, free tier for prototyping. |
| Groq | situational | — | https://console.groq.com/llms.txt (verify) | Very low latency on open models for real-time UX; OpenAI-compatible. |
| Together AI | situational | (verify) | https://docs.together.ai/llms.txt (verify) | Broad open-weight catalog + fine-tuning. |
| Fireworks AI | situational | — | https://docs.fireworks.ai/llms.txt (verify) | Fast, cost-efficient open-model serving; OpenAI-compatible. |
| Replicate | situational | (verify) | https://replicate.com/llms.txt (verify) | Image/video/audio generation or any community/custom model behind one API. |
| fal.ai | situational | — | https://fal.ai/docs/llms.txt (verify) | Fast image/video/audio generation with streaming/queue (Flux etc.). |
| ElevenLabs | situational | (verify) | https://elevenlabs.io/docs/llms.txt (verify) | High-quality TTS, voice cloning, STT, conversational voice. |
| Mastra | situational | (verify) | https://mastra.ai/llms.txt (verify) | TS agents with tools, workflows, memory, RAG on the Node/Fastify backend. |
| LangGraph (LangChain) | situational | — | https://docs.langchain.com/llms.txt (verify) | Stateful graph-based multi-agent orchestration with durable execution. |
| pgvector (Postgres) | strong-alt | — | — | Default RAG store: vector columns/indexes on existing Postgres+Drizzle. |
| Upstash Vector | situational | `upstash/skills` ✓ | https://upstash.com/docs/llms.txt (verify) | Serverless HTTP vector DB (pairs with Upstash Redis). |
| Pinecone | situational | (verify) | https://docs.pinecone.io/llms.txt (verify) | Fully managed high-scale vector DB with hybrid search. |
| Turso Vector (libSQL) | situational | — | https://docs.turso.tech/llms.txt ✓ | Vector columns + ANN in the same SQLite/libSQL DB. |
| Portkey | situational | (verify) | https://portkey.ai/docs/llms.txt (verify) | AI gateway: routing, caching, guardrails, budgets, observability. |
| LiteLLM | situational | (verify) | https://docs.litellm.ai/llms.txt (verify) | Open-source self-hostable proxy, 100+ providers, OpenAI-compatible. |
| Helicone | situational | — | https://docs.helicone.ai/llms.txt (verify) | Drop-in LLM observability (logging, caching, cost/latency). |

## Mobile state, data-fetching and storage (React Native / Expo)

Defaults: TanStack Query (server-state), Zustand (client state), react-hook-form + Zod (forms), expo-secure-store (secrets), react-native-mmkv (fast KV), FlashList v2 (lists).

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| TanStack Query | default | (verify) | https://tanstack.com/query/latest/llms.txt (verify) | Default remote-data: caching, refetch, mutations, pagination, offline. |
| Zustand | default | (verify) | https://zustand.docs.pmnd.rs/llms.txt ✓ | Default client/global state: small store with hooks, no boilerplate. |
| react-hook-form + Zod | default | (verify) | https://zod.dev/llms.txt (verify) | Default form stack: performant forms + one Zod schema for validation + types. |
| expo-secure-store | default | `expo/skills` ✓ | https://docs.expo.dev/llms.txt ✓ | Store tokens/secrets encrypted (Keychain/Keystore). Never MMKV/AsyncStorage for secrets. |
| react-native-mmkv | strong-alt | `margelo/react-native-skills` (verify) | — | Fast synchronous KV (settings, flags, cache). ~30x faster than AsyncStorage. |
| expo-sqlite | strong-alt | — | https://docs.expo.dev/llms.txt ✓ | On-device SQL: relational data, complex queries, offline-first. |
| FlashList (v2) | default | — | — | Default high-perf virtualized list; drop-in FlatList replacement. |
| SWR | strong-alt | (verify) | https://swr.vercel.app/llms.txt (verify) | Lighter alternative to TanStack Query for read-heavy fetching. |
| tRPC (client) | situational | (verify) | https://trpc.io/llms.txt ✓ | End-to-end type safety when you own a TS backend. |
| Jotai | situational | (verify) | — | Atomic bottom-up state with fine-grained re-renders. |
| Redux Toolkit + RTK Query | situational | (verify) | — | Large/complex apps or teams already on Redux. |
| TanStack Form | situational | (verify) | https://tanstack.com/form/latest/llms.txt (verify) | Headless fully type-safe form layer (Zod/Valibot). |
| Valibot | situational | (verify) | https://valibot.dev/llms.txt (verify) | Validation when bundle size matters (tree-shakeable, smaller than Zod). |
| Legend-State + Legend-List | situational | — | — | Maximal perf + local-first sync: fine-grained reactivity, persistence. |

## Mobile UI, styling and native capability (Expo / React Native)

Default styling/UI: NativeWind + React Native Reusables. Animation: Reanimated + Gesture Handler. Navigation: Expo Router.

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| NativeWind + React Native Reusables | default | `expo/skills` ✓ (expo-tailwind-setup) | https://www.nativewind.dev/llms-full.txt (verify) | Default: Tailwind-class styling + shadcn-style copy-paste RN components. |
| Tamagui | strong-alt | `tamagui/tamagui` (verify) | https://tamagui.dev/llms.txt (verify) | Compiler-optimized universal (web+native) design system with tokens/themes. |
| Gluestack UI (v5 / v4) | situational | (verify) | https://gluestack.io/llms-full.txt (verify) | Copy-paste universal component library on Tailwind v4 / NativeWind. |
| Unistyles 3.0 | situational | — | https://www.unistyl.es/llms.txt (verify) | Fast StyleSheet-compatible engine (C++ core, no re-renders on theme change). |
| Reanimated | default | `software-mansion-labs/skills` (verify) | https://docs.swmansion.com/react-native-reanimated/llms.txt (verify) | Default animation engine: UI-thread 60/120fps. |
| Gesture Handler | default | `software-mansion-labs/skills` (verify) | https://docs.swmansion.com/react-native-gesture-handler/llms.txt (verify) | Native-thread touch/gesture; pairs with Reanimated. |
| Moti | situational | — | — | Simplest declarative animation API (Framer-Motion-like) on Reanimated. |
| Skia (@shopify/react-native-skia) | situational | `software-mansion-labs/skills` (verify) | — | Custom 2D graphics, canvas, shaders, image filters. |
| Lottie (lottie-react-native) | situational | — | — | Play designer After Effects / JSON vector animations. |
| Expo Router | default | `expo/skills` ✓ | https://docs.expo.dev/llms-full.txt (verify) | Default file-based navigation; deep links, typed routes. |
| React Navigation | strong-alt | — | https://reactnavigation.org/llms-full.txt (verify) | Imperative/component-based nav control or non-Expo-Router apps. |
| VisionCamera | situational | — | https://react-native-vision-camera.com/llms-full.txt (verify) | Advanced camera: frame processors, ML/QR/barcode, custom capture. |
| Expo native capability (dev-client, camera, image, notifications) | default | `expo/skills` ✓ | https://docs.expo.dev/llms-full.txt (verify) | Default native primitives. |
| vercel-react-native-skills | strong-alt | `vercel-labs/agent-skills` (verify) | — | Cross-cutting RN performance/best-practices layer on top of any UI choice. |

## Notifications, email, storage, realtime, jobs

Pragmatic defaults: Expo Push (notifications), Resend (email), Cloudflare R2 (storage), Trigger.dev or QStash (jobs — HTTP-first, no self-run workers).

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| Expo Push (expo-notifications) | default | `expo/skills` ✓ | https://docs.expo.dev/llms.txt ✓ | Default push for Expo: single API, Expo push service. |
| OneSignal | strong-alt | — | https://documentation.onesignal.com/llms.txt (verify) | Multichannel campaign/segmentation dashboard (push+in-app+email+SMS). |
| FCM (@react-native-firebase/messaging) | situational | — | https://rnfirebase.io/llms.txt (verify) | Own the APNs/FCM pipeline directly (data messages, background handlers). |
| Knock | situational | (verify) | https://docs.knock.app/llms.txt (verify) | Notification-as-infra: workflows, preferences, batching, in-app inbox. |
| Novu | situational | (verify) | https://docs.novu.co/llms.txt (verify) | Open-source notification infra (self-hostable). |
| Courier | situational | — | https://www.courier.com/docs/llms.txt (verify) | Multichannel notification API with template designer. |
| Resend | default | `resend/react-email` (verify) | https://resend.com/docs/llms.txt (verify) | Default transactional email: clean API, React Email templates. |
| Postmark | strong-alt | (verify) | https://postmarkapp.com/llms.txt (verify) | When deliverability is the priority (strong sending reputation). |
| Loops | situational | — | https://loops.so/docs/llms.txt (verify) | Transactional + lifecycle/marketing email with no-code journeys. |
| SendGrid (Twilio) | situational | (verify) | https://www.twilio.com/docs/llms.txt (verify) | High-volume email or already in Twilio ecosystem. |
| Cloudflare R2 | default | `cloudflare/skills` ✓ | https://developers.cloudflare.com/r2/llms.txt (verify) | Default object storage: S3-compatible, zero egress. |
| AWS S3 | strong-alt | — | https://docs.aws.amazon.com/llms.txt (verify) | Deepest ecosystem/compliance or already on AWS. |
| UploadThing | situational | — | — | Opinionated batteries-included upload flow (auth, presign, callbacks). |
| Supabase Storage | situational | — | https://supabase.com/llms.txt ✓ | Already on Supabase; storage with RLS-based access. |
| Supabase Realtime | situational | — | https://supabase.com/llms.txt ✓ | Realtime on Postgres: broadcast, presence, change subscriptions. |
| Ably | strong-alt | — | https://ably.com/llms.txt (verify) | Production-grade managed realtime (pub/sub, presence, delivery guarantees). |
| Pusher Channels | situational | — | — | Simplest managed pub/sub realtime. |
| PartyKit | situational | — | — | Custom realtime/multiplayer server logic (rooms) on Cloudflare edge. |
| Convex | situational | — | https://docs.convex.dev/llms.txt ✓ | Reactive backend (DB + functions + live queries) as a Fastify+Postgres alternative. |
| Trigger.dev | default | `triggerdotdev/skills` ✓ | https://trigger.dev/docs/llms.txt (verify) | Default durable background jobs/workflows in TS (retries, scheduling). |
| QStash (Upstash) | strong-alt | `upstash/skills` ✓ | https://upstash.com/docs/llms.txt (verify) | Serverless message queue / delayed & scheduled HTTP jobs, zero infra. |
| Inngest | strong-alt | — | https://www.inngest.com/llms.txt (verify) | Event-driven durable functions/workflows (fan-out, step retries). |
| BullMQ (Redis) | situational | — | https://docs.bullmq.io/llms.txt (verify) | Already run Redis on VDS and want self-hosted queues/workers in Node. |

## Auth providers

Default Clerk (hosted). better-auth for self-hosted TS. Supabase Auth if already on Supabase. Every provider must satisfy the Auth Swap Contract in plan 002. Slugs resolved via the search API (see Authoritative slugs above).

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| Clerk | default | `clerk/skills` ✓ | https://clerk.com/llms.txt ✓ | Fastest hosted Expo auth; polished sessions, OAuth, `clerk-expo-patterns` skill. |
| better-auth | strong-alt | `better-auth/skills` ✓ | https://www.better-auth.com/llms.txt ✓ | Self-hosted, code-owned auth on a TS backend; you own the user table. |
| Supabase Auth | strong-alt | `supabase/agent-skills` ✓ | https://supabase.com/llms.txt ✓ | Already on Supabase / want Postgres-native auth with RLS. |
| Auth0 | situational | `auth0/agent-skills` ✓ (auth0-react-native) | https://auth0.com/docs (check llms.txt) | Enterprise SSO/compliance needs; has a React Native skill. |
| Firebase Auth | situational | `firebase/agent-skills` ✓ | — (no llms.txt) | Google ecosystem / quick mobile auth. |
| WorkOS | situational | `workos/skills` ✓ | https://workos.com/docs (check llms.txt) | B2B SSO, SCIM, directory sync. |
| Kinde / Logto / Stytch / Ory | situational | (no verified pack) | check vendor docs/llms.txt | Niche/self-host identity; verify skill+llms before use. |
| Custom JWT | situational | — | — | Only when a hard requirement forces it (you own all security). |

## Backend frameworks and runtimes

Default Fastify TypeScript on Coolify/VDS. Every backend must satisfy the Backend Swap Contract in plan 003.

| Option | Tier | Skill | llms.txt | When to use |
|---|---|---|---|---|
| Fastify (TS) | default | `mcollina/skills` (Fastify creator) | https://fastify.dev/llms.txt ✓ | Default Node TS API: low boilerplate, rich plugins. |
| Hono (TS) | strong-alt | `yusukebe/hono-skill` (Hono creator) | https://hono.dev/llms.txt ✓ | Edge/portable runtimes (Node/Bun/Workers/Deno). |
| Elysia (TS/Bun) | situational | `elysiajs/skills` ✓ | check hono/elysia docs | Bun-first, high-perf TS API. |
| NestJS (TS) | situational | `kadajett/agent-nestjs-skills` (community) | check docs | Opinionated enterprise structure/DI. |
| tRPC (TS layer) | situational | `trpc/trpc` ✓ | https://trpc.io/llms.txt ✓ | End-to-end typesafe RPC between mobile and API. |
| Encore (TS) | situational | `encoredev/skills` ✓ | check docs | Infra-from-code TS backend framework. |
| Go (chi/echo/fiber) | situational | — | check per-lib docs | Performance, single binary, strong concurrency. |
| Rust (axum/actix) | situational | — | check per-lib docs | Max performance/safety. |
| Python (FastAPI) | situational | — | check FastAPI docs/llms.txt | Python ecosystem, ML-adjacent work. |
| Convex (BaaS) | strong-alt | `get-convex/agent-skills` ✓ (440k installs) | https://docs.convex.dev/llms.txt ✓ | Reactive backend (DB + functions + live queries), no server to run. |
| Appwrite / Pocketbase / Nhost (BaaS) | situational | `appwrite/agent-skills` ✓ / `greendesertsnow/pocketbase-skills` | check docs | Self-hostable all-in-one backend. |

## Notes for planners

- **Ground truth vs. leads:** the "Verified skills.sh packs" and "Verified llms.txt" sections at the top were fetched and confirmed. Everything marked `(verify)` in the per-domain tables is a research-agent lead that was NOT in those ground-truth sets — verify before writing.
- **Suspect aggregator slugs:** repeated non-vendor slugs (e.g. `agents-inc/skills`, `mindrally/skills`, `bobmatnyc/claude-mpm-skills`) appeared across unrelated libraries. These are almost certainly not the authoritative vendor skill — prefer the vendor's own or drop the skill and keep the llms.txt.
- **Auth/Backend:** the first run's verifier produced degenerate output for these two domains; they were re-resolved directly via the skills.sh search API and are now included above (Auth providers / Backend frameworks tables + Authoritative slugs). Plans 002/003 own the doc structure; use these tables for the verified slugs/llms.
- **Skill slugs are now API-resolved, not guessed:** the Authoritative slugs table was built by querying `skills.sh/api/search` and ranking by install count. Where it lists a slug, it is real. The remaining `(verify)` marks in per-domain tables are for llms.txt URLs only (curl before shipping).
