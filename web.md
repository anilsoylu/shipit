# Web Frontend Playbook

Default web stack is Next.js (App Router) + TypeScript + Tailwind CSS + shadcn/ui,
deployed on your Coolify/VDS (Hetzner) next to the API, Postgres, and Redis.
Vercel is the swap. The web frontend is pluggable — TanStack Start is the
supported framework swap. Whatever you pick must satisfy the Web Swap Contract
below.

## Product shapes

Decide the shape first — it changes the architecture:

- **Mobile + web** (default when both are in scope): `apps/api` stays the single
  source of truth; the web app is just another client, same as Expo. Web calls
  `apps/api`, never the DB directly from client code.
- **Web-only**: Next.js may be fullstack — route handlers / server actions ARE
  the backend. They must still satisfy the Backend Swap Contract in
  `backend.md` (auth verification, validation, migrations, webhook signatures).
  A standalone `apps/api` remains the right call when you expect a mobile
  client later or need non-TS workers.
- **Marketing site only**: Next.js static/ISR pages, no backend beyond forms.
  Skip auth/payments until the product needs them.

## Choosing a framework

- **Next.js App Router** (default): RSC, ISR/SSG for SEO-heavy pages, largest
  ecosystem, first-class Vercel deploy, mature Clerk/Stripe/Sentry integrations.
- **TanStack Start**: client-heavy apps (dashboards, tools), fully type-safe
  routing/loaders end to end, Vite ecosystem. Weaker fit for marketing/SEO-heavy
  surfaces; pair with a separate marketing site if needed.
- Static-only landing page → Next.js SSG or Astro; don't spin up app
  infrastructure for a brochure.

## Web Swap Contract

Every web frontend, either framework, must satisfy these:

1. Server secrets (DB URL, provider secret keys, webhook secrets) live in
   server-only env; never in `NEXT_PUBLIC_*` / client-exposed vars, never
   imported into client components. Only publishable keys ship to the browser.
2. Auth uses the provider's web SDK with httpOnly cookie sessions (or the
   provider's session handling); tokens never go to `localStorage`.
3. Protected data is fetched server-side (server components, loaders, route
   handlers) or from `apps/api` with the session attached — the browser never
   holds credentials for infrastructure.
4. Server-side code that mutates state verifies the session per the Auth Swap
   Contract in `auth.md`; server actions validate input like any API endpoint.
5. Public pages ship SEO metadata: title/description, canonical, Open Graph,
   `sitemap.xml`, `robots.txt`. See the SEO section below.
6. The app builds and runs on the chosen target (Coolify/VDS Docker by default;
   Vercel as swap) with an explicit env var list, like every other deployable.

## Scaffold (Next.js default)

- Create in `apps/web`: Next.js + TypeScript + App Router + Tailwind.
- Add shadcn/ui for primitives; do not hand-roll standard controls.
- Server components by default; add `"use client"` only at interactive leaves.
- `.env.example` with `NEXT_PUBLIC_*` public vars and server-only vars listed
  separately.
- Shared types come from `packages/types`, same rules as mobile: no server
  code imported into client components.

## Env rules

Public (browser-safe, `NEXT_PUBLIC_` prefix):

- API base URL, auth publishable key, PostHog key, Sentry DSN.

Server-only (no prefix, never imported in client components):

- `DATABASE_URL`, `REDIS_URL` (web-only fullstack shape only)
- Auth secret keys and webhook secrets
- `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` when in scope
- AI provider keys when in scope

## SEO essentials

The web analog of `aso.md`. Before a public launch:

- Per-page metadata via the framework's metadata API: unique title +
  description on every indexable page.
- `sitemap.xml` and `robots.txt` generated from route conventions.
- Open Graph + Twitter card images for shared surfaces (OG image route).
- Canonical URLs; no duplicate-content routes.
- Structured data (JSON-LD) when the product type warrants it (products,
  articles, events).
- Core Web Vitals sanity pass (Lighthouse) on the landing and top pages.
- Use the installed `seo-audit` / `seo-geo` skills for a full pass when SEO is
  a growth channel, not just hygiene.

## Web client libraries

Shared with mobile (same defaults, same llms.txt — see `mobile-libs.md` and the
LLM Docs Registry): TanStack Query, Zustand, react-hook-form + Zod.

Web-specific swaps for mobile-only libs:

| Concern | Mobile (Expo) | Web |
|---|---|---|
| Lists/virtualization | FlashList v2 | TanStack Virtual |
| Animation | Reanimated | Motion (framer-motion) or CSS |
| Fast KV / URL state | MMKV | URL state via nuqs; cookies/localStorage for prefs only (never tokens) |
| Navigation | Expo Router | App Router / TanStack Router |
| UI kit | React Native Reusables | shadcn/ui |

## Deployment

- **Coolify/VDS** (default): Next.js standalone output in a Dockerfile on the
  same Hetzner/Coolify box as the API, Postgres, and Redis; same deploy
  requirements as `backend.md` (env list, logs, health). Load the Coolify
  llms-full.txt before deploy work: https://coolify.io/docs/llms-full.txt
- **Vercel** (swap): zero-config, preview deploys per PR, edge network. Use
  when previews-per-PR or edge matter more than owning the box. Keep server
  env vars in the Vercel project, not in the repo.
- TanStack Start deploys as a Node server on Coolify the same way — same env
  discipline.

Deployment order (mobile + web shape): deploy `apps/api` first (see
`backend.md`), then point the web app's API base URL at it, then deploy web.

## Add a web framework

To support a framework not listed here, add a section that satisfies every Web
Swap Contract invariant, add its rows to the LLM Docs Registry in `skills.md`,
and link it from this file and the index files.

## Official References

- Next.js docs: https://nextjs.org/docs (llms.txt: https://nextjs.org/docs/llms.txt)
- TanStack Start: https://tanstack.com/start/latest (llms.txt: https://tanstack.com/start/latest/llms.txt)
- TanStack Router: https://tanstack.com/router/latest (llms.txt: https://tanstack.com/router/latest/llms.txt)
- Tailwind CSS: https://tailwindcss.com/docs (no llms.txt as of 2026-07-07)
- shadcn/ui: https://ui.shadcn.com/docs (llms.txt: https://ui.shadcn.com/llms.txt)
- Coolify: https://coolify.io/docs (llms-full.txt: https://coolify.io/docs/llms-full.txt)
- Vercel: https://vercel.com/docs (llms.txt: https://vercel.com/docs/llms.txt)
- Clerk Next.js: https://clerk.com/docs/quickstarts/nextjs
- Next.js metadata: https://nextjs.org/docs/app/building-your-application/optimizing/metadata
