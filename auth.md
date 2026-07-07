# Auth Playbook

Default auth is Clerk. Auth is swappable — pick a provider, then follow its
playbook. Whatever you pick must satisfy the Auth Swap Contract below.

## Choosing a provider

- **Clerk** (default): fastest Expo auth, hosted, polished session handling. Use
  unless the user requires otherwise. See `auth-clerk.md`.
- **better-auth**: self-hosted, own your data, runs in the TS API process. Pairs
  with the Fastify default / Hono option. See `auth-betterauth.md`.
- **Supabase Auth**: already using Supabase, or want Postgres-native auth. See
  `auth-supabase.md`.
- **Custom JWT**: full control, you own all security. Discouraged unless a hard
  requirement forces it. See `auth-custom-jwt.md`.

## Auth Swap Contract

Every provider must satisfy these invariants:

1. Clients obtain a session/token from the provider's client SDK. Mobile stores
   tokens only in secure storage (`expo-secure-store`), never AsyncStorage or
   plaintext. Web uses the provider's httpOnly cookie session handling; tokens
   never go to `localStorage`.
2. Every protected API request carries the provider's bearer token or session.
3. The API verifies the token **server-side on every protected request** — via
   the provider's JWKS (for JWTs), the provider's server SDK verifier, or a
   token-introspection endpoint. An unverified token is never trusted.
4. The API derives actor identity (user id) **only** from the verified token —
   never from a `userId` in the request body, query, or header.
5. The API maps the provider user id to an internal user record via an
   idempotent upsert when app-specific profile data is needed.
6. Auth webhooks (user created/updated/deleted, session events) verify the
   provider's signature before mutating user records.
7. Only publishable/public keys ship in client bundles (Expo or web). Secret
   keys, JWKS signing secrets, and webhook secrets stay in server env only.
8. `GET /health` never requires auth.

## Add a new provider

Copy this skeleton into `auth-<provider>.md` and fill it in:

```markdown
# Auth Playbook: <Provider>

Use <Provider> when: <one or two selection criteria>.

## Client (Expo)
- Package(s) and install command.
- Provider wrapper/provider component and where it mounts.
- Sign-up / sign-in / sign-out flow.
- How the client retrieves the token for API calls.
- Secure token storage (expo-secure-store).

## Client (Web) — when web is in scope
- Package(s) for the web framework (e.g. Next.js SDK/middleware).
- Session handling (httpOnly cookies) and how server components/route
  handlers read the session.

## Server verification
- How the API validates the token (JWKS URL / server SDK / introspection).
- Middleware/plugin shape (framework-agnostic description).

## User mapping
- Provider user id -> internal user record (idempotent upsert).

## Webhooks
- Events to handle and how signatures are verified.

## Env vars
- Public (bundle-safe): ...
- Secret (server only): ...

## Contract compliance
- Restate the 8 contract invariants and confirm this provider meets each.

## Official References
- Provider docs, llms.txt, and skill (from the LLM Docs Registry).
```

To support a provider not listed here, copy the skeleton into
`auth-<provider>.md`, satisfy every contract invariant, add its row to the LLM
Docs Registry in `skills.md`, and link it from this file and the index files.

## Provider playbooks

- `auth-clerk.md` — Clerk (default).
- `auth-betterauth.md` — better-auth (self-hosted TS).
- `auth-supabase.md` — Supabase Auth.
- `auth-custom-jwt.md` — custom JWT (you own security).
