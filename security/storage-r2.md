# Security: Storage / Cloudflare R2

Part of `../security.md` (tags, threat model, and review method live there).

- **[BLOCKER] Sign a server-derived key, never a client-supplied path** — the object
  key comes from an ownership lookup; if the client names the key it swaps in
  someone else's object (IDOR).
  ```ts
  const file = await db.query.files.findFirst({ where: eq(files.id, fileId) })
  if (!file || file.ownerId !== session.userId) throw forbidden()          // authz BEFORE signing
  const url = await getSignedUrl(r2,
    new GetObjectCommand({ Bucket: "private", Key: file.key }),            // key from DB, not the request
    { expiresIn: 120 })
  ```
  Verify: mint a URL for your file, then request another user's id → 403; a signed URL for object A cannot GET object B.
- **[HARDEN] Bucket is private** — no `r2.dev` and no public custom domain; all reads
  go through signed URLs or an authorizing Worker. Verify: curl the object's public/`r2.dev` URL → 401/403.
- **[HARDEN] Constrain uploads at presign time** — `createPresignedPost` with a
  `content-length-range` + fixed `Content-Type` + per-user key prefix
  (`u/${userId}/…`). Verify: a presigned upload of 2x the size limit or wrong content-type → rejected.
- **[HARDEN] Validate the bytes, not just the declared type — one layer, not proof**
  — a client-declared `Content-Type`/extension lies; on accept, sniff the magic
  bytes and match them to an allowlist of expected types, and re-encode images
  through a processor. Sniffing alone is not safety proof (polyglots exist) — pair
  it with the presign constraints above and the serving rule below. Verify: upload a `.png` whose leading bytes are HTML/script → rejected by the magic-byte check.
- **[HARDEN] Serve user files from a cookieless domain, or force download** — an
  uploaded HTML/SVG served from the app origin is stored XSS; set
  `Content-Disposition: attachment` and a non-renderable `Content-Type`. Verify: upload an `.html`, open its URL → it downloads, does not execute on your origin.
- **[HARDEN] Sanitize or neutralize active-content uploads (SVG, HTML, PDF)** — an
  SVG can carry `<script>`/`onload`; strip it on ingest or serve it as an inert type,
  and AV-scan files shared between users before authorizing the download.
  ```ts
  import DOMPurify from "isomorphic-dompurify"
  // wrong: store/serve the uploaded SVG bytes as-is -> stored XSS on first view
  await putObject(key, svg, { ContentType: "image/svg+xml" })
  // right: sanitize on ingest (or serve inert per the attachment rule above)
  const clean = DOMPurify.sanitize(svg, { USE_PROFILES: { svg: true } })   // drops script/onload
  ```
  Verify: upload an SVG containing `<script>alert(1)</script>` → served bytes have no script (or it downloads as an attachment); a known EICAR test file → rejected/quarantined.
