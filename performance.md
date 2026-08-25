# Performance Playbook

Performance is not a stack choice — every stack ships something slow. This file
does not repeat the infrastructure that measures it; it adds the hunting layer
above it.

**Baseline (assumed, enforced elsewhere — do not restate here):**

- `observability.md` — Sentry/PostHog wiring, tracing, and where metrics land.
  This file consumes those numbers, it does not set them up.
- `data.md` — which cache/queue product you run (Redis, Upstash). Cache
  *strategy* is here; cache *selection* is there.
- `backend.md` — connection pooling, migrations, and the swap contract.
- `security/dos-limits.md` — **adversarial** load: ReDoS, zip bombs, cost
  amplification. This file owns **benign** load only. A slow path that is only
  slow because an attacker chose the input is a security finding, not a perf one.

**This file is a router.** It keeps the shared method — the measure-first loop,
the profiler matrix, the budget table, the severity legend, and the triage rules
that kill fake findings — and delegates each bottleneck class to a focused file
under `performance/`. When hunting one class, load only that class's file.

Every rule under `performance/` is tagged **[BLOCKER]** or **[HARDEN]**:

- **[BLOCKER]** — cost grows with data or traffic. N+1 queries, a missing index,
  an unbounded list render, a cache stampede. These pass every dev-machine test
  with 20 rows and fall over in production. Fix before ship.
- **[HARDEN]** — constant-factor waste. A heavy dependency, an unoptimized
  image, a redundant re-render. Real, worth fixing, never the reason a launch
  fails.

Snippets show `// wrong` vs `// right`; every rule ends with a `Verify:` that
carries a number you can compare against.

## When to read

- When something is slow and you do not yet know why. Start here, not in a leaf.
- At the `ship-checklist.md` **Performance** gate before submission.
- When adding a screen that renders a list, a query that joins, or an endpoint
  that fans out.
- Not for a routine already known to be the hot path with a known fix — that is
  a straight rewrite, no hunting needed.

## Bottleneck index

| File | Read when | Stacks |
| --- | --- | --- |
| `performance/bundle-size.md` | slow first load, big download, slow cold start | Next.js, Expo |
| `performance/rendering.md` | jank, dropped frames, typing lag, list stutter | React (web + native), SwiftUI |
| `performance/queries.md` | slow endpoint, DB CPU, timeouts under load | Postgres via Drizzle/Prisma, sqlc/pgx, SQLAlchemy |
| `performance/caching.md` | repeated identical work, thundering herd, stale or leaking data | all |
| `performance/runtime.md` | high CPU, memory growth, blocked event loop, slow under concurrency | Go, Python, Node/Hono, Swift |

## The loop

Never skip step 1. A fix without a baseline is a guess you cannot disprove.

1. **Reproduce and baseline.** Capture one number on the actual slow path —
   p95 latency, cold-start ms, frame time, bundle kB, query ms. Write it down.
   Production-shaped data, not the 20 rows in your dev DB.
2. **Profile, do not guess.** Pick the tool from the matrix below. Read the
   profile top-down: the hot path is where wall-clock actually goes, and it is
   routinely not where you assumed.
3. **Attribute to a class.** Map the hot path to one leaf file, load that file,
   and match the rule. If nothing matches, the bottleneck is novel — measure
   further before writing a fix.
4. **Fix the class, not the line.** A missing index on one query usually means
   the same access pattern is unindexed in three other places. Grep siblings.
5. **Re-measure the same number the same way.** A fix is done when the baseline
   moves, not when the diff looks right. Record before/after in the PR body.
   Label honestly: `measured` vs `reasoned-only`.

## Profiler matrix

One row per stack. Use the repo's existing profiler; do not build a parallel
harness.

| Stack | CPU / hot path | Allocation / memory | Bundle / size |
| --- | --- | --- | --- |
| Next.js (web) | Chrome DevTools Performance panel; Lighthouse for field-shaped scores | DevTools Memory heap snapshot | `ANALYZE=true next build` (`@next/bundle-analyzer`); `source-map-explorer` |
| React (render counts) | `npx react-scan@latest http://localhost:3000`; React DevTools Profiler | — | — |
| Expo / React Native | Hermes sampling profiler via React DevTools; Perf Monitor for JS/UI FPS | Xcode Instruments Allocations on the release build | `EXPO_ATLAS=1 npx expo export` then `npx expo-atlas` |
| Hono / Node | `node --cpu-prof`, open the `.cpuprofile` in DevTools; `clinic flame` | `node --heap-prof`; heap snapshot diff | — |
| Go | `go tool pprof -http=:6060 'http://localhost:8080/debug/pprof/profile?seconds=30'` | `pprof .../heap`; `GODEBUG=gctrace=1` | `go build -ldflags="-s -w"` |
| Python | `py-spy top --pid <pid>`; `py-spy record -o flame.svg --pid <pid> --duration 30` | `memray run -o out.bin app.py` | — |
| Swift / SwiftUI | Instruments **Time Profiler**; **Hangs** for main-thread stalls | Instruments **Allocations** / **Leaks** | Xcode App Thinning size report |
| Postgres | `EXPLAIN (ANALYZE, BUFFERS)`; `pg_stat_statements` ordered by `total_exec_time` | `pg_stat_statements.shared_blks_read` | — |

## Budgets

Numbers to compare a baseline against. A miss is a finding, not automatically a
blocker — apply the severity legend.

| Metric | Budget | Measured with |
| --- | --- | --- |
| LCP (p75, field) | < 2.5s | Lighthouse / web-vitals |
| INP (p75, field) | < 200ms | web-vitals |
| CLS (p75, field) | < 0.1 | web-vitals |
| Next.js First Load JS, per route | < 130 kB gzip | `next build` output |
| Expo JS bundle (Hermes bytecode) | < 4 MB | `expo export` output size |
| Mobile cold start to interactive | < 2s, mid-range device, release build | manual stopwatch or `MXAppLaunchMetric` |
| Frame time, any animation or scroll | < 16.7ms (60fps) | Perf Monitor / Instruments |
| API endpoint latency (p95, server-side) | < 300ms | Sentry / APM |
| Single DB query | < 50ms; anything > 200ms gets an `EXPLAIN` | `pg_stat_statements` |
| Cache hit rate, read-heavy endpoint | > 80% | `redis-cli INFO stats` |
| Renders per user interaction, per component | <= 2 | react-scan / React Profiler |

## Triage — kill the fake findings first

Perf findings are wrong more often than security findings, because the
measurement itself lies. Before reporting anything, rule out:

1. **Dev-mode artifacts.** React StrictMode double-invokes renders in dev. The
   Next dev server compiles per request. Expo dev builds run without Hermes
   bytecode optimizations. **Every number that matters comes from a production
   build.** A finding measured in dev mode is not a finding.
2. **Cold vs warm.** First run pays JIT warmup, an empty cache, a cold
   connection pool, and an unwarmed page cache. Measure the steady state, and
   separately measure cold start if cold start is the complaint.
3. **N=1 measurement.** One run is noise. Take the median of at least 5, and
   report p95 rather than the mean for anything user-facing.
4. **Dev-size data.** A missing index is invisible at 20 rows and fatal at 2M.
   If you cannot get production-shaped data, seed it — a query verdict on an
   empty table is worthless.
5. **The wrong layer.** A slow endpoint is not automatically a slow query. Time
   the DB call separately from the handler before blaming either.

## Do not optimize

- **Anything you have not profiled.** The most common wasted day.
- **Code that is not on the hot path.** A 10x win on 0.5% of wall-clock is 0.45%.
- **Before the data grows.** Correct-and-simple beats fast-and-clever until a
  measurement says otherwise. The exception is any **[BLOCKER]** class — those
  scale with N by construction, so they are fixed on sight, not on measurement.
- **By adding memoization everywhere.** `useMemo`/`React.memo` on cheap
  components costs more than it saves and hides the real cause.
- **By fiddling with the measurement.** Tuning the benchmark harness is not
  making the product faster.
- **Micro-benchmarks with no production shape.** Different data, different cache
  state, different answer.

Never report a gain you did not measure.

## Finding format

```markdown
### [CATEGORY] Short imperative title
- Evidence: `path/file.ts:123` — what's there, plus the profile line that points at it.
- Baseline: the number, and how it was measured (tool, build mode, sample size).
- Impact: what it costs at production scale, not at dev scale.
- Effort: S / M / L.
- Fix sketch: 1-3 sentences.
- After: the re-measured number, or `not yet measured`.
```
