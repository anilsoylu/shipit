# Auth Playbook: better-auth

Use better-auth when: you want self-hosted auth that runs inside the TS API
process, own your user data, and pair it with the Fastify default or Hono option.

## Client (Expo)

- Install `better-auth @better-auth/expo expo-secure-store expo-network`
  (social login also needs `expo-linking expo-web-browser expo-constants`).
- Create the client with the Expo plugin and SecureStore storage:

  ```ts
  import { createAuthClient } from "better-auth/react";
  import { expoClient } from "@better-auth/expo/client";
  import * as SecureStore from "expo-secure-store";

  export const authClient = createAuthClient({
    baseURL: process.env.EXPO_PUBLIC_API_URL,
    plugins: [
      expoClient({ scheme: "myapp", storagePrefix: "myapp", storage: SecureStore }),
    ],
  });
  ```

- Sessions/cookies are cached in `expo-secure-store` (never AsyncStorage).
- Sign-up / sign-in / sign-out via `authClient.signUp.email` /
  `authClient.signIn.email` / `authClient.signOut`.
- Attach auth to API calls. Cookie-based (same-origin API): `authClient.getCookie()`
  returns the cookie to set on `fetch` headers. For a bearer flow, enable the
  bearer plugin and send `Authorization: Bearer <token>`.
- `app.json`: set `"expo": { "scheme": "myapp" }`.

## Server verification

- Init in the API process:

  ```ts
  import { betterAuth } from "better-auth";
  import { expo } from "@better-auth/expo";

  export const auth = betterAuth({
    plugins: [expo()],
    emailAndPassword: { enabled: true },
    trustedOrigins: ["myapp://", "myapp://*"], // dev adds exp:// origins
    // database: drizzleAdapter(...) — see better-auth Drizzle adapter docs
  });
  ```

- Same-process (TS) verification: call `auth.api.getSession({ headers })` in a
  middleware and attach the verified user to the request context.
- **Non-TS or externally-hosted API**: enable the **JWT plugin**. The client
  obtains a JWT (`authClient.token()` or the `set-auth-jwt` response header); the
  API verifies it **offline against the JWKS** at `/api/auth/jwks` (path is
  customizable, e.g. `/.well-known/jwks.json`) using `jose`
  (`createRemoteJWKSet` + `jwtVerify`), matching `issuer`/`audience` to the
  better-auth base URL. The `kid` header selects the key; no DB round-trip.

## User mapping

- better-auth owns the user table. Map its user id to any app-specific profile
  record via an idempotent upsert keyed on the better-auth user id.

## Webhooks

- Not required for core email/password auth (better-auth writes the user table
  directly). If you fan out user events to another service, verify that channel's
  own signature before mutating.

## Env vars

- Public (bundle-safe): `EXPO_PUBLIC_API_URL` (the base URL).
- Secret (server only): `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`.

## Contract compliance

1. Tokens/cookies stored in `expo-secure-store` via `expoClient({ storage })`. ✓
2. Every protected request sends the better-auth cookie or bearer JWT. ✓
3. API verifies server-side: `getSession` (TS) or JWKS `jwtVerify` (external). ✓
4. Actor identity comes from the verified session/JWT, never client input. ✓
5. better-auth user id upserted to an internal profile record when needed. ✓
6. No auth webhooks in the default flow; any fan-out channel verifies its own
   signature. ✓
7. Only the base URL ships in the bundle; `BETTER_AUTH_SECRET` stays
   server-side. ✓
8. `GET /health` requires no auth. ✓

## Official References

- better-auth LLM docs: https://www.better-auth.com/llms.txt
- Expo integration: https://www.better-auth.com/docs/integrations/expo
- JWT plugin: https://www.better-auth.com/docs/plugins/jwt
- Drizzle adapter: https://www.better-auth.com/docs/adapters/drizzle
