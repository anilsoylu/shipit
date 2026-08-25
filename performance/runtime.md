# Performance: Runtime hot paths

Part of `../performance.md` (loop, budgets, severity legend, and triage rules
live there). Covers the server and native runtimes: Go, Python, Node/Hono, and
Swift. This is where you land when the bottleneck is neither the bundle, nor a
render, nor a query — the process itself is busy, growing, or blocked.

Budget: API endpoint p95 < 300ms server-side, event-loop delay p99 < 100ms, no
main-thread hang over 250ms on mobile.

## Cross-stack rules

- **[BLOCKER] Do not serialize independent awaits** — a request that awaits three
  unrelated calls in sequence costs their sum instead of their max. This is the
  single most common server-side waterfall.
  ```ts
  // wrong — 3 x 80ms = 240ms
  const user = await getUser(id); const plan = await getPlan(id); const flags = await getFlags(id)
  // right — 80ms
  const [user, plan, flags] = await Promise.all([getUser(id), getPlan(id), getFlags(id)])
  ```
  ```go
  g, ctx := errgroup.WithContext(ctx)
  g.Go(func() error { ... }); g.Go(func() error { ... })
  err := g.Wait()
  ```
  ```python
  user, plan, flags = await asyncio.gather(get_user(id), get_plan(id), get_flags(id))
  ```
  Verify: endpoint p95 drops to roughly the slowest single dependency, not their sum.
- **[BLOCKER] Bound every fan-out** — `Promise.all` over a user-sized array, or a
  goroutine per row, opens as many connections as the data happens to have.
  It works at 10 items and exhausts the pool at 10,000.
  ```ts
  const limit = pLimit(10)
  await Promise.all(ids.map((id) => limit(() => fetchOne(id))))
  ```
  ```go
  g.SetLimit(10)
  ```
  ```python
  sem = asyncio.Semaphore(10)
  ```
  Verify: run the path with 10,000 items — pool saturation stays under 80% and the process does not run out of file descriptors.
- **[BLOCKER] Keep work the response does not need off the request path** — email
  sends, thumbnail generation, webhook fan-out, and analytics writes belong in a
  queue (`../data.md` background jobs), not inline before the 200.
  Verify: the endpoint's p95 no longer moves when the downstream provider is slow.

## Go

```bash
# attach to a running server (import _ "net/http/pprof")
go tool pprof -http=:6060 'http://localhost:8080/debug/pprof/profile?seconds=30'
go tool pprof -http=:6060 'http://localhost:8080/debug/pprof/heap'
go test -bench . -benchmem -run '^$' ./internal/...
```

- **[BLOCKER] Every goroutine needs an exit** — a goroutine blocked on a channel
  nobody closes, or on a request with no context deadline, leaks its whole stack
  and everything it references. The process looks fine for hours, then does not.
  ```go
  // wrong
  go func() { for v := range ch { work(v) } }()      // never returns if ch is never closed
  // right
  go func() { for { select { case v, ok := <-ch: if !ok { return }; work(v); case <-ctx.Done(): return } } }()
  ```
  Verify: `curl 'localhost:8080/debug/pprof/goroutine?debug=1' | head -1` before and after a load run — the count returns to its baseline, it does not ratchet up.
- **[HARDEN] Preallocate in hot loops** — `append` to a nil slice reallocates and
  copies as it grows; in a per-request loop that is measurable GC pressure.
  ```go
  out := make([]Item, 0, len(rows))   // one allocation instead of log2(n)
  ```
  Verify: `go test -bench . -benchmem` — `allocs/op` drops; `GODEBUG=gctrace=1` shows fewer GC cycles under the same load.

## Python

```bash
py-spy top --pid <pid>                                   # live, no restart, no instrumentation
py-spy record -o flame.svg --pid <pid> --duration 30
memray run -o out.bin app.py && memray flamegraph out.bin
```

- **[BLOCKER] No blocking call inside an async handler** — one `requests.get`,
  one sync DB driver, or one `time.sleep` in a coroutine stalls the entire event
  loop, so every other in-flight request waits on it.
  ```python
  # wrong
  async def handler(): r = requests.get(url)          # blocks the loop for the whole round trip
  # right
  async def handler(): r = await client.get(url)      # httpx.AsyncClient
  # right — when the library has no async version
  result = await asyncio.to_thread(blocking_call, arg)
  ```
  Verify: run with `PYTHONASYNCIODEBUG=1` — no "Executing <Task> took X seconds" warnings; p95 stays flat as concurrency rises from 1 to 50.
- **[HARDEN] Hoist per-request setup to module scope** — compiling a regex,
  building a client, loading a model, or reading a config file per request is
  pure repeated cost.
  ```python
  PATTERN = re.compile(r"...")        # once at import, not inside the handler
  ```
  Verify: `py-spy record` — the setup frame disappears from the flame graph.

## Node / Hono

```bash
node --cpu-prof --cpu-prof-dir=./prof dist/server.js   # open the .cpuprofile in Chrome DevTools
npx clinic flame -- node dist/server.js
```

- **[BLOCKER] Nothing synchronous on the request path** — Node runs your code on
  one thread. `fs.readFileSync`, `crypto.pbkdf2Sync`, a synchronous zlib call, or
  `JSON.parse` on a multi-megabyte payload freezes every concurrent request for
  its whole duration.
  ```ts
  // wrong
  const tpl = fs.readFileSync("./template.html", "utf8")   // inside the handler
  // right — read once at startup, or await the async API
  const tpl = await fs.promises.readFile("./template.html", "utf8")
  ```
  Verify: instrument with `monitorEventLoopDelay` from `perf_hooks` — p99 delay stays under 100ms during a load run.
  ```ts
  const h = monitorEventLoopDelay({ resolution: 10 }); h.enable()
  setInterval(() => console.log("loop p99 ms", h.percentile(99) / 1e6), 10_000)
  ```
- **[HARDEN] Move genuine CPU work off the main thread** — image resizing,
  large-document parsing, and hashing belong in `worker_threads` or a job
  worker, not in the handler.
  Verify: event-loop delay stays flat while the CPU task runs.

## Swift

Instruments: **Time Profiler** for CPU, **Hangs** for main-thread stalls,
**Allocations** for growth. Profile the Release configuration; a Debug build's
numbers are meaningless.

- **[BLOCKER] The main thread does UI and nothing else** — JSON decoding, disk
  reads, image decoding, and Core Data fetches on the main actor freeze the
  frame. iOS reports anything over 250ms as a hang.
  ```swift
  // wrong
  let items = try JSONDecoder().decode([Item].self, from: data)   // in a @MainActor context
  // right
  let items = try await Task.detached(priority: .userInitiated) {
      try JSONDecoder().decode([Item].self, from: data)
  }.value
  ```
  Verify: Instruments **Hangs** reports zero hangs over 250ms during the flow; `MetricKit` `MXHangDiagnostic` stays empty in the field.
- **[HARDEN] Decode images at display size** — decoding a 4000px image into a
  60pt thumbnail costs memory and main-thread time proportional to the source,
  not the destination. Downsample with `CGImageSourceCreateThumbnailAtIndex` or
  let the async image loader do it.
  Verify: Allocations — peak memory during a scroll of an image list stays flat instead of climbing with row count.

## Do not

- Profile a Debug or dev build and report the result. Optimizer off, assertions
  on, different answer entirely.
- Chase allocations before you have a CPU profile. Allocation count is a proxy;
  wall-clock is the thing the user feels.
- Add concurrency to a CPU-bound path. More workers than cores makes it slower,
  and in Python the GIL means threads buy nothing for pure CPU work.
