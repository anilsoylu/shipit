# Security: Secrets & config

Part of `../security.md` (tags, threat model, and review method live there).

- **[BLOCKER] A committed secret is burned — rotate, don't just delete** — removing
  the line leaves it in git history, forks, and backups; rotate at the provider
  first, then purge. Report as `path:line` + credential type, never the value.
  ```bash
  gitleaks detect --no-git --redact -v          # working tree; drop --no-git for full history
  # fallback (detection patterns, not secrets):
  grep -rnE 'sk_live_|rk_live_|AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}|-----BEGIN [A-Z ]+PRIVATE KEY-----' .
  ```
  Verify: `gitleaks detect --no-git --redact` exits 0; after a burn, confirm the old key is revoked at the provider (a rotated-but-not-revoked key is still live).
- **[BLOCKER] Diagnostics endpoints are not public** — prod exposes nothing at
  `/debug`, `/admin`, `/metrics`, `/env`, `/actuator`, Node `--inspect`, or Go
  `pprof`/`expvar`; bind to localhost/private network or require auth at Traefik
  (`edge-proxy.md`).
  ```bash
  for p in /metrics /debug/pprof/ /env /actuator/health; do
    curl -s -o /dev/null -w "$p %{http_code}\n" https://api.example.com$p; done   # expect 404/401
  ```
  Verify: the loop above returns 404/401, never 200.
- **[HARDEN] `.gitignore` covers secret files and none are tracked** — ignore
  `.env*`, `*.pem`, `*.key`, `credentials.json`; a file added before the ignore rule
  stays tracked forever. Verify: `git ls-files | grep -E '\.env($|\.)|\.pem$|\.key$|credentials'` → empty.
- **[HARDEN] Secrets from the environment, not the image or repo** — inject via
  Coolify env/secret store at runtime; never bake a secret into a Docker layer or
  commit it to `eas.json`/CI YAML. Verify: `docker history --no-trunc <image>` and grep `eas.json .github/` show no secret literals.
- **[HARDEN] Error hygiene — generic to the client, detailed to the log** — no stack
  traces, SQL, driver errors, file paths, or internal hostnames in client responses;
  log detail server-side keyed by a request id; no publicly served source maps.
  Verify: force a DB error via the API → generic body + id, full detail only in the server log/Sentry.
