# App Store Submission Runbook (hands-on)

The exact end-to-end process used to take an Expo/React Native app from "build on
TestFlight" to "submitted for review", with the real commands and the gotchas that
cost the most time. Complements `app-store.md` (the conceptual gate) and `aso.md`.

Tooling used:

- **`asc`** (rorkai/App-Store-Connect-CLI) — App Store Connect CLI (metadata, media, IAP, age rating, review, submission). `asc auth login --name <n> --key-id <KEY_ID> --issuer-id <ISSUER_ID> --private-key <path/AuthKey.p8>` once (the `.p8` downloads **only once** at key creation; asc stores the key material in the macOS keychain). Disable telemetry: `asc telemetry disable`.
- **`xcrun simctl`** — simulator control for screenshots (no TCC permissions needed, unlike cliclick/screencapture/System Events which are blocked in headless shells).
- **Headless Chrome** — render marketing screenshots (perfect multilingual text via macOS system fonts).
- **Claude-in-Chrome** (browser automation) — only for the few remaining web-UI-only screens (see Phase 6, now mostly the privacy nutrition label). Burns usage limits fast; prefer CLI.

Key principle: **do as much as possible via `asc` (CLI). `asc` covers far more than the old tooling — age rating, primary language, content rights, review contact info, copyright, and price are all CLI now; the privacy nutrition label is the main web-only holdout.**

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

Two field groups, two asc resources:

- **app-info** localizations (`name`, `subtitle`, `privacyPolicyURL`) — set with `asc app-setup info set`.
- **version** localizations (`keywords`, `description`, `promotionalText`, `supportURL`, `whatsNew`) — set with `asc localizations update`.

Validate all lengths (chars for name/subtitle/desc/promo; **bytes** for keywords) before writing.

---

## Phase 4 — Upload via asc

Resolve IDs first: `asc apps list --bundle-id <bundleId>` → APP_ID; `asc versions list --app <APP_ID>` → VERSION_ID. Then per locale:

```
# app-info: name + subtitle (+ privacy URL / primary language on the primary locale)
asc app-setup info set --app <APP_ID> --locale <loc> --name "..." --subtitle "..."
asc app-setup info set --app <APP_ID> --primary-locale en-US --privacy-policy-url "https://site.com/privacy"

# version localization: keywords + description + promo + support URL (asc DOES write support-url/whats-new)
asc localizations update --version <VERSION_ID> --locale <loc> --keywords "a,b,c" --description "..." --promotional-text "..." --support-url "https://site.com/support"

# media: fan out over locale dirs under ./screenshots/framed (run `asc screenshots sizes` for exact device tokens)
asc screenshots upload --app <APP_ID> --version-id <VERSION_ID> --path ./screenshots/framed --device-type IPHONE_67

# category + build + readiness
asc app-setup categories set --app <APP_ID> --primary FINANCE
asc versions attach-build --version-id <VERSION_ID> --build-id <BUILD_ID>
asc review doctor --app <APP_ID>   # explains remaining blockers (ignore "What's New" for a 1.0)
```

`asc localizations create --version <VERSION_ID> --locale <loc>` auto-adds a missing locale; the bulk `asc metadata` canonical-file workflow (`init` → edit → `push`) is the alternative for many locales at once. ASC locale codes: `en-US, es-ES, pt-BR, de-DE, fr-FR, it, ar-SA, id, ja, ko, hi, ru, nl-NL, pl, tr` (verify per app).

---

## Phase 5 — In-App Purchases / Subscriptions (biggest time sink)

Match ASC product IDs to the RevenueCat products (`store_identifier`, e.g. `premium_monthly`, `premium_yearly`). Check RevenueCat first (MCP or dashboard) for the exact IDs.

The fastest path for a new sub is the one-shot `asc subscriptions setup` (it does group localization + sub localization + review screenshot + pricing + availability and verifies the final state):

```
asc subscriptions setup --app <APP_ID> --group-reference-name "Premium" \
  --reference-name "Premium Monthly" --product-id premium_monthly --subscription-period ONE_MONTH \
  --price 3.99 --price-territory "United States" --territories "US,..." --locale en-US --name "Premium" --description "..."
```

Low-level equivalents (for repair flows) — first create the group + subs:

```
asc subscriptions groups create --app <APP_ID> --reference-name "Premium"
asc subscriptions create --group-id <GROUP_ID> --reference-name "Premium Monthly" --product-id premium_monthly --subscription-period ONE_MONTH
asc subscriptions create --group-id <GROUP_ID> --reference-name "Premium Yearly"  --product-id premium_yearly  --subscription-period ONE_YEAR
```

For each subscription to become **"Ready To Submit"** it needs FIVE things (product-scoped resources are deprecated — use the version-scoped commands below):

1. **Price** — `asc subscriptions pricing prices set --subscription-id <SUB_ID> --price-point <PP_ID>` (or `asc subscriptions pricing equalize --subscription-id <SUB_ID> --base-price 3.49`) can still return **HTTP 409** for brand-new subs. Fallback: set the price in the **ASC web UI** (Subscriptions → product → Add Subscription Price → USD base → Apple auto-equalizes → Confirm). Apple's restriction, not a CLI limit.
2. **Localization** (display name ≤30 + description ≤45/55) — `asc subscriptions versions localizations create --version-id <SUB_VERSION_ID> --locale en-US --name "..." --description "..."`. Include the app's primary language.
3. **Availability** — `asc subscriptions pricing availability` (or handled by `asc subscriptions setup`). For a yearly sub with "Monthly-with-12-month-commitment", set the **1-Year-Upfront** availability first (`asc subscriptions pricing monthly-commitment`).
4. **Review screenshot** — `asc subscriptions review screenshots create --subscription-id <SUB_ID> --file <paywall.png>`. Use the paywall screenshot.
5. **⚠️ Subscription GROUP localization** — the single most common cause of a stuck "Missing Metadata". Even with all four above, the **group** needs a display-name localization:
   ```
   asc subscriptions groups versions localizations create --version-id <GROUP_VERSION_ID> --locale en-US --name "Premium"
   ```
   Adding this flips both subs to "Ready To Submit".

At submission time the **first subscription must be included with the app version** — select the subs in the version page's "In-App Purchases and Subscriptions" section (only selectable once "Ready To Submit").

---

## Phase 6 — Declarations (mostly CLI now; one web-only holdout)

Everything below except the privacy nutrition label is a first-class `asc` command:

- **Age Rating** — `asc age-rating edit --app <APP_ID> --all-none` sets every field to its safe default (a finance/tracker with no objectionable content → **4+**). Override individual fields as needed; `asc age-rating view --app <APP_ID>` to confirm. (Apple's 2026 questionnaire also asks about **social media / messaging** capabilities — a tracker with only support messaging answers `--messaging-and-chat false --social-media false --user-generated-content false`.)
- **Primary Language** — `asc app-setup info set --app <APP_ID> --primary-locale <loc>` (the locale must have a complete localization first).
- **Content Rights** — `asc app-setup info set --app <APP_ID> --content-rights USES_THIRD_PARTY_CONTENT` (a price/market-data app that shows provider data) or `DOES_NOT_USE_THIRD_PARTY_CONTENT`.
- **Copyright** — `asc versions update --version-id <VERSION_ID> --copyright "2026 Your Name"` (it lives on the version, not App Information).
- **Price tier** — `asc app-setup pricing set --app <APP_ID> --free` (freemium apps monetize via IAP). Availability at 175 countries via `asc pricing availability`.
- **App Review Information** — `asc review details-create --version-id <VERSION_ID> --contact-first-name ... --contact-last-name ... --contact-phone ... --contact-email ... --notes "..."`. Add `--demo-account-required=true --demo-account-name ... --demo-account-password ...` when review needs credentials. (The old CLI could not set contact first/last/phone; `asc` can.)
- **Encryption** — if the build has `ITSAppUsesNonExemptEncryption=false` in Info.plist, no ASC declaration is needed (`asc encryption` handles it otherwise).
- **Sign in with Apple** — required (4.8) if the app offers any third-party/social login (e.g. Google). App-side; must be present in the build.
- **Demo account** — create one via the backend signup API (no manual account needed); confirm it can sign in (email-verification must not block it).

Still web-UI-only (not in Apple's public API):

- **App Privacy nutrition label** — data-collection questionnaire. Use `asc web privacy` (web-session helper) or the ASC web UI. Answer from the real privacy policy: pick data types (Contact Info, Financial Info, User Content, Identifiers, Purchases, Diagnostics...), set purpose (**App Functionality**), linked-to-identity (**Yes** for account apps), tracking (**No** if no ads). Then **Publish**. Note: first-party product analytics that collects usage/diagnostics data must be declared here.

---

## Phase 7 — Support & Privacy URLs

- **Support URL** is a version-localization field; `asc localizations update` writes it directly:

  ```
  asc localizations update --version <VERSION_ID> --locale en-US --support-url "https://site.com/support"
  ```

  Locale must match the exact ASC form (`en-US`, `ar-SA`, `zh-Hans` — run `asc localizations supported-locales --version <VERSION_ID>` if one is rejected).

- Privacy Policy URL lives in **app-info** (`asc app-setup info set --primary-locale <loc> --privacy-policy-url ...`), Support URL in **version localizations**. Both must be **live** during review.
- Both URLs must be reachable in a real browser. Server-side/sandbox `curl`/`fetch` may get **522** from Cloudflare bot-blocking even when the site is fine in a browser — verify from an actual browser, not a headless fetch.

---

## Phase 8 — "Add for Review" checklist

`asc review doctor --app <APP_ID>` surfaces the definitive blocker list. Common ones and where to fix:
| Blocker | Fix |
| --- | --- |
| Support URL required (all locales) | `asc localizations update --support-url` (Phase 7) |
| Contact Information | `asc review details-create` (Phase 6) |
| Copyright | `asc versions update --copyright` (Phase 6) |
| Price tier | `asc app-setup pricing set --free` (Phase 6) |
| Content Rights | `asc app-setup info set --content-rights` (Phase 6) |
| Age Rating | `asc age-rating edit --all-none` (Phase 6) |
| IAP "Missing Metadata" | group localization (Phase 5) |

Then submit with `asc review submit` (attaches the build, creates the submission, adds items, and submits in one wrapper) — do this last, after `asc review doctor` is clean apart from the 1.0 "What's New" false positive. `asc review status --app <APP_ID>` tracks state afterward.

---

## Legal-pages web note (Astro)

If the marketing/legal site (privacy/terms/support pages) 522s or 301-loops on trailing slashes: set Astro `trailingSlash: 'never'` + `build: { format: 'file' }` (flat `.html` files) and an nginx `try_files $uri $uri.html ...` with a `^/(.+)/$ → /$1` 301. Make link helpers emit no-slash paths. Keep per-page canonical/hreflang (SEO) but point the language switcher at each locale's homepage (`/`, `/tr`, `/es`), not the translated current path, if that's the desired UX.

---

## Time-savers next time

1. `asc auth login` and confirm the RevenueCat product IDs before anything else.
2. Do screenshots + ASO + IAP config in parallel background jobs; verify every screenshot (RTL/slow-boot silently fail).
3. Expect the **subscription group localization** trap (Phase 5) — the top cause of "why is it still not ready" loops. The old support-url / web-only traps are mostly gone now that `asc` covers those fields.
4. Keep the demo account and backend live from submission through review.
