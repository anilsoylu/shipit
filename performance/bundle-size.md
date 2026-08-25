# Performance: Bundle size

Part of `../performance.md` (loop, budgets, severity legend, and triage rules
live there). Applies to Next.js and Expo. Server stacks (Go, Python, Hono) have
no client bundle — their size concern is the container image, which is
`dockerfile-optimizer` territory, not this file.

The theme: bytes the user downloads and parses before anything works. On web
that is Time to Interactive; on mobile it is cold start and app store download
size.

## Measure first

```bash
# Next.js — per-route First Load JS, then the treemap
next build                                   # read the route table it prints
ANALYZE=true next build                      # @next/bundle-analyzer, opens the treemap

# Expo — Hermes bytecode + asset weight
EXPO_ATLAS=1 npx expo export --platform ios
npx expo-atlas .expo/atlas.jsonl             # per-module treemap
```

Budgets: Next.js First Load JS < 130 kB gzip per route; Expo JS bundle < 4 MB.

## Next.js

- **[BLOCKER] Split what most users never reach** — a chart library, a rich text
  editor, a map, a PDF viewer, or an emoji picker loaded at the module top level
  is downloaded by every visitor including the ones who never open it. Load it
  at the interaction that needs it.
  ```tsx
  // wrong
  import { Chart } from "recharts"                  // +90 kB on every route that touches this file
  // right
  const Chart = dynamic(() => import("./chart"), { ssr: false, loading: () => <Skeleton /> })
  ```
  Verify: `next build` route table — First Load JS for that route drops by the library's weight; the chunk appears in the network panel only after the interaction.
- **[BLOCKER] Keep server-only code out of the client bundle** — one accidental
  import of a server module into a client component drags its whole dependency
  tree (and sometimes a secret) across the boundary. Mark server modules so the
  build fails instead of silently shipping them.
  ```ts
  // right — at the top of any module that must never reach the browser
  import "server-only"
  ```
  Verify: `ANALYZE=true next build` — no `node_modules/pg`, `stripe`, or your `lib/server/*` in the client treemap. Cross-check `../security/secrets-config.md` for the secrets half of this.
- **[HARDEN] Kill barrel imports** — `import { Icon } from "some-ui-kit"` can pull
  the package's entire index. Next can rewrite these for known packages; do it
  rather than trusting tree-shaking.
  ```ts
  // next.config.ts
  experimental: { optimizePackageImports: ["lucide-react", "date-fns", "@radix-ui/react-icons"] }
  ```
  Verify: treemap shows only the used modules from that package, not the full index.
- **[HARDEN] One library per job** — `moment` + `date-fns` + `dayjs` in one
  lockfile is three copies of the same 20 kB idea. Same for icon sets, HTTP
  clients, and validation libraries.
  Verify: `npx depcheck` and the treemap; two libraries in the same category is a finding.
- **[HARDEN] Fonts and images are bundle weight too** — `next/font` self-hosts and
  subsets (no render-blocking Google Fonts request); `next/image` serves modern
  formats at the right size. An unoptimized hero image outweighs the entire JS
  bundle.
  Verify: Lighthouse reports no "Properly size images" or "Ensure text remains visible during webfont load" opportunity above 100ms.

## Expo

- **[BLOCKER] Do not bundle large assets into the app** — every image, font,
  Lottie file, and seed JSON in `assets/` ships in the binary and inflates both
  download size and cold start. Bundle only what the first screen needs;
  everything else is fetched and cached at runtime.
  ```tsx
  // wrong
  import hero from "../assets/hero-4k.png"          // 3 MB in the .ipa forever
  // right
  <Image source={{ uri: CDN_HERO }} cachePolicy="disk" />   // expo-image
  ```
  Verify: `npx expo-atlas` asset section, and the Xcode App Thinning report — the install size drops by the asset weight.
- **[BLOCKER] Import icon and utility sets by path, not by barrel** —
  `@expo/vector-icons` and similar re-export every family; one barrel import can
  add megabytes of font and JS.
  ```tsx
  // wrong
  import { Ionicons } from "@expo/vector-icons"
  // right
  import Ionicons from "@expo/vector-icons/Ionicons"
  ```
  Verify: atlas treemap — only the imported family appears.
- **[HARDEN] Enable inline requires** — without them every module in the graph is
  evaluated at startup, even ones the first screen never touches. Metro's
  `inlineRequires` defers evaluation to first use.
  ```js
  // metro.config.js
  config.transformer.getTransformOptions = async () => ({
    transform: { inlineRequires: true },
  })
  ```
  Verify: cold start to first interactive frame on a release build drops; measure 5 runs, take the median.
- **[HARDEN] Measure the release build, never the dev build** — the dev bundle is
  unminified, unbytecoded, and carries the dev client. A dev-build size number
  is not a finding.
  Verify: sizes come from `expo export` or a real `.ipa`/`.aab`, not from the Metro dev server.

## Do not

- Chase a 5 kB win while a 2 MB image ships next to it. Sort the treemap by size
  and start at the top.
- Split so aggressively that a click waits on a network round trip. Splitting
  moves the cost, it does not delete it — put the boundary where the user
  already expects a wait.
