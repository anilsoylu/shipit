# AI Features Playbook

AI features are optional. Do not add AI infrastructure unless the product needs
it. Default gateway is OpenRouter + the Vercel AI SDK, with the Claude model
family as the default. Providers are pluggable — whatever you pick must satisfy
the swap contract below, and all calls stay server-side.

## When To Add

Add an AI module only if the MVP has a clear user-facing job:

- Generate, rewrite, summarize, classify, extract, coach, search, rank, or automate.
- The user can see the output and it changes retention, conversion, or monetization.
- There is a cost control story before public launch.

Do not add chat UI by default. Many apps need a button, form, result card, or background automation instead.

## AI Swap Contract

Every AI integration must satisfy these:

1. Provider API keys live in server env ONLY (`apps/api`); never in the Expo
   bundle. The mobile app calls your API, never a provider directly.
2. Every AI endpoint verifies the user, then checks plan/credits/quota and a
   Redis rate limit before calling the provider.
3. Validate input length/type and cap max input; enforce request timeouts.
4. Stream responses through your API when the UX needs streaming.
5. Log cost metadata without sensitive prompt/PII data unless explicitly needed;
   feature-flag expensive endpoints.

## API Pattern

Recommended API shape:

```txt
POST /ai/<job>
```

Server responsibilities:

- Verify the authenticated user (see `auth.md`).
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

## Add a provider

Copy this skeleton into a provider section and fill it in:

```markdown
# AI Playbook: <Provider>

Use <Provider> when: <selection criteria>.

## Server setup
- SDK/package, server env key, how it's wired into a POST /ai/<job> handler.

## Contract compliance
- Restate the 5 swap-contract invariants; confirm each is met.

## Official References
- Provider docs, llms.txt, and skill (from the LLM Docs Registry).
```

## Playbooks

Load the option's llms.txt before implementing. Skills/URLs are in the LLM Docs
Registry in `skills.md`.

### Gateways

- **OpenRouter** (default) — one key to Claude/GPT/Gemini/Llama with fallback.
  Skill `openrouterteam/agent-skills`; llms.txt https://openrouter.ai/docs/llms.txt
- **Vercel AI SDK** (default) — TS toolkit to wire providers into Fastify
  handlers with streaming. Skill `vercel/ai`; llms.txt https://ai-sdk.dev/llms.txt
- **Portkey** — routing, caching, guardrails, budgets, observability. Check https://portkey.ai/docs.
- **LiteLLM** — self-hostable proxy, 100+ providers, OpenAI-compatible. Check https://docs.litellm.ai.
- **Helicone** — drop-in LLM observability (logging, caching, cost). Check https://docs.helicone.ai.

### Direct providers

- **Anthropic (Claude)** — default model family; best reasoning, tool use, prompt
  caching. Skill `anthropics/skills`; llms.txt https://platform.claude.com/llms.txt
- **OpenAI** — GPT, Realtime/voice, Whisper, image gen. Skill `openai/skills`; llms.txt https://developers.openai.com/api/docs/llms.txt
- **Google Gemini** — large context, native multimodal. llms.txt https://ai.google.dev/gemini-api/docs/llms.txt
- **Groq** — very low latency, OpenAI-compatible. llms.txt https://console.groq.com/llms.txt
- **Together / Fireworks** — broad open-weight catalog; OpenAI-compatible. Check vendor docs.

### Media generation

- **Replicate** — image/video/audio or any community model. llms.txt https://replicate.com/llms.txt
- **fal.ai** — fast image/video/audio with streaming/queue. llms.txt https://fal.ai/docs/llms.txt
- **ElevenLabs** — TTS, voice cloning, STT, conversational voice. llms.txt https://elevenlabs.io/docs/llms.txt

### Agent frameworks

- **Mastra** — TS agents with tools, workflows, memory, RAG on the backend.
  Skill `mastra-ai/skills`; llms.txt https://mastra.ai/llms.txt
- **LangGraph** — stateful graph-based multi-agent orchestration. Check https://docs.langchain.com.

### RAG / vector stores

Default RAG store is **pgvector** on the existing Postgres+Drizzle. Store setup
lives in `data.md` — link, do not duplicate here.

- **pgvector** (default) — vector columns/indexes on existing Postgres. See `data.md`.
- **Upstash Vector** — serverless HTTP vector DB. Skill `upstash/skills`. See `data.md`.
- **Pinecone** — managed high-scale vector DB. Skill `pinecone-io/skills`; llms.txt https://docs.pinecone.io/llms.txt
- **Turso Vector (libSQL)** — vector columns + ANN in SQLite/libSQL. See `data.md`.

## UX Rules

- Prefer structured forms over open-ended chat unless chat is the product.
- Show progress and allow retry.
- Cache or persist generated results when recomputation costs money.
- Make failure states useful: "try again", "shorten input", or "upgrade" depending on the product.

## Official References

- OpenRouter quickstart: https://openrouter.ai/docs/quickstart
- OpenRouter models: https://openrouter.ai/models
