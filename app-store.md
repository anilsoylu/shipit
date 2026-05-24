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

## Metadata Gate

Check before review:

- App name, subtitle, description, keywords, category, age rating, support URL, marketing URL, and privacy URL.
- Copyright and contact information.
- What's New text for updates.
- No claims that the app does not actually support.
- No placeholder screenshots, lorem ipsum, staging labels, debug UI, or internal URLs.

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
- Data collection answers do not match SDK behavior.
- Stripe used for mobile-consumed digital goods where IAP is required.
- Sign in with Apple missing when the app offers third-party social login and Apple requires it.

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
