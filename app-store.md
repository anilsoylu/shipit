# App Store Review Playbook

Goal: reduce first-review rejection risk before submitting Expo apps.

## Review Gate

Do not submit until every item is true:

- App has no known crashes or blocking bugs.
- Backend services are live and reachable.
- App metadata is complete and accurate.
- Screenshots and previews match the submitted build.
- In-app purchases, subscriptions, prices, and descriptions are accurate.
- Login works, and Sign in with Apple is present when required.
- Privacy policy URL is live.
- App privacy answers match actual SDKs and data collection.
- Demo account or fully featured demo mode is provided if the app needs login.
- Review notes explain non-obvious features, required hardware, test data, purchases, and restricted flows.
- A screen-recorded demo video is attached to the review.
- No screen is unfinished, empty, or labelled "coming soon".
- User-generated content apps have report, block, a support contact, and automated moderation.
- Regional compliance issues are handled for the target countries.

## Apple Review Notes

Include:

- Demo email and password, or demo mode instructions.
- Paid feature test instructions.
- Subscription/IAP explanation if present.
- Required account state, sample data, QR codes, invite codes, or admin setup.
- Backend availability note.
- Contact email.

Do not make reviewers discover hidden setup steps.

Write them for someone who has never seen the app:

- Short bullet points, numbered steps for each flow to test.
- Plain sentences. Generated copy reads dense and slows the reviewer down.
- One block per area: subscription testing, main flow, social features, permissions, account and safety controls.
- Give the reviewer prepared state instead of setup work — a dedicated account that auto-accepts friend requests, a fixed invite code, seeded data.

Attach a demo video as well. Screen record the main flow and add a text overlay per section. This is the highest-return item in the whole submission: it answers the questions that otherwise become a rejection or an escalation.

The goal is a boring review.

## Review Speed and Triage

Reviews are sorted into tiers, and the tier decides the wait:

1. Straightforward review → first pile, usually within 48 hours.
2. Open questions or complexity → escalated to a senior specialist, where the multi-week waits happen.

Stay in the first pile:

- Submit an MVP. The first review of an app is the slowest one; every extra feature adds review surface.
- Add features in later updates, when the app already has an approval history.
- Accept tier 2 only when the product genuinely is complex (banking, health or biometric data, marketplaces).

When you need it faster:

- Request an expedited review for a critical bug fix, a launch event, or a marketing date. No justification field is required.
- Request a phone call from App Review. Almost nobody does, and it appears to be a separate queue.
- After a rejection, ask for **all** outstanding reasons at once. Otherwise a fix ships, a second unrelated reason arrives, and each round costs another cycle. A call is the fastest way out of that loop.

## Hold Until a Later Update

These are allowed but reviewer-dependent. Shipping them in a first submission risks the escalation tier for no gain:

- Transaction abandon offers (a discount shown when the user dismisses the paywall).
- Exit offers on the in-app "manage subscription" flow — a questionnaire plus a cheaper plan, instead of a straight deep link to the App Store subscription page.

Ship them in a later update, explain them in the review notes, and include a demo of the feature. Never make a reviewer discover a monetization flow that was not described.

## Metadata Gate

Check before review:

- App name, subtitle, description, keywords, category, age rating, support URL, marketing URL, and privacy URL.
- Copyright and contact information.
- What's New text for updates.
- No claims that the app does not actually support.
- No placeholder screenshots, lorem ipsum, staging labels, debug UI, or internal URLs.
- **Auto-renewable subscriptions: the description must carry a working Terms of Use (EULA) link, in every locale.** An in-app link on the paywall does not satisfy this; the scan only reads metadata. Either link your own terms page or Apple's standard EULA (https://www.apple.com/legal/internet-services/itunes/dev/stdeula/). Open each URL and confirm it returns 200 before submitting.

## Screenshot Gate

Screenshots must:

- Show the current app version.
- Show real product value, not generic phone mockups.
- Put the strongest value proposition in screenshot 1.
- Use readable text at App Store thumbnail size.
- Avoid tiny UI details that only make sense full-screen.
- Match actual device UI and localization.
- Avoid showing unavailable paid states as free.

Tooling:

- Use `app-store-screenshots` when screenshot production is in scope.
- Capture starting screenshots on a 6.1 inch simulator when using that workflow.

## Packaging Rules

Before trying to scale revenue, package the app like a product. If an app is stuck around low MRR, assume packaging is weak before assuming traffic is weak.

Use Paul Solt and Viktor Seraleev's 5-part packaging model:

1. Icon
   - Bold, clear, readable at App Store thumbnail size.
   - One clear subject.
   - Strong color contrast.
   - Must communicate the app idea before the user reads copy.
   - Do not get attached to the first icon. Test variations.

2. Screenshots
   - Stylize real app screens.
   - Use large readable text.
   - Tell a story across the screenshot sequence.
   - The first 3 screenshots matter most because they appear in iOS search results.
   - Do not make screenshots a tutorial. Make users understand the outcome before tapping Get.

3. Onboarding
   - Keep onboarding to 3 or 4 screens.
   - Show the outcome, not a tutorial.
   - Use one action per step.
   - Reinforce App Store screenshot messaging.
   - Get the user to the aha moment fast.
   - Test video onboarding when the product benefits from motion.

4. Paywall
   - If monetization is core, show the paywall early enough to be seen.
   - A paywall only 1% of users see is usually useless.
   - Place it after onboarding or the first value moment.
   - Use clear offer, clear value, and easy pricing math.
   - Avoid hidden close buttons, fake urgency, and price tricks.
   - Test 3-day, 7-day, and 30-day trials.
   - For recurring revenue, prefer weekly and annual subscriptions over lifetime deals when the market supports it.

5. Simple user flow
   - One screen, one action, clear path.
   - Every extra tap is a chance to lose the user.
   - Remove choices that make users stop and think.
   - Make the first successful outcome obvious.

## Product Page Optimization

When the app has traffic:

- Test app icon, screenshots, and preview videos with product page optimization.
- Test one hypothesis at a time.
- Promote the treatment only after it beats the original.
- Use custom product pages for different audiences or campaigns when useful.

## Common Rejection Traps

- Crashes on launch or login.
- Demo account missing or broken.
- Backend turned off during review.
- Metadata claims unsupported features.
- Screenshots show old UI.
- IAP products missing, inaccurate, or not reviewable.
- Privacy policy missing.
- **Guideline 3.1.2 — no Terms of Use (EULA) link in the description of a subscription app.** This one is caught by an automated metadata scan, so it burns the whole review slot before a human ever opens the build. Compliant in-app paywall copy does not help.
- Legal URLs in the metadata that redirect to a dead endpoint. Apple requires a *functional* link, and a marketing site behind a proxy can 301 into an internal `http://host:8080/...` origin. Test the exact string you pasted, following redirects.
- Data collection answers do not match SDK behavior.
- Stripe used for mobile-consumed digital goods where IAP is required.
- Sign in with Apple missing when the app offers third-party social login and Apple requires it.
- A remote paywall changed to a non-compliant state after review passed.
- Review prompt fired during onboarding.
- Fake social proof ("#1 on the App Store") or absolute claims ("fixes 100% of X") in metadata or UI.
- A 1:1 clone of an existing app. Shared patterns are fine; a copy is not.
- User-generated content with no report, block, or moderation path.

Review outcomes are subjective by design, so the same build can pass with one reviewer and fail with another. Treat the list above as reducing the odds, not as a guarantee. If a rejection is wrong, challenge it by citing the specific guideline and the evidence in the build — complaining does not move it.

## Official References

- App Review Guidelines: https://developer.apple.com/app-store/review/guidelines/
- App information: https://developer.apple.com/help/app-store-connect/reference/app-information/
- Screenshots and previews: https://developer.apple.com/help/app-store-connect/manage-app-information/upload-app-previews-and-screenshots/
- App privacy: https://developer.apple.com/help/app-store-connect/manage-app-information/manage-app-privacy/
- Product page: https://developer.apple.com/app-store/product-page/
- Product page optimization: https://developer.apple.com/app-store/product-page-optimization/
- App Store screenshots generator: https://github.com/ParthJadhav/app-store-screenshots
- Paul Solt packaging post: https://x.com/PaulSolt/status/2045580498232373433
- Viktor Seraleev MRR story: https://x.com/seraleev/status/2044408846735507902?s=20
- Viktor experiments link: https://super-easy-apps.kit.com/12-app-store-experiments
- Paul Solt screenshot article: https://x.com/PaulSolt/status/2037591726227902555
- Viktor deeper packaging post: https://x.com/seraleev/status/2010023256984879170?s=20
- Frederick James on 48-hour approvals (review tiers, notes, expediting): https://x.com/frederickjames/status/2086419927548809285
