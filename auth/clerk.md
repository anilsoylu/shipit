# Auth Playbook: Clerk

Use Clerk when: you want the fastest hosted Expo auth with polished session
handling and minimal server code. This is the default.

## Client (Expo)

- Install `@clerk/clerk-expo` and `expo-secure-store`.
- Mount `<ClerkProvider publishableKey={...} tokenCache={...}>` at the app root;
  wrap protected routes with `<SignedIn>` / `<SignedOut>`.
- Use Clerk's secure-store token cache so session tokens live in
  `expo-secure-store`, not AsyncStorage.
- Sign-up / sign-in / sign-out via Clerk's hooks (`useSignIn`, `useSignUp`,
  `useAuth().signOut`).
- Retrieve the API token with `useAuth().getToken()` and send it as
  `Authorization: Bearer <token>` on every protected request.

## Server verification

- The API verifies the incoming Clerk session/JWT server-side on every protected
  request via Clerk's backend SDK (or its JWKS). Never trust an unverified token.
- Shape: an auth middleware/plugin that reads the bearer token, verifies it, and
  attaches the verified `userId` to the request context. Describe it
  framework-agnostically; the Fastify wiring lives in `../backend.md`.

## User mapping

- Map the Clerk user id to an internal user record via an idempotent upsert when
  the product needs app-specific profile data.

## Webhooks

- Handle `user.created` / `user.updated` / `user.deleted`.
- Verify the Clerk webhook signature (Svix headers) using `CLERK_WEBHOOK_SECRET`
  before mutating user records. Store the raw body for verification.

## Env vars

- Public (bundle-safe): `CLERK_PUBLISHABLE_KEY` (or `EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY`).
- Secret (server only): `CLERK_SECRET_KEY`, `CLERK_WEBHOOK_SECRET`.

## Contract compliance

1. Tokens cached in `expo-secure-store` via Clerk's token cache. ✓
2. Every protected request sends `Authorization: Bearer <getToken()>`. ✓
3. API verifies the token server-side via Clerk's backend SDK / JWKS. ✓
4. Actor identity comes only from the verified token, never client input. ✓
5. Clerk user id upserted to an internal user record when needed. ✓
6. Webhooks verify the Svix signature before mutating. ✓
7. Only the publishable key ships in the bundle; secret + webhook secret stay
   server-side. ✓
8. `GET /health` requires no auth. ✓

## Official References

- Clerk LLM docs: https://clerk.com/docs/llms-full.txt
- Clerk Expo quickstart: https://clerk.com/docs/quickstarts/expo
- Clerk backend requests: https://clerk.com/docs/backend-requests/overview
- Clerk skills: `clerk`, `clerk-expo`, `clerk-expo-patterns` (see `../skills.md`).
