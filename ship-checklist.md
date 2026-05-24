# Ship Checklist

Use this before calling an Expo app ready for testing or release.

## Before Coding

- App idea is known.
- MVP scope is 3 to 5 screens.
- Core loop is written in one sentence.
- Monetization decision is explicit or deferred.
- Required integrations are listed.
- Default stack overrides are listed, if any.
- Required skills from `skills.md` are available or installed.

## Repo

- Monorepo structure exists.
- `apps/mobile` starts.
- `apps/api` starts.
- Shared types do not import server-only code.
- `.env.example` files exist.
- No real secrets are committed.

## Mobile

- Expo Router routes match the intended screens.
- NativeWind works.
- UI has loading, empty, error, and success states.
- Auth gate works if auth is in scope.
- API base URL is environment-driven.
- No server secrets are present in mobile env or code.

## API

- `/health` works.
- Protected routes reject unauthenticated calls.
- Authenticated calls derive user from Clerk verification.
- Request bodies are validated.
- Drizzle migrations exist for schema changes.
- Redis is used only where justified.
- Webhook routes verify signatures.

## Payments

- Payment path matches `payments.md`.
- Stripe secret operations are API-only.
- RevenueCat/IAP is used for mobile digital entitlements when required.
- Webhooks update server-side entitlements or orders.
- Mobile refreshes entitlement/order state from API.
- RevenueCat MCP is used or explicitly skipped when RevenueCat is in scope.

## AI Features

- Provider keys are API-only.
- Endpoint has auth.
- Endpoint has input limits.
- Endpoint has Redis rate limit or quota before public launch.
- Output is typed and product-specific.

## Observability

- Sentry initialized where in scope.
- PostHog initialized where in scope.
- Core loop events are captured.
- Errors do not log secrets or full tokens.

## Build

- Typecheck passes.
- Mobile local start works.
- API local start works.
- Native module apps use Expo development builds.
- EAS profiles exist before external testing.
- Preview/internal build is smoke tested on device.

## Release

- App name, icon, splash, scheme, and bundle identifiers are configured.
- Store screenshots and privacy answers are prepared when submitting.
- EAS Submit is used for store upload when ready.
- EAS Update is used only for JS-only changes that comply with store policy.

## App Store Review

- No crashes or blocking bugs.
- Backend services are live for review.
- Metadata is complete and accurate.
- Screenshots and previews match the submitted build.
- IAP/subscription descriptions are correct.
- Sign in with Apple is added when required.
- Privacy policy URL is live.
- App privacy answers match actual SDK behavior.
- Demo account or demo mode is provided if login is required.
- Review notes explain non-obvious flows.

## ASO

- Title, subtitle, keywords, description, and What's New are final.
- Screenshot copy is outcome-focused.
- Screenshot 1 communicates the strongest value proposition.
- First 3 screenshots tell a value story visible from search results.
- Icon is not generic.
- Icon is readable at 60x60 points and stands out against competitors.
- Onboarding reaches value quickly.
- Onboarding is 3 to 4 screens and not a tutorial.
- First user flow is one screen, one action, clear path.
- Paywall placement is deliberate if monetization is in scope.
- Paywall appears after onboarding or the first value moment, not in a hidden late path.
- Pricing comparison is simple if subscriptions are in scope.
- Product page optimization is planned once traffic exists.

## Official References

- Expo app config: https://docs.expo.dev/workflow/configuration/
- EAS Build: https://docs.expo.dev/build/introduction/
- EAS Submit: https://docs.expo.dev/submit/introduction/
- EAS Update: https://docs.expo.dev/eas-update/introduction/
- Sentry Expo: https://docs.expo.dev/guides/using-sentry/
- PostHog React Native: https://posthog.com/docs/libraries/react-native
