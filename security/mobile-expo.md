# Security: Mobile / Expo

Part of `../security.md` (tags, threat model, and review method live there).

- **[BLOCKER] No real secret in the JS bundle** — `EXPO_PUBLIC_*` and any imported
  literal are inlined into the shipped bundle at build; only publishable/anon keys
  belong client-side, every real key stays behind the API. Verify: `grep -r EXPO_PUBLIC apps/mobile | grep -iE 'secret|sk_live|service_role|private'` → empty; unzip the IPA/APK and grep the JS.
- **[HARDEN] Lock SecureStore accessibility** — tokens in `expo-secure-store`
  (baseline) with `keychainAccessible: WHEN_UNLOCKED_THIS_DEVICE_ONLY` so they
  never sync to iCloud/backups. Verify: `grep -rn SecureStore apps/mobile` shows the option; no token in AsyncStorage/MMKV.
- **[HARDEN] Strip logs and PII from release builds** — `console.*` persists to
  device logs and can carry tokens/PII; drop it in production
  (`transform-remove-console`), never log request/response bodies. Verify: `adb logcat` during sign-in shows no token/PII.
- **[HARDEN] Verified links for auth, not custom schemes** — custom schemes are
  claimable by other apps; use Universal Links / App Links (AASA, `assetlinks.json`)
  for OAuth/magic-link callbacks, validate every param, and treat a deep link as
  never proof of auth. Verify: a crafted `myapp://…` link → no privileged nav without a verified session.
- **[HARDEN] Block sensitive screens from snapshots** — Android `FLAG_SECURE` /
  `expo-screen-capture` + a privacy overlay on background, so the OS app-switcher
  snapshot and screenshots capture nothing on token/PII screens. Verify: background on a sensitive screen → recents thumbnail is blank/blurred.
- **[HARDEN] Don't trust the client for integrity** — root/jailbreak flags are
  bypassable and must never gate server-side authorization; for high-value flows
  use server-verified Play Integrity / App Attest, and sign OTA updates (EAS Update
  code signing). Verify: server rejects the action when attestation is absent; a tampered OTA update is rejected.
