# Security: Dependencies & supply chain

Part of `../security.md` (tags, threat model, and review method live there).

- **[BLOCKER] CI fails on reachable critical/high advisories** — a read-only audit
  gate per ecosystem, blocking on critical/high in runtime or build/distribution
  paths; dev-only/unreachable advisories are triaged, not silently ignored (and
  never auto-fixed in CI).
  ```yaml
  - run: pnpm audit --prod --audit-level=high     # apps/web, apps/mobile
  - run: pip-audit -r apps/api/requirements.txt    # Python API
  - run: cargo audit --deny warnings               # Rust API
  - run: govulncheck ./...                         # Go API (reachability-aware)
  ```
  Verify: add a package with a known high advisory → the CI job fails and blocks the merge.
- **[HARDEN] Lockfile committed, pinned, no drift** — commit the lockfile, install
  with `--frozen-lockfile` in CI, dedupe versions across `apps/*`. Verify: `pnpm install --frozen-lockfile` is clean in CI; `git ls-files '*lock*'` shows it tracked.
- **[HARDEN] SRI or self-host third-party scripts** — a CDN `<script>`/`<link>`
  carries `integrity` + `crossorigin`, or you self-host; a mutable CDN URL is a
  supply-chain injection path. Verify: view-source → each external script tag has an `integrity=` hash.
- **[HARDEN] Pin CI actions and base images by digest** — GitHub Actions by commit
  SHA (not `@v3`) and Docker `FROM` by `@sha256:`, so a moved tag can't swap in
  malicious build steps. Verify: grep workflows for `uses:.*@[0-9a-f]{40}`; `grep FROM Dockerfile` shows `@sha256:` digests.
- **[HARDEN] Harden the container image** — run as a non-root `USER`, start from a
  minimal base (slim/distroless), use `COPY` not `ADD` (ADD auto-fetches URLs and
  auto-extracts archives), and take build-time credentials via BuildKit
  `--mount=type=secret`, never `ARG`/`ENV` (those persist in the image layers — see
  `secrets-config.md`). `EXPOSE` is documentation, not a control; publish/route only
  the needed port at Coolify/Traefik (`edge-proxy.md`).
  ```dockerfile
  # wrong: root, secret in a build arg (visible in `docker history`), ADD
  ARG NPM_TOKEN
  ADD https://example.com/app.tar.gz /app
  # right: BuildKit secret, non-root, explicit COPY
  RUN --mount=type=secret,id=npm_token  npm ci
  COPY --chown=app:app . /app
  USER app
  ```
  Verify: `docker history --no-trunc <image>` shows no secret literal; `docker inspect` / the running container reports a non-root user; `grep -E '^(ADD|USER|ARG)' Dockerfile` — no `ADD` on remote/archive input, a `USER` set, no secret-bearing `ARG`.
