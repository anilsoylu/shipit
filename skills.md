# Agent Skills Routing

Goal: make agents use the right specialized skill before touching important subsystems.

## Core Installed Skills

Use these when available:

- `postgresql-database-engineering`: schema design, indexes, migrations, query performance, backups, production Postgres.
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

## Clerk Docs

Before implementing Clerk auth, use the official LLM docs:

- https://clerk.com/docs/llms-full.txt

Use this for current Expo SDK patterns, backend request verification, webhooks, auth flows, and token handling.

## Routing Rules

- UI work: use `ui-ux-pro-max` first, then `frontend-design` or `design-taste-frontend`.
- Marketing/screenshot/landing-style composition: use `gpt-taste` when motion or high-conversion visuals matter.
- React component architecture: use `vercel-composition-patterns`.
- React performance: use `vercel-react-best-practices`.
- Postgres: use `postgresql-database-engineering`.
- Postgres performance: also use `supabase-postgres-best-practices`.
- Redis: use `redis-best-practices` before production cache, rate limit, queue, or AI quota work.
- Clerk Expo auth: use `clerk`, `clerk-expo`, and `clerk-expo-patterns`.
- Expo styling: use `expo-tailwind-setup`.
- Expo API calls: use `native-data-fetching`.
- Native SDKs: use `expo-dev-client`.
- Release: use `expo-deployment`, then `expo-cicd-workflows` if automation is needed.
- IAP/subscriptions: use `payments.md`, RevenueCat docs, and RevenueCat MCP if available.

## skills.sh References

- Directory: https://www.skills.sh/
- Expo skills directory: https://www.skills.sh/expo/skills
- Clerk skills directory: https://www.skills.sh/clerk/skills
- Supabase skills directory: https://www.skills.sh/supabase/agent-skills
- Redis skills directory: https://www.skills.sh/redis/agent-skills
- RevenueCat directory: https://www.skills.sh/revenuecat
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
