# Data Layer Playbook

Default data layer is Postgres + Drizzle, on a managed host (Neon) or your
Coolify/VDS Postgres. The data layer is pluggable — whatever you pick must
satisfy the swap contract below.

For server-side Database and Redis **rules**, see `backend.md`'s Database and
Redis sections — this doc only adds the **menu of options** (host, ORM,
on-device, cache, jobs). It links those rules, it does not restate them.

## Choosing ORM + DB host + on-device store

- **ORM** → **Drizzle** (default) or Prisma; Kysely for raw-SQL builders.
- **Managed Postgres host** → **Neon** (default, serverless/branching) or your
  own Coolify/VDS Postgres; Supabase for Postgres+BaaS; Turso for edge SQLite;
  PlanetScale for scalable MySQL/Postgres.
- **On-device / offline** → none (server-only, default); **Expo SQLite** for a
  local cache; WatermelonDB/PowerSync for offline-first sync.
- **Cache** → **Redis** (self-host) or **Upstash** (serverless).
- **Background jobs** → **Trigger.dev** (default), QStash, Inngest, or BullMQ.

## Data Swap Contract

Every data-layer choice must satisfy these:

1. Schema changes go through committed migrations, run before production deploy.
2. Index foreign keys, lookup fields, and uniqueness constraints; use
   transactions for multi-table writes.
3. DB/Redis secrets live in server env only; never in the Expo bundle.
4. On-device secrets go in `expo-secure-store`, never in an on-device DB or MMKV.
5. The on-device DB is a cache/offline copy; the server DB is the source of
   truth. Reconcile via sync, don't trust device state for authorization.

## Add a data option

Copy this skeleton into a section and fill it in:

```markdown
# Data Playbook: <Option>

Use <Option> when: <selection criteria>.

## Setup
- Package/CLI, connection, migration command (if applicable).

## Contract compliance
- Restate the 5 swap-contract invariants that apply; confirm each is met.

## Official References
- Docs, llms.txt, and skill (from the LLM Docs Registry).
```

## Playbooks

Load the option's llms.txt before implementing. Skills/URLs live in the LLM Docs
Registry in `skills.md`.

### ORM

- **Drizzle** (default) — schema-in-TS, thin SQL-like queries, serverless-ready.
  No vendor skill; llms.txt https://orm.drizzle.team/llms.txt
- **Prisma** (strong-alt) — generated client, Studio, broad DB support.
  Skill `prisma/skills`; llms.txt https://www.prisma.io/docs/llms.txt
- **Kysely** (situational) — type-safe raw-SQL query builder. llms.txt https://kysely.dev/llms.txt

### Managed Postgres / DB host

- **Neon** (default) — serverless Postgres with branching. Skill `neondatabase/agent-skills`; llms.txt https://neon.com/llms.txt
- **VDS Postgres** — your own Postgres on Coolify/VDS (default self-host).
- **Supabase** — Postgres + auth + storage + realtime. Skill `supabase/agent-skills`; llms.txt https://supabase.com/llms.txt
- **Turso / libSQL** — SQLite at the edge or synced. llms.txt https://docs.turso.tech/llms.txt
- **PlanetScale** — scalable MySQL (Vitess) or managed Postgres. Skill `planetscale/database-skills`; llms.txt https://planetscale.com/llms.txt

### On-device / offline

- **Expo SQLite** (default local cache) — works with Drizzle. Skill `expo/skills`; llms.txt https://docs.expo.dev/llms.txt
- **op-sqlite** (situational) — fastest RN SQLite (JSI) with SQLCipher/libSQL. Check https://github.com/OP-Engineering/op-sqlite.
- **WatermelonDB** (situational) — large offline-first RN apps with a reactive store + sync. Check https://watermelondb.dev.
- **PowerSync** (situational) — bidirectional offline sync between on-device SQLite and Postgres. llms.txt https://docs.powersync.com/llms.txt

### Cache

- **Redis** (self-host) — cache/queue/rate-limit on Coolify/VDS. Skill `redis/agent-skills`; llms.txt https://redis.io/llms.txt
- **Upstash** (serverless) — serverless Redis + QStash. Skill `upstash/skills`; llms.txt https://upstash.com/docs/llms.txt

### Background jobs

- **Trigger.dev** (default) — durable jobs/workflows in TS. Skill `triggerdotdev/skills`; llms.txt https://trigger.dev/docs/llms.txt
- **QStash** — serverless message queue / scheduled HTTP jobs. Skill `upstash/skills`; llms.txt https://upstash.com/docs/llms.txt
- **Inngest** — event-driven durable functions. llms.txt https://www.inngest.com/llms.txt
- **BullMQ** — self-hosted Redis queues/workers in Node. llms.txt https://docs.bullmq.io/llms.txt

### RAG / vector stores

Vector store SETUP lives here; AI usage stays in `ai-features.md`.

- **pgvector** (default) — vector columns/indexes on the existing Postgres+Drizzle.
  Enable the `vector` extension, add a vector column, create an HNSW/IVFFlat index.
- **Upstash Vector** — serverless HTTP vector DB. Skill `upstash/skills`; llms.txt https://upstash.com/docs/llms.txt
- **Pinecone** — managed high-scale vector DB. Skill `pinecone-io/skills`; llms.txt https://docs.pinecone.io/llms.txt
- **Turso Vector (libSQL)** — vector columns + ANN in SQLite/libSQL. llms.txt https://docs.turso.tech/llms.txt
