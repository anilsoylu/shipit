# App Store Submission Runbook (hands-on)

The exact end-to-end process used to take an Expo/React Native app from "build on
TestFlight" to "submitted for review", with the real commands and the gotchas that
cost the most time. Complements `app-store.md` (the conceptual gate) and `aso.md`.

Tooling used:
- **ascelerate** — App Store Connect CLI (metadata, media, IAP, review). `ascelerate configure` once (needs ASC API key `.p8` + Issuer ID + Key ID; the `.p8` downloads **only once** at key creation).
- **`xcrun simctl`** — simulator control for screenshots (no TCC permissions needed, unlike cliclick/screencapture/System Events which are blocked in headless shells).
- **Headless Chrome** — render marketing screenshots (perfect multilingual text via macOS system fonts).
- **Claude-in-Chrome** (browser automation) — only for ASC web-UI-only screens (see Phase 6). Burns usage limits fast; prefer CLI.

Key principle: **do as much as possible via `ascelerate` (CLI). A surprising amount is web-UI-only** — plan for it.

---

## Phase 1 — Screenshots (tap-free, 15 languages)

App Store requires iPhone **6.9"/6.7"** screenshots: **1290×2796** (iPhone 15 Pro Max works; a 6.3" iPhone 17 Pro at 1206×2622 is NOT an accepted size). iPhone-only app (UIDeviceFamily=1) → no iPad screenshots needed.

Capture with zero UI taps (macOS TCC blocks synthetic clicks in headless shells):
- **Language switch** = set the simulator's device locale + reboot. Works when the app reads `getLocales()` and has no saved in-app language override:
  ```
  xcrun simctl spawn <sim> defaults write -g AppleLanguages -array "es"
  xcrun simctl spawn <sim> defaults write -g AppleLocale -string "es_ES"
  xcrun simctl shutdown <sim> && xcrun simctl boot <sim>
  ```
- **Navigation** = deep links (`xcrun simctl openurl <sim> "yourscheme://portfolio"`). For detail screens needing an id, fetch a real id from the backend API or read it once (temporary on-screen `<Text>{id}</Text>`, or the running Metro's `console.log`).
- **Clean status bar**: `xcrun simctl status_bar <sim> override --time "9:41" --dataNetwork wifi --wifiBars 3 --cellularMode active --cellularBars 4 --batteryState charged --batteryLevel 100` (resets on reboot — re-apply after each boot).
- **Capture**: `xcrun simctl io <sim> screenshot out.png` (native res, App-Store-ready).
- **Remove dev overlays** (dev-client / `expo run:ios` debug build):
  - LogBox error toasts → temporary `LogBox.ignoreAllLogs(true)` in the root layout (Metro fast-refreshes; revert after).
  - expo-dev-menu floating button → `xcrun simctl spawn <sim> defaults write <bundleId> EXDevMenuShowFloatingActionButton -bool NO` (persists across reboots).
- **Gotchas**: after reboot the dev client sometimes doesn't auto-reconnect to Metro (shows the launcher) → force with `openurl "exp+<slug>://expo-development-client/?url=http%3A%2F%2Flocalhost%3A8081"` (no confirm dialog once it's in recently-opened). RTL locales (Arabic) and slow boots need longer waits (12–14s after `openurl`) or the screen captures a skeleton/home fallback — always verify each capture.
- **Auth/onboarding**: no bypass → sign in once on the sim; onboarding flag is per-user in SecureStore, survives reboots.
- **Neutral currency**: for a multi-currency app, set the demo account's base currency to **USD** so net-worth/prices read universally across all 15 markets (not the home-market currency).

Loop all languages in a background shell script; verify with a montage (`montage` from ImageMagick). Budget ~1.5 min/language.

---

## Phase 2 — Marketing frames (optional but recommended)

Raw simulator captures are valid, but framed "caption + device + branded gradient" images convert better and their captions are indexed by Apple (since 2025). Build them with **headless Chrome** rendering an HTML/CSS template (1290×2796 canvas, gradient bg, localized headline, device screenshot with rounded corners + shadow), one per language:
```
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --force-device-scale-factor=1 \
  --window-size=1290,2796 --screenshot=out.png "file:///tmp/frame.html"
```
macOS system fonts cover Arabic (RTL), CJK, Devanagari, Cyrillic — set `dir="rtl"` for Arabic. Reuse the app's own onboarding slide titles as captions (already professionally localized). Keep raw captures in `media/`, framed in `framed/`.

---

## Phase 3 — ASO metadata (15 languages)

Fields and hard limits (per locale):
- **name** (app-info) — **30 chars**
- **subtitle** (app-info) — **30 chars**
- **keywords** (version) — **100 BYTES** (not chars! Arabic/CJK/Cyrillic/Devanagari = 2–3 bytes/char — validate with `Buffer.byteLength(s,'utf8')`)
- **description** (version) — 4000 chars (NOT indexed by Apple; conversion only)
- **promotionalText** (version) — 170 chars (updatable without a new build; not indexed)
- **whatsNew** (version) — release notes; **not required for a 1.0 first submission** (preflight tools flag it as a false positive)

Coordination rules (Apple indexes name + subtitle + keywords, each word once):
- Put top keywords in the **title** (strongest signal), asset-type keywords in subtitle, the rest in keywords — **no word repeats across the three**.
- Keyword list: comma-separated, **no spaces** after commas.
- Run generated copy through a **humanizer** pass (cut AI-tells: rigid "• Label: description" lists, three-part structures, puffery) so it doesn't read machine-generated.

Store as two JSON files (ascelerate import format):
- `app-infos.json`: `{ "en-US": { "name", "subtitle", "privacyPolicyURL" } }`
- `localizations.json`: `{ "en-US": { "keywords", "description", "promotionalText", "supportURL" } }`

Validate all lengths (chars for name/subtitle/desc/promo; **bytes** for keywords) before importing.

---

## Phase 4 — Upload via ascelerate

```
ascelerate apps app-info import <app> --file app-infos.json -y          # name + subtitle + privacy URL
ascelerate apps localizations import <app> --file localizations.json -y # keywords + description + promo (see Support URL caveat below)
ascelerate apps media upload <app> screenshots/framed -y                # media/{ascLocale}/APP_IPHONE_67/NN_name.png, alpha order = display order
ascelerate apps app-info update <app> --primary-category FINANCE -y
ascelerate apps build attach-latest <app> -y                            # attach the processed TestFlight build to the version
ascelerate apps review preflight <app>                                  # shows remaining gaps (ignore "What's New" for 1.0)
```
Import auto-creates missing locales. ASC locale codes: `en-US, es-ES, pt-BR, de-DE, fr-FR, it, ar-SA, id, ja, ko, hi, ru, nl-NL, pl, tr` (verify per app).

---

## Phase 5 — In-App Purchases / Subscriptions (biggest time sink)

Match ASC product IDs to the RevenueCat products (`store_identifier`, e.g. `premium_monthly`, `premium_yearly`). Check RevenueCat first (MCP or dashboard) for the exact IDs.

```
ascelerate sub create-group <app> --name "Premium" -y
ascelerate sub create <app> --name "Premium Monthly" --product-id premium_monthly --period ONE_MONTH -y
ascelerate sub create <app> --name "Premium Yearly"  --product-id premium_yearly  --period ONE_YEAR  -y
```

For each subscription to become **"Ready To Submit"** it needs FIVE things:
1. **Price** — `ascelerate sub pricing set` often returns **HTTP 409** for brand-new subs. Fallback: set the price in the **ASC web UI** (Subscriptions → product → Add Subscription Price → pick USD base → Apple auto-equalizes 175 territories → Confirm). The web flow works where the CLI fails.
2. **Localization** (display name ≤30 + description ≤45/55) — `ascelerate sub localizations import <app> <pid> --file sub.json` (`{ "en-US": { "name", "description" } }`). Include the app's primary language (and add en-US if you switch primary to English later).
3. **Availability** — web UI (Set Up Availability → All → Confirm), or `sub availability`. For a yearly sub with "Monthly-with-12-month-commitment", set the **1-Year-Upfront** availability first.
4. **Review screenshot** — `ascelerate sub review-screenshot upload <app> <pid> <paywall.png>` (CLI works; browser `file_upload` no longer accepts host paths). Use the paywall screenshot.
5. **⚠️ Subscription GROUP localization** — the single most common cause of a stuck "Missing Metadata". Even with all four above, the **group** needs a display-name localization:
   ```
   ascelerate sub group-localizations import <app> --file group.json   # { "en-US": { "name": "Premium" }, "tr": { "name": "Premium" } }
   ```
   Adding this flips both subs to "Ready To Submit".

At submission time the **first subscription must be included with the app version** — select the subs in the version page's "In-App Purchases and Subscriptions" section (only selectable once "Ready To Submit").

---

## Phase 6 — Web-UI-only items (ascelerate can't do these)

Use the ASC web site (or Claude-in-Chrome). None of these have CLI support:
- **App Privacy nutrition label** — data-collection questionnaire. Answer from the real privacy policy: pick data types (Contact Info, Financial Info, User Content, Identifiers, Purchases, Diagnostics...), and for each set purpose (**App Functionality**), linked-to-identity (**Yes** for account apps), and tracking (**No** if no ads/analytics). Then **Publish**.
- **Age Rating** — questionnaire (7 steps). A finance/tracker with no objectionable content → all "None"/"No" → **4+**.
- **Primary Language** — App Information → Primary Language dropdown (target locale must have a complete localization first). Not exposed by ascelerate.
- **Content Rights** — App Information → declare third-party content (a price/market-data app that shows provider data → "Yes... I have the necessary rights").
- **Copyright** — on the **version page** (not App Information), e.g. `2026 Your Name`.
- **Price tier** — Pricing and Availability → set **Free** (freemium apps monetize via IAP). App Availability at 175 countries is correct/desired.
- **App Review Information** (version page) — **contactFirstName + contactLastName + contactPhone are REQUIRED** and NOT settable by ascelerate (its `review info` command only has email/demo/notes → 409 without them). Fill contact + tick "Sign-in required" + demo account + review notes here.
- **Encryption** — if the build has `ITSAppUsesNonExemptEncryption=false` in Info.plist, no ASC declaration/documentation is needed.
- **Sign in with Apple** — required (4.8) if the app offers any third-party/social login (e.g. Google). Must be present in the app.
- **Demo account** — create one via the backend signup API (no manual account needed); confirm it can sign in (email-verification must not block it).

---

## Phase 7 — Support & Privacy URLs (subtle CLI trap)

- **`ascelerate apps localizations import` does NOT write `supportURL`/`whatsNew`** (only description/keywords/promo), even though it prints them. Use the per-locale `update`:
  ```
  ascelerate apps localizations update <app> --locale en-US --support-url "https://site.com/support"
  ```
  - **No `-y` flag** on `update` (it's an invalid option and makes every call fail).
  - In loops, `ascelerate` can drop out of PATH → call it by full path (`/opt/homebrew/bin/ascelerate`).
- Privacy Policy URL lives in **app-info** (`privacyPolicyURL`), Support URL in **version localizations**. Both must be **live** during review.
- Both URLs must be reachable in a real browser. Server-side/sandbox `curl`/`fetch` may get **522** from Cloudflare bot-blocking even when the site is fine in a browser — verify from an actual browser, not a headless fetch.

---

## Phase 8 — "Add for Review" checklist

The web "Add for Review" button surfaces the definitive blocker list. Common ones and where to fix:
| Blocker | Fix |
| --- | --- |
| Support URL required (all locales) | `ascelerate localizations update --support-url` (Phase 7) |
| Contact Information | version page → App Review Information (contact + demo + notes) |
| Copyright | version page → Copyright field |
| Price tier | Pricing and Availability → Free |
| Content Rights | App Information → declare |
| IAP "Missing Metadata" | group localization (Phase 5) |

Then the developer clicks **Add for Review → Submit** (do this last, after preflight is clean apart from the 1.0 "What's New" false positive).

---

## Legal-pages web note (Astro)

If the marketing/legal site (privacy/terms/support pages) 522s or 301-loops on trailing slashes: set Astro `trailingSlash: 'never'` + `build: { format: 'file' }` (flat `.html` files) and an nginx `try_files $uri $uri.html ...` with a `^/(.+)/$ → /$1` 301. Make link helpers emit no-slash paths. Keep per-page canonical/hreflang (SEO) but point the language switcher at each locale's homepage (`/`, `/tr`, `/es`), not the translated current path, if that's the desired UX.

---

## Time-savers next time
1. Configure `ascelerate` and confirm the RevenueCat product IDs before anything else.
2. Do screenshots + ASO + IAP config in parallel background jobs; verify every screenshot (RTL/slow-boot silently fail).
3. Expect the **group localization** + **support-url `update`** + **web-UI-only** traps — they cause most of the "why is it still not ready" loops.
4. Keep the demo account and backend live from submission through review.
