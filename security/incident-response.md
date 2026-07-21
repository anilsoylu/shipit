# Security: Detection & response (log hunting + incidents)

Part of `../security.md` (tags, threat model, and review method live there). This
is the **runtime** class — everything else in `security/` is pre-ship; this is
what you run against logs when you suspect (hunt) or know (respond) something
happened. Findings here re-enter the pipeline: a confirmed root cause is handed
back as a triage-ready finding for `patch`.

This page departs from the attack-class schema on purpose: the five hard rules and
the format blocks below are methodology, not shippable rules, so they carry no
tag. Where a rule *is* tagged, **[BLOCKER]**/**[HARDEN]** marks how much it matters
to a sound investigation — not a pre-ship gate, since nothing here blocks a launch.

## Five hard rules (they are the discipline)

1. **Log evidence alone never confirms a vulnerability.** Three separate questions,
   three separate verdicts — don't collapse them. `matching_attack_activity` = logs
   show attack-shaped traffic; it proves someone *tried*, nothing about success or a
   real flaw. `confirmed_vulnerable` = the flaw is located in source (`file:function`)
   **and** a PoC fires against a local instance this session; the code is exploitable
   whether or not this attacker landed it. `confirmed_impact` = discriminating
   evidence (not a 200 alone) that the attack actually reached the asset — the two
   are independent, code can be vulnerable with no successful hit and a hit can
   precede the fix. `attempted_not_vulnerable` = activity seen but source shows a
   working defense and the PoC bounces. Agents love to "confirm" off an observability
   dashboard — the PoC fires or it doesn't, and that can't be talked past.
2. **Query, don't read.** Log corpora are bigger than any context window. Profile
   with `wc`/`du`/`head`/`tail`; interrogate with `grep`/`awk`/`sort`/`uniq`/
   `python3`. Never `cat` a multi-MB file into context (small <200-line alert/
   error files are exempt).
3. **Every number is a query result** — never estimate a count, victim total, or
   time window from memory; each figure traces to a command you ran.
4. **Propose first; execute only what's approved and reversible.** Produce a
   response plan (block, rotate, disable, revoke) and get explicit sign-off before
   touching production. Once approved, a *reversible* containment action (block an
   IP, disable a feature flag, revoke a session/token) can be executed; an
   *irreversible or destructive* one (delete data, wipe an account, drop a table)
   stays proposal-only regardless. Rotation of a leaked credential is reversible and
   usually the first approved step.
5. **Local only.** PoCs fire at `127.0.0.1`, never against production or a third
   party.

## Hunt (no alert in hand)

Proactive sweep of the logs looking for what got missed. Output an incident list,
not a vibe.

- **[HARDEN] Baseline before hypotheses** — a dozen cheap one-liners first, so
  "anomalous" has a definition. Time window, status histogram, top talkers vs the
  median, top routes (normalize ids: `/orders/123` -> `/orders/N`), user-agent
  inventory (flag `curl`/`python-requests`/`sqlmap`/`zgrab`), the auth picture,
  per-route response-size profile (an egress tell).
  ```bash
  awk '{print $9}' access.log | sort | uniq -c | sort -rn        # status histogram
  awk '{print $1}' access.log | sort | uniq -c | sort -rn | head # top talkers
  # normalize ids, then rank routes
  awk '{print $7}' access.log | sed -E 's#/[0-9]+#/N#g' | sort | uniq -c | sort -rn | head
  ```
  Verify: you can state the corpus's time span, request count, and top 3 routes as query output before proposing a single hypothesis.
- **[HARDEN] One hypothesis, one ledger row — no query without a hypothesis** —
  keep a written ledger; a hunt with no ledger is unauditable and gets re-run by
  hand forever. Refuted rows stay (they become `ruled_out`).
  ```markdown
  | # | Hypothesis | Query | Result | Verdict | Next pivot |
  |---|-----------|-------|--------|---------|-----------|
  | 1 | SQLi on /search | grep "UNION|%27" access.log | 41 hits from 3 IPs | supported | pivot on those IPs |
  ```
  Verify: every quantitative claim in the report has a matching ledger row with the exact query that produced it.
- **[HARDEN] Run the seed sweeps, then pivot on entities** — 5xx-by-payload,
  every-alert-verdicted, top-talker outliers, injection grammar in query strings
  (decode `%27`/`UNION`/`--`, reconstruct per-IP time-ordered), sequential-id
  enumeration at machine cadence, auth anomalies (new IP + volume), size anomalies
  (response = a multiple of the route median), quiet-hours activity. On a hit, the
  next hypotheses come from that hit's entities (IP, user, route); hunt both ways —
  how did they get here, where did they go next. **Stop** after two consecutive
  rounds surface no new entities. Verify: each incident carries a `timeline[]` and an `evidence[]` of `<source>: <pipeline> -> <result>` lines, not a prose assertion.

## Respond (a lead in hand)

You have an INC id, an alert, an IOC, or free text. Answer the incident
commander's four questions in order: **what happened -> did it succeed -> how far
did it go -> what do we do.**

- **[BLOCKER] "Did it succeed?" is answered by discriminating evidence, not status
  codes** — a wall of 401s is a failed attempt; 200s with anomalous response sizes,
  an error-then-success progression, or data-shaped bodies *indicate probable*
  success — corroborate with a second signal (the response size matching the stolen
  shape, a follow-on authenticated action, the row actually present) before calling
  it confirmed, since a 200 can be an empty result or a WAF block page. State the
  specific evidence that discriminates, or the verdict is a guess. Verify: the verdict cites the exact log lines (and their sizes/codes) that separate "tried" from "worked".
- **[BLOCKER] Quantify blast radius from queries, recover the queried list** —
  which routes/tables, how many records, over what span, how many distinct victims
  (walk the touched ids to owners through the app's own lookup). Containment and
  recovery target *the enumerated list*, never "all users" as a hand-wave. Verify: victim count and record count are each a query result; the recovery target is the id list, not a round number.
- **[HARDEN] Turn the winning query into detection** — every hunt that doesn't
  improve the alert queue gets re-run by hand next time. The Phase-2 discriminating
  query becomes alert logic; note which existing alerts fired vs were noise. Verify: the report ends with a concrete detection rule (the query + threshold) and a list of alerts that should have fired.

## Response-plan format (proposed, never executed)

```markdown
### INC-<id> <one-line title>
- Verdict & summary: 3 sentences — what happened, did it succeed, blast radius.
- Timeline: query-sourced events, earliest first.
- Containment (proposed): each action + why + what breaks if the call is wrong
  (e.g. blocking a shared NAT egress IP takes out legitimate users too).
- Eradication: the code fix at `file:function`, handed off as a triage-ready
  finding (see `../security.md` finding format) for the patch stage.
- Recovery: the enumerated victim/record list to reset/notify.
- Detection engineering: the new alert rule (query + threshold).
- Open questions: what this corpus cannot answer.
```

A negative outcome still ships a report — what the attacker tried and why it
failed feeds hardening and the `ruled_out` list.
