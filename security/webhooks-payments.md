# Security: Webhooks & payments

Part of `../security.md` (tags, threat model, and review method live there).

Signed-webhook handling is baseline (`backend.md`); raw-body access per framework
is in `per-framework.md`. The traps:

- **[BLOCKER] Verify against the raw bytes the framework never parsed** — the
  signature is an HMAC of the exact payload; a parser that decodes first (then you
  re-`stringify`) changes bytes and lets forged events through.
  ```ts
  const event = stripe.webhooks.constructEvent(
    req.rawBody, req.headers["stripe-signature"]!, process.env.STRIPE_WEBHOOK_SECRET!)
  ```
  Verify: `stripe trigger` a real event → verifies; swap `req.rawBody` → `JSON.stringify(req.body)` → `constructEvent` throws.
- **[BLOCKER] RevenueCat auth is a static header secret, not an HMAC** — there is no
  signature; compare the `Authorization` header in constant time, or anyone POSTs
  fake purchase events and self-grants entitlements.
  ```ts
  const got = Buffer.from(req.headers.authorization ?? "")
  const want = Buffer.from(process.env.RC_WEBHOOK_SECRET!)
  if (got.length !== want.length || !timingSafeEqual(got, want)) return reply.code(401).send()
  ```
  Verify: POST with a wrong/missing `Authorization` → 401, no entitlement row written.
- **[BLOCKER] Mutate shared counters atomically, not read-modify-write** — credits,
  quotas, coupon redemptions, and balances double-spend under concurrent requests
  when you SELECT, compare in app code, then UPDATE (TOCTOU). Make the check part of
  the write (conditional UPDATE) or take a row lock; enforce one-time redemption
  with a unique constraint. Same idempotency principle as the webhook dedup below,
  and it applies to AI credit counters (`ai-openrouter.md`) and any quota — not
  payments alone.
  ```ts
  // wrong: two concurrent requests both pass the check
  const w = await db.query.wallets.findFirst({ where: eq(wallets.userId, uid) })
  if (w!.credits >= cost) await db.update(wallets).set({ credits: w!.credits - cost })...
  // right: atomic conditional decrement, 0 rows affected -> reject
  const r = await db.update(wallets)
    .set({ credits: sql`${wallets.credits} - ${cost}` })
    .where(and(eq(wallets.userId, uid), gte(wallets.credits, cost)))
  if (r.rowCount === 0) throw insufficient()
  ```
  Verify: fire two concurrent spends against a wallet holding exactly one unit → one succeeds, one 4xx, final balance never negative; redeem the same coupon twice concurrently → exactly one grant.
- **[BLOCKER] Enforce allowed state transitions; multi-step in one transaction** —
  a status field is not a free-for-all: reject transitions that skip steps or
  replay a completed flow (a `pending → paid` a second time, a `refunded → paid`,
  acting on an order between soft-delete and hard-delete). Gate the transition in
  the same conditional write that flips the status, and wrap multi-table steps in a
  transaction so a partial failure rolls back instead of stranding half-applied
  state.
  ```ts
  // wrong: set status unconditionally, side effects outside a tx
  await db.update(orders).set({ status: "paid" }).where(eq(orders.id, id))
  await grantAccess(id)                                   // runs even on a replayed event
  // right: only from the expected prior state, all-or-nothing
  await db.transaction(async (tx) => {
    const r = await tx.update(orders).set({ status: "paid" })
      .where(and(eq(orders.id, id), eq(orders.status, "pending")))   // 0 rows -> already handled
    if (r.rowCount === 0) return                          // idempotent no-op, not a double-grant
    await grantAccess(id, tx)
  })
  ```
  Verify: replay a completed payment event → no second grant; force step 2 of a 3-step flow to throw → step 1 is rolled back, not left applied.
- **[BLOCKER] The server computes money — never trust a client amount or quantity**
  — recompute the charge from a trusted unit price (DB/config) × a bounded quantity;
  reject negative, zero, non-integer, and overflow quantities at the Zod boundary
  (a negative qty flips a charge into a credit). Audit `z.coerce.number()` — it
  turns `"1e999"`/`" 1 "`/`true` into numbers.
  ```ts
  // wrong: client sends the amount, or an unbounded qty
  const amount = body.amount                              // attacker sets 1
  // right: server-side price, bounded integer qty
  const Body = z.object({ qty: z.number().int().min(1).max(100) }).strict()
  const price = await priceFor(sku)                       // from DB, not the request
  const amount = price * Body.parse(body).qty
  ```
  Verify: POST a `qty` of `-1`, `0`, `1e999`, or an `amount` override → 400 / ignored; the charged total always equals server price × qty.
- **[BLOCKER] Idempotency key on client-initiated payment creation** — webhook dedup
  stops duplicate *processing*, not duplicate *charges*: a double-clicked or retried
  checkout POST creates two PaymentIntents. Derive a stable key from the user +
  operation (e.g. cart/order id) and pass it as Stripe's `idempotencyKey`, so a
  retry returns the same intent instead of a second charge.
  ```ts
  await stripe.paymentIntents.create(
    { amount, currency, customer }, { idempotencyKey: `pi:${userId}:${orderId}` })
  ```
  Verify: fire the create endpoint twice for the same order → one PaymentIntent in Stripe, not two.
- **[BLOCKER] Revoke on refund, dispute, and cancel** — grant-only logic lets a
  buyer pay, refund/chargeback, and keep access. Handle `charge.refunded` /
  `charge.dispute.created` / `customer.subscription.deleted` (RevenueCat:
  `CANCELLATION`/`EXPIRATION`/`REFUND`). Verify: refund a test charge → entitlement flips to revoked before the next authorized request.
- **[HARDEN] Bind the payment to your user server-side** — resolve identity via
  `client_reference_id`/`metadata.userId` or your Stripe-customer↔user table; never
  grant by an email read from the body. Verify: a webhook with a spoofed email grants nothing.
- **[HARDEN] Dedup in the same transaction as the side effect** — insert `event.id`
  under a unique constraint and grant in one tx; check-then-write races on
  concurrent retries and double-grants. Verify: replay the same webhook id twice concurrently → exactly one grant.
- **[HARDEN] Enforce the signature timestamp window** — keep `constructEvent`'s default
  (~5 min) tolerance so a captured request can't be replayed later, and dedup `event.id`
  so even an in-window replay grants once; a hand-rolled verifier or a raised tolerance
  re-opens replay. Verify: replay a captured webhook 10 min later → rejected on timestamp; replay within the window → deduped to a single grant.
