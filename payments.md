# Payments Playbook

Pick the payment path from the product, not developer preference. Default for
in-app digital goods is RevenueCat (IAP); default for web/physical is Stripe.
Payments are pluggable — whatever you pick must satisfy the swap contract below.

## Choosing a payment path

Use RevenueCat/IAP when:

- The mobile app sells digital subscriptions.
- The mobile app sells digital content consumed in the app.
- The app unlocks premium app features for mobile users.
- Store rules require in-app purchase.

Use Stripe when:

- The app sells physical goods.
- The app sells offline services, appointments, bookings, deposits, or reservations.
- The product is B2B SaaS primarily accessed outside the mobile app.
- The payment happens on web and mobile only reflects account status.
- You need invoices, saved cards, customer portal, Connect, or custom billing.

Use both only when:

- Mobile digital entitlements need IAP/RevenueCat.
- Web or service payments need Stripe.
- The backend unifies entitlements and avoids double-granting.

If uncertain, do not implement payment first. Build the core loop, then add the smallest compliant payment path.

## Payments Swap Contract

Every payment integration must satisfy these:

1. In-app digital goods/subscriptions consumed inside the app MUST use the
   platform IAP (Apple/Google) — via RevenueCat or native billing. Never route
   these through Stripe/web to avoid the store fee.
2. Entitlements are server-authoritative. The server verifies receipts/webhooks;
   the mobile client never sends a trusted price, amount, currency, entitlement,
   plan, or role.
3. Webhooks verify the provider signature before mutating entitlement/order
   state; store the raw body when signature verification needs it.
4. Product/price IDs come from server config or DB, not mobile input.
5. Secrets (provider secret keys, webhook secrets) live in server env only;
   only publishable/public keys ship in the Expo bundle.
6. If backend access is gated by entitlement, the check is server-side.

Additional store-review guardrails (hold at all times):

- Do not unlock app-consumed digital features from an external payment unless store rules allow it.
- Do not ship hidden payment flows.

## Add a payment provider

Copy this skeleton into a provider section/playbook and fill it in:

```markdown
# Payment Playbook: <Provider>

Use <Provider> when: <selection criteria>.

## Client (Expo)
- SDK/package, publishable key, purchase/checkout flow.

## Server verification
- How the API creates payment objects and verifies receipts/webhooks.

## Webhooks
- Events to handle and how the signature is verified.

## Env vars
- Public (bundle-safe) vs secret (server only).

## Contract compliance
- Restate the 6 swap-contract invariants; confirm each is met.

## Official References
- Provider docs, llms.txt, and skill (from the LLM Docs Registry).
```

## Provider playbooks

### RevenueCat (default IAP)

Use RevenueCat for mobile digital entitlements:

- Configure products in App Store Connect and Google Play.
- Configure offerings and entitlements in RevenueCat.
- Mobile SDK fetches offerings and purchases.
- Backend or webhook syncs entitlements when server authorization needs them.
- Do not invent custom subscription state in mobile storage.

Use RevenueCat MCP when available to manage projects, apps, products, offerings, packages, paywalls, analytics, and experiments without manual dashboard drift.

Contract compliance: IAP via StoreKit/Play Billing (1); entitlements synced
server-side via webhook (2, 6); RevenueCat webhook signature verified before
mutating (3); product IDs from RevenueCat/store config (4); only the public SDK
key ships in the app (5). ✓

References: skill `revenuecat/ai-toolkit`; llms.txt https://www.revenuecat.com/docs/llms.txt

### Stripe (default web)

Stripe must be server-driven:

- API creates customers, ephemeral keys, payment intents, setup intents, checkout sessions, subscriptions, and portal sessions.
- Expo app uses publishable key plus server-created client secrets only.
- Webhooks verify Stripe signature before changing DB state.
- Prices and product IDs come from server config or DB, not mobile input.
- Mobile never sends final amount, currency, entitlement, or role as trusted data.

Typical flow:

1. Mobile asks API to start checkout.
2. API verifies the authenticated user (see `auth.md`).
3. API creates Stripe object.
4. Mobile presents Stripe UI or opens checkout.
5. Stripe sends webhook.
6. API verifies webhook and updates entitlement/order.
7. Mobile refreshes server entitlement.

Contract compliance: web/physical only, never in-app digital (1);
server-created intents, no trusted client amounts (2, 4); webhook signature
verified before DB change (3); secret key server-only, publishable in app (5);
entitlement checks server-side (6). ✓

References: skill `stripe/ai`; llms.txt https://docs.stripe.com/llms.txt

### Situational providers

Short list — each satisfies the same contract; use only when its niche fits.
Load its llms.txt before implementing (no full playbook here):

- **Adapty** — RevenueCat alternative with heavy paywall A/B testing. llms.txt: https://adapty.io/docs/llms.txt
- **Superwall** — remote-configurable paywall design + A/B testing (skill `superwall/skills`). llms.txt: https://superwall.com/docs/llms.txt
- **Paddle** — merchant-of-record for web/desktop SaaS (owns global tax/VAT). llms.txt: https://developer.paddle.com/llms.txt
- **Polar** — developer-first open-source merchant-of-record. llms.txt: https://polar.sh/docs/llms.txt
- **Lemon Squeezy** — simple MoR checkout for indie/web SaaS and license keys. Check https://docs.lemonsqueezy.com for docs.

## Official References

- Stripe React Native SDK: https://docs.stripe.com/sdks/react-native
- Stripe webhooks: https://docs.stripe.com/webhooks
- RevenueCat React Native: https://www.revenuecat.com/docs/getting-started/installation/reactnative
- RevenueCat MCP: https://www.revenuecat.com/docs/tools/mcp
- Expo development builds: https://docs.expo.dev/develop/development-builds/introduction/
