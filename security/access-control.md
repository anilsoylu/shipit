# Security: Access control & data ownership

Part of `../security.md` (tags, threat model, and review method live there).

The highest-value category — most breaches here are authorization gaps, not
missing auth.

- **[BLOCKER] Authority from token + stored row, never the payload** — derive
  identity, role, and tenant from the verified token per request, and on mutation
  from the existing DB row. A `role`/`plan`/`ownerId` field in the body is a
  privesc/revenue hole.
  ```ts
  // wrong: caller asserts its own entitlement
  if (body.plan === "pro" || body.role === "admin") grant()
  // right: from server-verified state, keyed to the verified user
  const sub = await db.query.subs.findFirst({
    where: and(eq(subs.userId, userId), eq(subs.status, "active")) })
  if (sub?.tier === "pro") grant()
  ```
  Verify: grep authz decisions for `body.role|body.plan|body.tier|body.userId`; forge those fields in a payload → access still denied.
- **[BLOCKER] Scope every query by the session owner** — bind each row to the
  session-derived owner/tenant in the same `WHERE`; for a child row bind it to the
  session-scoped parent. Checking membership elsewhere then fetching by id alone
  is cross-tenant IDOR. Default-deny: no match → 404, not a global read.
  ```ts
  // wrong: fetched by id; ownership checked somewhere else (or not at all)
  await db.query.tasks.findFirst({ where: eq(tasks.id, taskId) })
  // right: bind to the session-derived owner/parent in one WHERE
  await db.query.tasks.findFirst({
    where: and(eq(tasks.id, taskId), eq(tasks.orgId, session.orgId)) })
  ```
  Verify: user A (org1) requests org2's id → expect 404; grep handlers for `findFirst`/`.where(eq(` on a path-param id with no owner/parent `eq`.
- **[BLOCKER] Same authz on create AND update** — the classic hole is "CREATE
  validates, PATCH doesn't": a user creates a valid row, then PATCHes it into a
  privileged or corrupt state (self-assigns role, nulls a foreign key). Authorize
  against the *existing* row's owner, not the incoming payload's. Red-team the
  *sequence*, not the endpoint.
  ```ts
  // wrong: PATCH trusts the payload; CREATE validated but UPDATE re-opens it
  await db.update(tasks).set(input).where(eq(tasks.id, id))     // input.ownerId self-assigns
  // right: authorize against the EXISTING row, then set only mutable fields
  const row = await db.query.tasks.findFirst({
    where: and(eq(tasks.id, id), eq(tasks.orgId, session.orgId)) })
  if (!row) throw forbidden()
  await db.update(tasks).set({ title: input.title }).where(eq(tasks.id, id))
  ```
  Verify: for each resource, PATCH a row you own to set `ownerId`/`role`/`tenantId` to another value → rejected; treat `ownerId`/`tenantId`/`createdAt`/`status` as immutable post-create.
- **[BLOCKER] No mass-assignment** — never spread the request body into `.set()`/
  `.insert()`. Pick mutable columns explicitly per role; sensitive columns
  (`role`, `balance`, `price`, `tenantId`) are server-set only.
  ```ts
  // wrong: body decides which columns change
  await db.update(users).set(req.body).where(eq(users.id, userId))
  // right: an explicit, role-bounded column allowlist
  await db.update(users).set({ displayName: input.displayName }).where(eq(users.id, userId))
  ```
  Verify: grep for `.set(req.body`/`.set(body`/`.values(req.body`; send an extra `role`/`price` key → ignored, not written.
- **[BLOCKER] Check the capability on every privileged endpoint, not just the session**
  — function-level authorization (BFLA): an admin/moderator/internal route that
  only checks "is logged in" lets any authenticated user hit it by guessing the
  path. Reachability is not authorization; gate the *operation*, not just the row.
  ```ts
  // wrong: authenticated, but no capability gate on an admin action
  const session = await requireSession(req)
  await db.delete(users).where(eq(users.id, targetId))
  // right: assert the capability from server-verified role/permissions
  const session = await requireSession(req)
  requirePermission(session, "users:delete")   // throws 403 unless the role grants it
  ```
  Verify: enumerate admin/internal routes, call each as a plain authed non-admin user → 403, not 200; grep privileged handlers for a role/permission check, not just an `auth()` presence check.
- **[HARDEN] Field-allowlist ≠ ownership** — *which* fields may change is not *who*
  may change them; always pair a column allowlist with the ownership check. Verify: a valid field edit on a row you don't own → 404.
- **[HARDEN] IDOR on bulk & alternate paths** — enforce the same per-item check on
  bulk/export/import endpoints and any weaker route to the same mutation, not just
  the single-item one. Verify: call the bulk/export endpoint with a mix of your ids and others' → others' rows are absent.
- **[HARDEN] Tenant-scope cache keys** — Redis keys and Next `unstable_cache`
  keys/tags embed owner/tenant, or one tenant's cached response is served to the
  next. Verify: grep `redis.set(|unstable_cache(|revalidateTag(` for keys/tags with no userId/tenant segment.

Concurrent mutation of an owned counter (credits/quota/balance) has its own
integrity trap — see the atomic-mutation rule in `webhooks-payments.md`.
