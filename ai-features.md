# AI Features Playbook

AI features are optional. Do not add AI infrastructure unless the product needs it.

## Default Rule

All provider calls happen in `apps/api`.

Never put OpenRouter, DeepSeek, OpenAI, Anthropic, Google, or other provider keys in the Expo app.

## When To Add

Add an AI module only if the MVP has a clear user-facing job:

- Generate, rewrite, summarize, classify, extract, coach, search, rank, or automate.
- The user can see the output and it changes retention, conversion, or monetization.
- There is a cost control story before public launch.

Do not add chat UI by default. Many apps need a button, form, result card, or background automation instead.

## API Pattern

Recommended API shape:

```txt
POST /ai/<job>
```

Server responsibilities:

- Verify Clerk user.
- Check plan, credits, quota, or rate limit.
- Validate input length and type.
- Select provider/model from server config.
- Call provider.
- Store only the product-relevant result.
- Log cost metadata without sensitive prompt data unless explicitly needed.
- Return typed output to mobile.

## Provider Strategy

Default abstraction:

- `modelProvider`: `openrouter`, `direct`, or `local`.
- `model`: provider model slug.
- `maxInputChars`, `timeoutMs`, `dailyLimit`, `costClass`.

OpenRouter is useful when:

- You want one API for multiple models.
- You want to test DeepSeek, OpenAI, Anthropic, Google, or other models without changing mobile code.
- You need routing flexibility.

Direct provider is useful when:

- A specific API feature is required.
- Latency, cost, compliance, or model behavior demands direct integration.

## Cost Controls

Before public exposure:

- Add Redis token bucket rate limits.
- Add per-user daily or monthly quotas.
- Add request timeouts.
- Add max input size.
- Add abuse logging.
- Add feature flags for expensive endpoints.

## UX Rules

- Prefer structured forms over open-ended chat unless chat is the product.
- Show progress and allow retry.
- Cache or persist generated results when recomputation costs money.
- Make failure states useful: "try again", "shorten input", or "upgrade" depending on the product.

## Official References

- OpenRouter quickstart: https://openrouter.ai/docs/quickstart
- OpenRouter models: https://openrouter.ai/models
