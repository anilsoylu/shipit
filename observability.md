# Observability Playbook

Default crash/error is Sentry; default product analytics is PostHog.
Observability is pluggable — whatever you pick must satisfy the Observability
Swap Contract below.

## Choosing crash + analytics

- **Crashes/errors** → **Sentry** (default). Full-stack across mobile, web, and API (Expo and Next.js SDKs).
- **Product analytics** → **PostHog** (default). Events, funnels, flags, replay; RN and web/Next.js SDKs.
- In the Google ecosystem → **Firebase** (Crashlytics + Analytics).
- Deep behavioural analytics → **Amplitude** or **Mixpanel**.
- Privacy-first / open-source, runs in Expo Go → **Aptabase** or **OpenPanel**.
- Backend APM/logs → **Datadog**, **Axiom**, **Better Stack**, or vendor-neutral
  **OpenTelemetry**.

## Observability Swap Contract

Every crash/analytics integration must satisfy these:

1. Never send PII, tokens, secrets, or full auth headers in analytics/crash
   events; scrub before sending.
2. Initialize crash + analytics before first render so early errors are caught.
3. Upload source maps / symbolication artifacts so stack traces are readable.
4. Honor analytics opt-out / consent where required (GDPR/ATT).
5. Backend observability keys live in server env only.

## Add a provider

Copy this skeleton into a provider section and fill it in:

```markdown
# Observability Playbook: <Provider>

Use <Provider> when: <selection criteria>.

## Client / server setup
- SDK/package, init point (before first render), public vs secret key.

## Data hygiene
- What is scrubbed; source-map/symbolication upload.

## Contract compliance
- Restate the 5 swap-contract invariants; confirm each is met.

## Official References
- Provider docs, llms.txt, and skill (from the LLM Docs Registry).
```

## Playbooks

### Sentry (default crash)

- Add the Sentry Expo SDK; init before first render.
- Upload source maps on EAS build so stack traces are readable.
- Wrap app/API failures; scrub PII/tokens before send.
- Backend DSN in the app is fine (public); server keys stay server-side.
- Contract compliance: scrub (1), early init (2), source maps (3), consent (4),
  server keys server-only (5). ✓
- Skill `getsentry/skills`; llms.txt https://docs.sentry.io/llms.txt

### PostHog (default analytics)

- Add the PostHog RN SDK; init before first render.
- Send the core-loop events only; never PII/tokens.
- Use feature flags / A/B where product needs them.
- Honor opt-out/consent (GDPR/ATT).
- Contract compliance: scrub (1), early init (2), n/a source maps (3), consent
  (4), server key server-only (5). ✓
- Skill `posthog/skills`; llms.txt https://posthog.com/llms.txt

### Alternates

Short stubs — load the llms.txt before implementing:

- **Firebase (Analytics + Crashlytics)** — Google ecosystem; battle-tested native
  crash reporting + free analytics. Skill `firebase/agent-skills`; no llms.txt —
  check https://firebase.google.com/docs.
- **Amplitude** — deep behavioural analytics, funnels, retention. llms.txt: https://amplitude.com/docs/llms.txt
- **Mixpanel** — event-centric product analytics. llms.txt: https://docs.mixpanel.com/llms.txt
- **Aptabase** — privacy-first open-source analytics (runs in Expo Go). llms.txt: https://aptabase.com/llms.txt
- **OpenPanel** — open-source Mixpanel alternative, optional self-host. llms.txt: https://openpanel.dev/llms.txt
- **Bugsnag** — dedicated stability monitoring + JS-only Expo path. Check https://docs.bugsnag.com.
- **LogRocket** — mobile session replay + state capture. llms.txt: https://docs.logrocket.com/llms.txt

### Backend observability

- **Datadog** — full-stack APM, logs, metrics, mobile RUM. Skill `datadog-labs/agent-skills`; llms.txt: https://docs.datadoghq.com/llms.txt
- **Axiom** — cheap high-volume log/event storage with APL. Skill `axiomhq/skills`; llms.txt: https://axiom.co/docs/llms.txt
- **Better Stack** — hosted log management + uptime/status page. Check https://betterstack.com/docs.
- **OpenTelemetry** — vendor-neutral backend instrumentation. llms.txt: https://opentelemetry.io/llms.txt
