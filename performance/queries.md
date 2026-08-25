# Performance: Database queries

Part of `../performance.md` (loop, budgets, severity legend, and triage rules
live there). Postgres is the default; the diagnosis is the same whichever client
you use, so each rule shows the fix in Drizzle/Prisma (TS), sqlc/pgx (Go), and
SQLAlchemy (Python).

The theme: work that grows with row count. Every rule here passes on a dev
database with 20 rows. Seed production-shaped data before you judge anything.

Budget: single query p95 < 50ms; anything over 200ms gets an `EXPLAIN`.

## Measure first

```sql
-- the 20 queries that actually cost you time (needs pg_stat_statements)
SELECT calls,
       round(mean_exec_time::numeric, 2)  AS mean_ms,
       round(total_exec_time::numeric)    AS total_ms,
       query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

Sort by `total_exec_time`, not `mean_exec_time`: a 3ms query called 40,000 times
per minute outranks a 900ms report nobody opens. High `calls` with low `mean_ms`
on a per-row-shaped query is the N+1 signature.

Then explain the top offender with real parameters:

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

Read three things: the top-level `actual time`, any `Seq Scan` over a large
table, and `rows` estimated vs actual. An estimate off by 100x means the planner
is working from stale statistics — `ANALYZE <table>` before concluding anything
else.

To catch slow queries in a running app rather than hunting them, enable
`auto_explain` with `auto_explain.log_min_duration = 200ms`.

## Rules

- **[BLOCKER] Never query inside a loop (N+1)** — one query per row turns a
  20-row page into 21 round trips and a 2000-row job into an outage. Fetch the
  relation in one statement.
  ```ts
  // wrong — Drizzle
  const posts = await db.select().from(postsT).limit(20)
  for (const p of posts) p.author = await db.select().from(users).where(eq(users.id, p.authorId))
  // right — Drizzle relational query, one round trip
  const posts = await db.query.posts.findMany({ with: { author: true }, limit: 20 })
  ```
  ```python
  # wrong — SQLAlchemy lazy load fires per row
  posts = session.scalars(select(Post).limit(20)).all()
  names = [p.author.name for p in posts]
  # right
  posts = session.scalars(select(Post).options(selectinload(Post.author)).limit(20)).all()
  ```
  ```go
  // wrong — one call per id in a range loop
  // right — sqlc query with `WHERE id = ANY($1::uuid[])`, one call
  authors, err := q.ListUsersByIDs(ctx, authorIDs)
  ```
  Verify: hit the endpoint once, then `SELECT calls FROM pg_stat_statements WHERE query LIKE '%users%'` — the per-row query's `calls` increments by 1, not by the page size.
- **[BLOCKER] Index every column you filter, join, or sort on** — a `Seq Scan`
  over a growing table is a time bomb: fast at 10k rows, a timeout at 5M. Order
  a composite index by equality columns first, then the range or sort column.
  ```sql
  -- query: WHERE org_id = $1 ORDER BY created_at DESC LIMIT 20
  CREATE INDEX CONCURRENTLY idx_posts_org_created ON posts (org_id, created_at DESC);
  ```
  `CONCURRENTLY` because a plain `CREATE INDEX` takes a write lock on the table
  for the duration — acceptable in a dev DB, an outage in production.
  Verify: re-run `EXPLAIN (ANALYZE, BUFFERS)` — plan changes from `Seq Scan` to `Index Scan`, `actual time` drops below 50ms, and `shared read` blocks drop by an order of magnitude.
- **[BLOCKER] Bound every result set** — a query with no `LIMIT` returns whatever
  the table holds today, which is not what it held when you wrote it. This is
  the same clamp the API layer already owes you (`../security/dos-limits.md` covers
  the attacker-driven half); here it is about your own growth.
  Verify: grep for `findMany(`/`select(` with no `limit`; each one either has a bound or a written reason.
- **[BLOCKER] Paginate by keyset, not by OFFSET** — `OFFSET 10000` makes Postgres
  read and discard 10,000 rows. Page 1 is instant and page 500 times out.
  ```ts
  // wrong
  db.select().from(posts).orderBy(desc(posts.createdAt)).limit(20).offset(page * 20)
  // right — cursor is the last row's sort key
  db.select().from(posts)
    .where(lt(posts.createdAt, cursor))
    .orderBy(desc(posts.createdAt)).limit(20)
  ```
  Verify: time page 1 and page 500 — with keyset both land within the same few ms; with OFFSET page 500 is visibly slower.
- **[HARDEN] Select the columns you use** — `SELECT *` drags TOASTed text and
  JSON columns over the wire and blocks index-only scans. Naming the columns can
  turn an `Index Scan` plus heap fetch into an `Index Only Scan`.
  Verify: `EXPLAIN (ANALYZE, BUFFERS)` shows `Index Only Scan` with `Heap Fetches: 0`.
- **[HARDEN] Do not `COUNT(*)` a large table for a UI badge** — an exact count
  scans the whole table or index every time. Use `LIMIT n+1` to answer "is there
  another page", or an approximate count from `pg_class.reltuples` for a
  headline number.
  Verify: the count query disappears from the top of `pg_stat_statements`.
- **[HARDEN] Set a statement timeout and size the pool deliberately** — without a
  timeout, one pathological query holds a connection until it finishes; with an
  oversized pool, Postgres context-switches instead of working. Start near
  `(cores * 2) + effective_spindles` per instance and measure.
  ```ts
  // right — per-connection guard, set at pool creation
  options: "-c statement_timeout=5000"
  ```
  Verify: run a deliberately slow query — it is cancelled at the timeout instead of pinning a connection; pool saturation under load test stays below 80%.

## Do not

- Add an index per query without checking existing ones. Every index costs write
  throughput and disk, and a composite index already covers its leading prefix.
- Trust an `EXPLAIN` without `ANALYZE`. Without it you are reading the planner's
  guess, not what happened.
- Blame the database before timing the DB call separately from the handler. Half
  of "slow query" reports are serialization, connection acquisition, or an N+1
  in the ORM layer, not the query itself.
- Optimize a query the profile never surfaced. Sort by `total_exec_time` and
  start at the top.
