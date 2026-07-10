# iOS "Plans are loading…" / empty RevenueCat offering — diagnosis & fix playbook

> An Expo/React Native + RevenueCat app shows "Plans are loading" / `offering.availablePackages` is empty; the products exist in App Store Connect but StoreKit returns nothing. Hand this doc to a Claude CLI to fix the same issue on another project. **Order matters: the fastest and most likely cause comes first.**

## Tools you need

- **RevenueCat MCP** (or the dashboard) — verify offering/app/public key
- **`ascelerate` CLI** + App Store Connect API key (`~/.ascelerate/*.p8`, Key ID, Issuer ID) — ASC operations
- **`curl`** — test "what the app actually receives" via the RevenueCat REST API (no rebuild needed)
- **Direct JWT to the ASC API** (ES256, signed with the .p8) — for things the CLI can't do (e.g. creating app availability)

---

## Check in this order

### 0) Most important — is the Paid Applications Agreement "Active"? (30 seconds, cause #1)

App Store Connect → top right → **Business** → Agreements, Tax, and Banking.

- **If the "Paid Apps" row is NOT "Active"** ("Pending User Info" / banking missing) → **StoreKit returns no products in any environment (Sandbox, TestFlight, Production).** The paywall stays empty even with a flawless RevenueCat + ASC config.
- **Fix:** Add a bank account (+ tax forms + accept the agreement). Activation takes up to **24 hours**; the products then propagate to StoreKit and the paywall **fills on its own** (no rebuild/submit — clear the RevenueCat cache by deleting and reinstalling the app).
- **Region-specific banking notes (Turkey example):**
  - The "Bank Code / Branch Code" field is a **search box** — don't type a combined code by hand (e.g. `00151068`); use the **"Don't know your Bank Code / Branch Code?"** link to search the branch by city/name and select it.
  - Enter the **IBAN with NO SPACES** (a trailing space gives "IBAN invalid").
  - **Account Number = the account portion of the IBAN** (not the Customer Number). Example: IBAN `TR00 00015 0 XXXXXXXXXXXXXXXX` → Account Number = the last 16 digits `XXXXXXXXXXXXXXXX` (excluding the bank code `00015` and the check digits).
  - Name/address fields reject non-Latin characters → transliterate (İzmir→Izmir, Şişli→Sisli, ç/ş/ı/ö/ü/ğ → c/s/i/o/u/g).
- ⚠️ **The "my other app works" trap:** that app may be showing a **hardcoded fallback price** on the paywall (code pattern: a `PRICE` constant + `pkg ? pkg.product.priceString : fallback`), not the real StoreKit price. Confirm with the RevenueCat REST API (step 2). Don't let another app on the same account "working" mislead you.

### 1) Does the build have the LIVE key? (`appl_`, not `test_`)

- `apps/mobile/eas.json` (the `env` of the build profiles) + `.env` → `EXPO_PUBLIC_REVENUECAT_IOS_KEY` must be **`appl_...`**.
- Typical code **skips** the `test_` (RevenueCat Test Store) key in a release build (to avoid the SDK's "Wrong API Key" force-close) → RevenueCat never configures → `getOfferings` throws → permanent "Plans are loading".
- If the build under test was made BEFORE the key was fixed, you need a new build.

### 2) What is RevenueCat serving the app? (a definitive test, no rebuild)

```bash
curl -s "https://api.revenuecat.com/v1/subscribers/diag-1/offerings" \
  -H "Authorization: Bearer appl_XXXXXXXX" \
  -H "X-Platform: ios"
```

- `current_offering_id` + `packages[]` should come back **populated** (with the product IDs).
- **Empty (`[]`)** → no packages are attached to the offering in RevenueCat → in the RC dashboard/MCP, **attach** the products to packages and to the current offering.
- **Populated but empty in the app** → RevenueCat is fine; the problem is on the **StoreKit fetch** side (steps 0 and 3).

### 3) Are the ASC products "Ready to Submit"? (not "Missing Metadata")

```bash
ascelerate sub list <bundleId>
```

Every subscription must be **"Ready to Submit"**. If it's **"Missing Metadata"**, these 5 things are required:

1. **Price** — `ascelerate sub pricing set` returns **HTTP 409** on new subs (a known Apple restriction). Set the first price from the **ASC web UI** (USD base → Apple auto-equalizes to 175 territories).
2. **Localization** — `ascelerate sub localizations import <bundle> <pid> --file f.json` → `{"en-US":{"name":"...","description":"..."}}`
3. **Group localization** — `ascelerate sub group-localizations import <bundle> --file g.json` → `{"en-US":{"name":"..."}}` **(most often forgotten / the most common cause of "Missing Metadata")**
4. **Review screenshot** — `ascelerate sub review-screenshot upload <bundle> <pid> shot.png` (produce a 1242×2208 image; rendering HTML with headless Chrome is easiest)
5. **Availability** — if the app has no territory availability (ASC API `GET .../appAvailabilities` → 404), `ascelerate apps availability` **can't create it** (it only edits an existing one). Create it with ASC API v2:
   ```
   POST https://api.appstoreconnect.apple.com/v2/appAvailabilities
   ```
   Send `territoryAvailabilities` with all territories `available:true` + `${local-id}` format for the inline create (fetch v1/territories and loop to build the list).

### 4) Product ID / bundle match

- Build bundle == ASC app bundle == RevenueCat app bundle.
- RevenueCat `store_identifier` == ASC product ID (exact). A trailing digit added later like "premium_monthly**1**" is the classic mismatch trap.

---

## Outcome on this project

Steps 1–3 were all green (RevenueCat offering correct, key correct, products Ready to Submit, price/localization/group-localization/availability/review-screenshot done). But the **real blocker was step 0: the Paid Applications Agreement was "Pending User Info"** (banking missing). Once a bank account was added and the agreement went **Active**, StoreKit returned the products and the paywall filled **without a rebuild/submit**.

**Lesson:** Check the **Agreement/Banking (step 0)** first, then the config chain. "Ready to Submit" + a correct RevenueCat config are worthless if the agreement isn't active.

---

## Quick command summary

```bash
# What is RevenueCat serving the app?
curl -s "https://api.revenuecat.com/v1/subscribers/diag/offerings" \
  -H "Authorization: Bearer appl_XXXX" -H "X-Platform: ios"

# ASC sub statuses
ascelerate sub list <bundleId>
ascelerate sub info <bundleId> <productId>

# Fill the gaps
ascelerate sub localizations import <bundleId> <pid> --file sub.json -y
ascelerate sub group-localizations import <bundleId> --file group.json -y
ascelerate sub review-screenshot upload <bundleId> <pid> shot.png -y
# price: ASC web UI (the CLI returns 409)
# no availability: ASC API v2 POST /v2/appAvailabilities

# Agreement/Banking (step 0): ASC web → Business → Agreements, Tax, and Banking → "Paid Apps" must be "Active"
```
