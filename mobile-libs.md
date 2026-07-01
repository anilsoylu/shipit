# Mobile Libraries Menu

Advisory menu for Expo/React Native client libraries — state, data-fetching,
forms, storage, lists, animation, and native capability. This is a menu, not a
contract domain: take the defaults unless a swap note fits. Load each option's
skill + `llms.txt` (also in the LLM Docs Registry in `skills.md`) before coding.

**Secrets rule (hard):** store tokens/secrets in `expo-secure-store` only —
never MMKV, AsyncStorage, or an on-device DB.

## Defaults

| Concern | Default | Skill | llms.txt |
|---|---|---|---|
| Server-state / data-fetching | TanStack Query | — | https://tanstack.com/query/latest/llms.txt |
| Client / global state | Zustand | — | https://zustand.docs.pmnd.rs/llms.txt |
| Forms + validation | react-hook-form + Zod | — | https://zod.dev/llms.txt |
| Secrets storage | expo-secure-store | `expo/skills` | https://docs.expo.dev/llms.txt |
| Fast KV cache | react-native-mmkv | `margelo/react-native-skills` | check https://github.com/mrousavy/react-native-mmkv |
| Lists | FlashList v2 | — | check https://shopify.github.io/flash-list |
| Animation + gesture | Reanimated + Gesture Handler | `software-mansion-labs/skills` | https://docs.swmansion.com/react-native-reanimated/llms.txt |
| Navigation | Expo Router | `expo/skills` | https://docs.expo.dev/llms-full.txt |

## When to swap

- **SWR** — lighter read-heavy fetching vs TanStack Query. llms.txt https://swr.vercel.app/llms.txt
- **tRPC (client)** — end-to-end type safety when you own the TS backend. Skill `trpc/trpc`; llms.txt https://trpc.io/llms.txt
- **Jotai** — atomic bottom-up state with fine-grained re-renders. Check https://jotai.org/docs.
- **Redux Toolkit + RTK Query** — large/complex apps or teams already on Redux. Check https://redux-toolkit.js.org.
- **TanStack Form** — headless fully type-safe form layer. llms.txt https://tanstack.com/form/latest/llms.txt
- **Valibot** — validation when bundle size matters (smaller than Zod). llms.txt https://valibot.dev/llms.txt
- **Legend-State** — maximal perf + local-first sync. Check https://legendapp.com/open-source/state.
- **op-sqlite** — fastest RN SQLite (JSI). See `data.md`.
- **expo-sqlite** — on-device relational data. Skill `expo/skills`; see `data.md`.

## Animation & graphics

- **Reanimated + Gesture Handler** (default) — UI-thread 60/120fps animation and
  native gestures. Skill `software-mansion-labs/skills`; llms.txt https://docs.swmansion.com/react-native-reanimated/llms.txt
- **Skia** (situational) — custom 2D graphics, canvas, shaders. Skill `software-mansion-labs/skills`.
- **Moti** (situational) — simplest declarative animation API on Reanimated.
- **Lottie** (situational) — play After Effects / JSON vector animations.

## Native capability

- **Default primitives** — `expo-dev-client`, `expo-camera`, `expo-image`,
  `expo-notifications`. Skill `expo/skills`; llms.txt https://docs.expo.dev/llms-full.txt
- **VisionCamera** (advanced camera: frame processors, ML/QR/barcode). llms.txt https://react-native-vision-camera.com/llms-full.txt
