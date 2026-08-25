# Performance: Cache strategy

Part of `../performance.md` (loop, budgets, severity legend, and triage rules
live there). `../data.md` picks the cache *product* (Redis on Coolify, Upstash);
this file decides what to cache, where, and how it gets invalidated.

The theme: never compute the same answer twice. The failure mode is not a slow
cache, it is a cache that returns the wrong thing or collapses the moment it
expires.

Budget: hit rate > 80% on any read-heavy endpoint you bothered to cache. Below
that the cache is costing you a round trip and buying nothing.

## Pick the layer

Cheapest first. Each layer down is faster but serves fewer users.

| Layer | Serves | TTL shape | Use for |
| --- | --- | --- | --- |
| CDN / edge (`Cache-Control`) | everyone | seconds to hours | public pages, static JSON, images |
| Framework cache (Next.js `use cache` / `unstable_cache`) | everyone | tag-invalidated | rendered pages, fetch results |
| Redis | all app instances | seconds to days | shared computed results, sessions, rate counters |
| In-process (LRU map) | one instance | seconds | hot config, JWKS, feature flags |
| Client (TanStack Query, SWR) | one user | `staleTime` | data the user just fetched |

Rule of thumb: cache as close to the user as the data's ownership allows.
Public and identical for everyone goes to the CDN. Per-user goes no further out
than Redis keyed by user id, or the client.

## Rules

- **[BLOCKER] Never put a per-user response in a shared cache** — one wrong
  `Cache-Control: public` on an authenticated route serves user A's dashboard to
  user B from the CDN. This is a data leak with a performance excuse.
  ```ts
  // wrong — authenticated JSON, cached by every proxy on the path
  res.setHeader("Cache-Control", "public, s-maxage=300")
  // right
  res.setHeader("Cache-Control", "private, no-store")
  ```
  Verify: `curl -I` an authenticated endpoint — no `public` and no `s-maxage`; log in as a second user and confirm a fresh response. The security half of this is `../security/web-nextjs.md`.
- **[BLOCKER] Defend against the stampede** — when a hot key expires, every
  in-flight request misses at once and they all hit the database together. The
  cache does not fail gradually; it fails all at once, under peak load.
  Two fixes, use both: jitter the TTL so keys do not expire in lockstep, and
  single-flight the recompute so only one caller does the work.
  ```go
  // right — Go, golang.org/x/sync/singleflight
  v, err, _ := group.Do(key, func() (any, error) { return loadFromDB(ctx, key) })
  ```
  ```ts
  // right — Node/Hono: one in-flight promise per key
  const inflight = new Map<string, Promise<T>>()
  function once(key: string, fn: () => Promise<T>) {
    let p = inflight.get(key)
    if (!p) { p = fn().finally(() => inflight.delete(key)); inflight.set(key, p) }
    return p
  }
  ```
  ```ts
  // right — jittered TTL
  await redis.set(key, value, { ex: 300 + Math.floor(Math.random() * 60) })
  ```
  Verify: fire 200 concurrent requests for one cold key — `pg_stat_statements.calls` for the underlying query increases by 1, not by 200.
- **[BLOCKER] Invalidate on write, not on hope** — a TTL is a guess about
  staleness, not an invalidation strategy. Anything the user can edit must be
  invalidated by the write path that edits it.
  ```ts
  // right — Next.js: tag on read, invalidate on write
  const getPost = async (id: string) => { "use cache"; cacheTag(`post:${id}`); return db.query... }
  // in the server action / route handler that updates it
  revalidateTag(`post:${id}`)
  ```
  On older Next versions the same shape is `unstable_cache(fn, keys, { tags })`
  plus `revalidateTag`. Outside Next, delete the Redis key in the same
  transaction boundary as the write.
  Verify: edit a record, reload immediately — the new value appears without waiting out the TTL.
- **[HARDEN] Design the key, and version it** — a key must contain every input
  that changes the answer (tenant, locale, feature flag, schema version). A key
  that omits one serves cross-tenant or cross-locale garbage; a key with no
  version prefix means a shipped format change reads old blobs.
  ```ts
  const key = `v2:post:${orgId}:${postId}:${locale}`   // bump v2 to invalidate the whole class
  ```
  Verify: change the cached object's shape, bump the prefix, deploy — no deserialization errors in logs. Never deserialize an untrusted cached blob (`../security/injection.md`).
- **[HARDEN] Cache misses too (negative caching)** — a key that does not exist is
  a database hit every single time, and a 404-scanning bot turns that into a
  free load test. Store a tombstone with a short TTL.
  ```ts
  await redis.set(key, NOT_FOUND, { ex: 30 })          // shorter than the positive TTL
  ```
  Verify: request a nonexistent id 100 times — the DB sees 1 query, not 100.
- **[HARDEN] Serve stale while you revalidate** — the user should never wait for
  a recompute they did not cause. At the HTTP layer this is one header.
  ```
  Cache-Control: public, s-maxage=60, stale-while-revalidate=600
  ```
  Verify: after the TTL expires, the next request is served in single-digit ms from stale content and the cache refreshes behind it.
- **[HARDEN] Set a client `staleTime` deliberately** — TanStack Query defaults to
  `staleTime: 0`, so every mount, window focus, and reconnect refetches. On a
  screen with five queries that is five needless round trips per focus.
  ```ts
  useQuery({ queryKey, queryFn, staleTime: 60_000 })   // pick from how fast the data actually changes
  ```
  Verify: switch away and back — the network panel shows no refetch inside the staleTime window.

## Measure

```bash
redis-cli INFO stats | grep keyspace   # keyspace_hits vs keyspace_misses
```

Hit rate = `hits / (hits + misses)`. Below 80%, the TTL is too short, the key is
too specific, or the data was never hot enough to cache. Delete the cache rather
than keep a layer that only adds a round trip.

## Do not

- Cache to hide an N+1 or a missing index. You are paying memory to avoid fixing
  the thing (`queries.md`), and the first cold cache brings the whole problem
  back at the worst moment.
- Set a long TTL on data the user can edit. They will edit it, see the old
  value, and edit it again.
- Cache the cheap wrapper instead of the expensive call. Profile which layer the
  time is in before choosing where the cache goes.
