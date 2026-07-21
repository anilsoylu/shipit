# Security: Logging & data minimization

Part of `../security.md` (tags, threat model, and review method live there).

- **[BLOCKER] Scrub before events leave the process** — Sentry/PostHog capture
  headers, bodies, breadcrumbs, and local variables by default; strip
  auth/cookies/tokens/PII in `beforeSend`.
  ```ts
  Sentry.init({ dsn: process.env.SENTRY_DSN, sendDefaultPii: false,
    beforeSend(event) {
      const DROP = /authorization|cookie|x-api-key|token|password|secret/i
      const scrub = (o?: Record<string, unknown>) =>
        o && Object.keys(o).forEach((k) => DROP.test(k) && (o[k] = "[redacted]"))
      scrub(event.request?.headers); scrub(event.extra)
      if (event.request) { delete event.request.data; delete event.request.query_string }
      if (event.user) event.user = { id: event.user.id }        // drop email/ip/username
      return event
    } })
  ```
  Verify: trigger a captured error on an authed request → the Sentry event has no `Authorization`/cookie header, no request body, `user` is id-only.
- **[HARDEN] Redact at the logger, not the call site** — a central redaction list
  (pino `redact`, a scrub middleware) so a new `logger.info(req)` can't leak; never
  `JSON.stringify(req)` or log full bodies. Verify: grep `logger.*req|console.log(.*body` and confirm the redactor covers `authorization`, `cookie`, `password`, `token`.
- **[HARDEN] Log structured, so user input can't forge log lines** — a newline or
  control char in user input concatenated into a log message forges fake entries
  and corrupts downstream parsers/SIEM. Use the structured logger's field API
  (`logger.info({ email }, "signup")`), never string-concatenate user input into
  the message; the JSON encoder escapes CR/LF. Verify: log a value containing `\n[ERROR] forged` → it appears as one escaped JSON field, not a new log line.
- **[HARDEN] Minimize what you collect** — no email/full IP/precise location/device
  ids to PostHog; stable pseudonymous id, masked autocapture inputs
  (`data-ph-no-capture`), session-replay masking on. Verify: inspect a PostHog event → no raw email/PII; replay masks input fields.
- **[HARDEN] Route auth events to an audit trail** — session create/revoke, login
  success+failure, password/email change, account link, role change → structured
  audit events (actor, target, source ip, timestamp). Verify: a role change produces a structured audit event; failed-login spikes are queryable/alertable.
- **[HARDEN] Make the audit trail append-only and bound log retention** — no app-path
  UPDATE/DELETE touches the audit table, and it is readable only by admins; that makes
  it tamper-*resistant*. For tamper-*evidence* (detecting after-the-fact edits) you
  need more: a hash chain over rows, signed entries, or a WORM/append-only sink the
  app can't rewrite. General logs expire on a schedule so PII (tokens in query strings,
  request context) doesn't accumulate forever. Verify: `grep -rEn 'update\(audit|delete\(audit|DELETE FROM audit|UPDATE audit' src/` → none outside migrations; a retention/rotation policy exists and runs.
