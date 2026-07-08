# Auth Playbook: Supabase Auth

Use Supabase Auth when: you already use Supabase, or want Postgres-native auth
with a hosted identity layer. Your custom API stays separate from Supabase and
verifies the JWT itself.

## Client (Expo)

- Install `@supabase/supabase-js @react-native-async-storage/async-storage
  react-native-url-polyfill`.
- Create the client with a React Native storage adapter and `processLock`:

  ```ts
  import 'react-native-url-polyfill/auto'
  import AsyncStorage from '@react-native-async-storage/async-storage'
  import { createClient, processLock } from '@supabase/supabase-js'

  export const supabase = createClient(
    process.env.EXPO_PUBLIC_SUPABASE_URL!,
    process.env.EXPO_PUBLIC_SUPABASE_PUBLISHABLE_KEY!,
    { auth: {
        storage: AsyncStorage,
        autoRefreshToken: true,
        persistSession: true,
        detectSessionInUrl: false,
        lock: processLock,
      } },
  )
  ```

  The quickstart uses AsyncStorage. To satisfy contract invariant 1, swap in an
  `expo-secure-store`-backed storage adapter (implement `getItem`/`setItem`/
  `removeItem` over SecureStore) so session tokens never sit in plaintext
  AsyncStorage.
- Register an `AppState` listener to call `supabase.auth.startAutoRefresh()` /
  `stopAutoRefresh()` on foreground/background.
- Sign-up / sign-in / sign-out via `supabase.auth.signUp` /
  `signInWithPassword` / `signOut`.
- Retrieve the access token with `supabase.auth.getSession()` and send it as
  `Authorization: Bearer <access_token>` to your API.

## Server verification

- The custom API is separate from Supabase's PostgREST, so it verifies the
  Supabase-issued JWT itself. **Do not rely on RLS as the only gate for the
  custom API.**
- **Current-recommended (2026)**: Supabase asymmetric signing keys (ES256/RS256/
  EdDSA) + JWKS. Fetch public keys from
  `https://<project-ref>.supabase.co/auth/v1/.well-known/jwks.json` and verify
  offline with `jose` (`createRemoteJWKSet` + `jwtVerify`), matching `kid`/`alg`.
- Legacy HS256 shared-secret (`SUPABASE_JWT_SECRET`) verification is discouraged
  (impersonation/compliance risk). If stuck on HS256, verify by calling
  `GET /auth/v1/user` with the bearer token instead.

## User mapping

- Map the Supabase `sub` (user id) to an internal user record via an idempotent
  upsert when app-specific profile data is needed.

## Webhooks

- If using Supabase Auth Hooks / database webhooks to react to user events,
  verify the hook signature/secret before mutating records.

## Env vars

- Public (bundle-safe): `EXPO_PUBLIC_SUPABASE_URL`,
  `EXPO_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (new `sb_publishable_...`; legacy anon
  key still works).
- Secret (server only): `sb_secret_...` (legacy `service_role`); for legacy JWT
  verification only, `SUPABASE_JWT_SECRET`. Legacy anon/service_role keys are
  deprecated by end of 2026.

## Contract compliance

1. Session tokens in secure storage — use a SecureStore adapter (not the default
   AsyncStorage) to comply. ✓ (with the adapter swap above)
2. Every protected request sends `Authorization: Bearer <access_token>`. ✓
3. API verifies the JWT server-side via JWKS (`jwtVerify`), not RLS alone. ✓
4. Actor identity comes from the verified JWT `sub`, never client input. ✓
5. Supabase user id upserted to an internal record when needed. ✓
6. Auth hooks/webhooks verify their signature before mutating. ✓
7. Only the URL + publishable key ship in the bundle; secret + JWT secret stay
   server-side. ✓
8. `GET /health` requires no auth. ✓

## Official References

- Supabase LLM docs: https://supabase.com/llms.txt
- React Native auth quickstart: https://supabase.com/docs/guides/auth/quickstarts/react-native
- JWT signing keys / JWKS: https://supabase.com/docs/guides/auth/signing-keys
- Supabase skills: `supabase` (see `../skills.md`).
