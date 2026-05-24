# ASO Playbook

Goal: make App Store packaging a build artifact, not an afterthought.

## When To Use

Use this file when:

- Preparing first launch.
- Updating metadata.
- Producing screenshots.
- Testing icons, screenshots, subtitles, or keywords.
- Localizing store listings.
- Trying to improve download conversion.

## ASO Workflow

1. Pull current metadata if the app already exists.
2. Identify competitors and review their positioning.
3. Extract user language from reviews, not only founder assumptions.
4. Build keyword candidates per locale.
5. Write title, subtitle, keywords, promo text, description, and What's New.
6. Validate character limits before pushing.
7. Produce screenshot copy before designing screenshots.
8. Capture real app screens.
9. Compose App Store screenshots.
10. Run review readiness checks.

## Claude ASO Toolkit

Use `claude-aso` when Claude Code is available and ASO work is in scope.

Useful commands:

- `/aso init`: detect project and generate config.
- `/aso pull`: pull current App Store metadata.
- `/aso research`: guided keyword research.
- `/aso competitor`: competitor keyword analysis.
- `/aso metadata`: generate metadata with character limit validation.
- `/aso screenshots capture`: capture screenshots with iOS Simulator.
- `/aso screenshots compose`: generate marketing screenshots.
- `/aso screenshots upload`: upload screenshots.
- `/aso icons generate`: generate icon candidates.
- `/aso score`: ASO readiness score.
- `/aso check`: App Store Review compliance checks.
- `/aso privacy-manifest`: generate privacy manifest.

Reference: https://github.com/omerycll/claude-aso

## Screenshot Production

Use `app-store-screenshots` when the project needs generated App Store and Google Play screenshots.

Rules:

- Start from real app screens.
- Use a 6.1 inch simulator for source captures in that workflow.
- Generate screenshots after the UI is stable.
- Re-run screenshots after visual changes that affect the submitted build.
- Keep screenshot text short and outcome-focused.
- Treat the first 3 screenshots as the search-result sales pitch.
- Screenshot 1 must communicate the strongest value.
- Screenshot 2 should tease the workflow or picker/tool.
- Screenshot 3 should prove the editor/result/output.
- Use recognizable examples when the app category benefits from them.

Reference: https://github.com/ParthJadhav/app-store-screenshots

## Copy Rules

- Screenshot 1 sells the core outcome.
- Screenshot 2 proves the workflow.
- Screenshot 3 shows trust, accuracy, or saved time.
- Screenshot 4 shows the premium or habit loop if relevant.
- Screenshot 5 handles edge value: widgets, AI, history, reports, sharing, or automation.
- Never describe buttons. Describe the result.
- Do not write "all-in-one", "next-gen", "seamless", or vague benefit soup.
- Do not explain UI controls. Explain the result.
- Do not write a tutorial sequence unless the store visitor cannot understand the app without it.

## Packaging Audit

Run this before first launch and before any monetization push:

- Icon: can a user understand the app at 60x60 points?
- Icon: does it stand out beside the top 20 competing search results?
- Screenshots: do the first 3 screenshots tell a complete value story?
- Screenshots: is all text readable at thumbnail size?
- Onboarding: is it 3 to 4 screens with one action per step?
- Onboarding: does it repeat the same promise from the App Store screenshots?
- Paywall: does enough traffic see it to matter?
- Paywall: is it placed after onboarding or after the first value moment?
- Paywall: are weekly and annual prices easy to compare?
- Flow: is the first path one screen, one action, clear next step?

## Metadata Rules

- Title: primary searchable phrase plus brand when possible.
- Subtitle: direct outcome or differentiator.
- Keywords: avoid repeating words already in title/subtitle when platform rules make that wasteful.
- Description: first lines must explain the concrete value before feature lists.
- What's New: state real user-facing changes, not internal refactors.

## Locales

Default to English first.

Add locales only when:

- The app has target traffic in that market.
- Screenshots can be localized.
- Support/privacy text can handle that market.
- Keywords differ meaningfully from English.

## Official References

- App Store product page: https://developer.apple.com/app-store/product-page/
- Product page optimization: https://developer.apple.com/app-store/product-page-optimization/
- App information: https://developer.apple.com/help/app-store-connect/reference/app-information/
- Screenshots and previews: https://developer.apple.com/help/app-store-connect/manage-app-information/upload-app-previews-and-screenshots/
