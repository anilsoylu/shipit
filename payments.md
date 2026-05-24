# Payments Playbook

Pick the payment path from the product, not developer preference.

## Decision Tree

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

## Stripe Pattern

Stripe must be server-driven:

- API creates customers, ephemeral keys, payment intents, setup intents, checkout sessions, subscriptions, and portal sessions.
- Expo app uses publishable key plus server-created client secrets only.
- Webhooks verify Stripe signature before changing DB state.
- Prices and product IDs come from server config or DB, not mobile input.
- Mobile never sends final amount, currency, entitlement, or role as trusted data.

Typical flow:

1. Mobile asks API to start checkout.
2. API verifies Clerk user.
3. API creates Stripe object.
4. Mobile presents Stripe UI or opens checkout.
5. Stripe sends webhook.
6. API verifies webhook and updates entitlement/order.
7. Mobile refreshes server entitlement.

## RevenueCat Pattern

Use RevenueCat for mobile digital entitlements:

- Configure products in App Store Connect and Google Play.
- Configure offerings and entitlements in RevenueCat.
- Mobile SDK fetches offerings and purchases.
- Backend or webhook syncs entitlements when server authorization needs them.
- Do not invent custom subscription state in mobile storage.

Use RevenueCat MCP when available to manage projects, apps, products, offerings, packages, paywalls, analytics, and experiments without manual dashboard drift.

## Store Review Guardrails

- Do not route mobile digital purchases through Stripe to avoid IAP.
- Do not unlock app-consumed digital features from an external payment unless store rules allow it.
- Do not ship hidden payment flows.
- Keep entitlement checks server-authoritative when backend access is gated.

## Official References

- Stripe React Native SDK: https://docs.stripe.com/sdks/react-native
- Stripe webhooks: https://docs.stripe.com/webhooks
- RevenueCat React Native: https://www.revenuecat.com/docs/getting-started/installation/reactnative
- RevenueCat MCP: https://www.revenuecat.com/docs/tools/mcp
- Expo development builds: https://docs.expo.dev/develop/development-builds/introduction/
