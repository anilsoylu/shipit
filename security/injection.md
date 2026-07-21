# Security: Injection & unsafe input execution

Part of `../security.md` (tags, threat model, and review method live there).
SSRF (a server-side fetch of a user-influenced URL) lives in `ssrf.md`.

- **[BLOCKER] Parameterized queries + allowlisted identifiers** — Drizzle/`sql`
  interpolation is parameterized; `sql.raw()` and string concatenation are the
  escape hatch. Columns, tables, sort, and direction can't be parameterized — map
  user input through a fixed allowlist of real column refs.
  ```ts
  // wrong: user string becomes a column/direction
  const rows = await db.select().from(posts).orderBy(sql.raw(req.query.sort))
  // right: allowlist maps input -> a real column ref
  const cols = { created: posts.createdAt, title: posts.title } as const
  const col = cols[req.query.sort as keyof typeof cols] ?? posts.createdAt
  const rows = await db.select().from(posts).orderBy(desc(col))
  ```
  Verify: grep `sql.raw(`/`$dynamic(`/`orderBy(` fed by `req.`; never `child_process`/`eval`/shell on untrusted input; send an unknown sort → safe fallback, no SQL error leaked.
- **[BLOCKER] Coerce query fields to scalars before they reach a filter** — a JSON
  body parsed straight into a Mongo/query filter lets `{"$ne":null}` or `{"$gt":""}`
  match any row, and `$where` (plus `$function`/`$accumulate` where server-side JS is
  enabled) runs JS.
  ```ts
  // wrong: body becomes the filter — { "email": {...}, "password": { "$ne": null } }
  //        skips the password check for whatever email row it matches
  await users.findOne({ email: req.body.email, password: req.body.password })
  // right: validate to scalars, query by email only, verify the hash in app code
  const { email, password } = Login.parse(req.body)   // z.string().email(), never z.any()
  const u = await users.findOne({ email: email.toLowerCase() })
  if (!u || !(await argon2.verify(u.passwordHash, password))) throw unauthorized()
  ```
  Reject Mongo `$`-prefixed keys. LDAP filters have the same shape (`*)(uid=*` escapes the filter) — escape with the driver's filter-escaper, never string-concat. Verify: send `{"$ne":null}` / `{"$gt":""}` as a query field → rejected at validation, no auth bypass or full-table match.
- **[BLOCKER] No shell string with user input — pass argv, never a command line** —
  building a shell command from user input (filenames, ids, a URL handed to
  ffmpeg/imagemagick/sharp/git/pdf tooling) is RCE. Use the array form with the
  binary and args separated so the shell never parses user bytes.
  ```ts
  import { execFile } from "child_process"; import { promisify } from "util"
  // wrong: user filename becomes shell syntax
  exec(`convert ${req.body.name} out.png`)          // "a; rm -rf ~", $(...), backticks all execute
  // right: argv array, no shell, validated path
  await promisify(execFile)("convert", [safePath, "out.png"])   // execFile does not spawn a shell
  ```
  Go: `exec.Command("convert", safePath)` — never `sh -c "…"`. Python: `subprocess.run(["convert", path], shell=False)`. Verify: grep `exec(|execSync(|"sh"|os.system(|shell=True|exec.Command(.*"-c"` fed by request/filename data → none; a filename `a; id` runs nothing.
- **[BLOCKER] Never deserialize untrusted input with a code-executing loader** —
  JSON only for external data, then validate; a function/object-constructing
  deserializer on attacker bytes is RCE.
  ```ts
  // wrong
  nodeSerialize.unserialize(cookie)   // node-serialize runs embedded functions -> RCE
  yaml.load(input)                     // js-yaml v3 resolves !!js/function tags -> RCE (v4 load is safe)
  // right
  const obj = JSON.parse(input)        // then Zod-validate
  ```
  Python: never `pickle.loads`/`yaml.load(..., Loader)` on untrusted input — use `json`/`yaml.safe_load`. Redis: values you wrote are trusted, but never `eval`/`unserialize` a cached blob.
  Verify: grep `unserialize(|yaml.load(|pickle.loads(|vm.runInContext(` on external-input paths → only `JSON.parse`/`safe_load` reach untrusted data.
- **[BLOCKER] Disable external entities in XML parsing (XXE)** — any XML from SAML,
  RSS/sitemap, SVG/Office uploads, or XML webhooks: turn off DTD + external
  entities, or the parser reads local files and makes server-side requests.
  ```ts
  // wrong: entity expansion on -> file read / SSRF
  libxml.parseXml(xml, { noent: true })   // noent EXPANDS entities
  // right
  libxml.parseXml(xml, { noent: false, nonet: true })
  // fast-xml-parser: leave processEntities off; never a DTD-resolving parser on untrusted XML
  ```
  Python: use `defusedxml`, not `xml.etree`/`lxml` defaults. Verify: grep `parseXml|DOMParser|etree|lxml|SAXParser` on request/upload data; feed an XML referencing an external entity in staging → not resolved.
- **[BLOCKER] Path traversal & zip-slip** — never build a filesystem path or an
  archive-extraction target from user input; resolve and confirm it stays under
  the base dir.
  ```ts
  // wrong: a "../../etc/passwd" filename escapes the base
  fs.readFile(path.join(UPLOAD_DIR, req.body.filename))
  // right: basename + resolve + containment check
  const p = path.resolve(UPLOAD_DIR, path.basename(req.body.filename))
  if (!p.startsWith(path.resolve(UPLOAD_DIR) + path.sep)) throw forbidden()
  ```
  Zip extraction: validate each entry stays under the target before writing. (R2 keys are owner-scoped in `storage-r2.md`; archive-size bounds in `dos-limits.md`.) Verify: grep `path.join(|readFile(|createReadStream(|extract(` fed by `req.`/upload names; a `../` filename → rejected.
- **[HARDEN] No server-side template injection (SSTI)** — a user string is data,
  never the template; rendering user input as a template is RCE in Handlebars/EJS/
  Nunjucks (React Email/JSX is safe — data, not template).
  ```ts
  // wrong: user input IS the template
  Handlebars.compile(userInput)(data)     // also ejs.render(userInput), nunjucks.renderString(userInput)
  // right: fixed template, user input only as data
  Handlebars.compile(FIXED_TEMPLATE)({ name: userInput })
  ```
  Verify: grep `compile(|renderString(|new Function(` fed by request data → none; templates are static assets.
- **[HARDEN] Zod `.strict()` at every boundary** — request objects reject unknown
  keys (never `.passthrough()`); validate before any DB write; bound string length
  and array size; audit `z.coerce` usage. Verify: grep `z.object(` on request bodies without `.strict()`, and any `.passthrough(`; oversized array/string → 400.
- **[HARDEN] Prototype pollution** — reject `__proto__`/`constructor`/`prototype`
  keys; never deep-merge or `Object.assign` a raw body into config/query objects.
  Verify: POST a body carrying a `__proto__` key → rejected/ignored, no global mutation.
- **[HARDEN] Stored & indirect injection** — catch input safe on write but
  dangerous on later read, and injection via field names/keys/headers, not just
  values. Verify: store a payload with an injection marker, then exercise every read path that renders or queries it.
