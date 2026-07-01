# Auth Playbook: Custom JWT

Use custom JWT when: a hard requirement forces you to own the full auth stack.
**Prefer a managed provider (Clerk, better-auth, Supabase) unless you have no
choice** — you take on password storage, token rotation, and every security edge
case yourself.

## Client (Expo)

- No provider SDK. The client posts credentials to your API's `/auth/login` and
  receives a short-lived access JWT + a refresh token.
- Store both tokens in `expo-secure-store` only — never AsyncStorage or plaintext.
- Send `Authorization: Bearer <access_token>` on every protected request.
- On a `401`, call `/auth/refresh` with the refresh token to rotate, then retry.

## Server verification

- Hash passwords with a memory-hard algorithm (argon2id preferred, or bcrypt).
  Never store plaintext or fast-hash (SHA/MD5) passwords.
- Issue a short-lived access JWT (minutes) signed with a server-only secret
  (HS256) or private key (RS256/ES256), plus a longer-lived refresh token.
- **Refresh token rotation**: each refresh issues a new refresh token and
  invalidates the old one; detect reuse of a rotated token and revoke the family.
- Verify the access JWT signature and expiry server-side on every protected
  request. Derive actor identity only from the verified token claims.
- Persist refresh tokens (hashed) so they can be revoked on logout / compromise.

## User mapping

- Your own user table is the source of truth. The JWT `sub` is the internal user
  id directly — no external mapping needed.

## Webhooks

- None inherent. If external systems notify your API of user events, verify their
  signature before mutating.

## Env vars

- Public (bundle-safe): API base URL only.
- Secret (server only): `JWT_SECRET` (or `JWT_PRIVATE_KEY`), `JWT_REFRESH_SECRET`.

## Contract compliance

1. Access + refresh tokens stored in `expo-secure-store`. ✓
2. Every protected request sends the bearer access token. ✓
3. API verifies the JWT signature + expiry server-side on every request. ✓
4. Actor identity comes only from verified token claims. ✓
5. `sub` is the internal user id (own table is source of truth). ✓
6. Any external event channel verifies its signature before mutating. ✓
7. No secrets in the bundle; signing + refresh secrets stay server-side. ✓
8. `GET /health` requires no auth. ✓

## Official References

- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP JWT / session management guidance:
  https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- argon2 password hashing: https://github.com/ranisalt/node-argon2
