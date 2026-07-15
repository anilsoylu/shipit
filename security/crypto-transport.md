# Security: Cryptography & transport

Part of `../security.md` (tags, threat model, and review method live there).

- **[BLOCKER] CSPRNG for every security token** — reset tokens, OTPs, session ids,
  API keys, and nonces come from a CSPRNG; `Math.random()` is predictable, so a
  forgeable reset token is account takeover.
  ```ts
  // wrong
  const token = Math.random().toString(36).slice(2)
  // right
  import { randomBytes, randomUUID } from "crypto"
  const token = randomBytes(32).toString("base64url")   // randomUUID() for ids
  ```
  Go `crypto/rand` (not `math/rand`); Python `secrets` (not `random`); Rust `OsRng`. Verify: grep `Math.random(|math/rand|random.random(|random.randint(` in token/id/secret/otp paths → none.
- **[BLOCKER] Password hashing is a slow salted KDF** — self-hosted auth
  (custom-JWT, better-auth) hashes with argon2id (or bcrypt/scrypt), per-password
  salt; never MD5/SHA-1/SHA-256/plaintext.
  ```ts
  // wrong
  const h = crypto.createHash("sha256").update(password).digest("hex")
  // right
  import { hash, verify } from "@node-rs/argon2"
  const stored = await hash(password)                   // argon2id, salted
  ```
  Verify: grep `createHash(.*(sha|md5)` in the password path → none; stored hashes start with `$argon2id$`/`$2b$`.
- **[BLOCKER] Never disable TLS verification** — server-to-server calls (webhook
  verify, provider APIs, DB) keep certificate validation on; `rejectUnauthorized:
  false` / `NODE_TLS_REJECT_UNAUTHORIZED=0` / `verify=False` / `InsecureSkipVerify`
  is a MITM hole that voids every signature and secret in transit.
  ```ts
  // wrong
  new https.Agent({ rejectUnauthorized: false })
  // right — verification on; pin a private CA explicitly only if you run one
  new https.Agent({ ca: fs.readFileSync("private-ca.pem") })
  ```
  Verify: grep `rejectUnauthorized:\s*false|NODE_TLS_REJECT_UNAUTHORIZED|verify=False|InsecureSkipVerify` → none in app/prod code.
- **[HARDEN] No weak or homemade crypto** — AES-GCM / ChaCha20-Poly1305 for
  symmetric (never ECB, never a static/reused IV/nonce); no hand-rolled crypto;
  constant-time compare (`timingSafeEqual`) for secrets and MACs. Verify: grep `createCipheriv(.*ecb` and secret comparisons using `===`/`==` instead of `timingSafeEqual`.
