# Security playbook upgrade (from defending-code skill suite)

## Goal
Fold the distilled methodology + concrete checks from the defending-code skill
suite (threat-model / vuln-scan / triage / patch / verify / dnr-*) into shipit's
self-contained `security/` playbook. Skills are sources, not deps — no skill
install/run in the deliverable.

## Acceptance Criteria
- Router reframed as a full pipeline (threat-model -> scan -> triage -> patch ->
  verify) + D&R, with a severity-derivation rubric, disprove-first N-vote,
  numbered FP exclusion rules, recall/precision split, prompt-injection note.
- New `security/incident-response.md` (log-hunt baseline + hypothesis ledger +
  evidence ladder + IR report format), indexed in router + CLAUDE.md.
- All per-class gaps filled, house style exactly (snippet + `Verify:` + tag).

## Plan
- [ ] security/incident-response.md (new)
- [ ] security.md router: threat-model litmus+coverage, severity rubric,
      pipeline review pass, exclusion rules, prompt-injection note, index row
- [ ] CLAUDE.md: add incident-response.md to the security list
- [ ] access-control: create/update snippet; BFLA endpoint-level authz
- [ ] injection: NoSQL/ORM operator injection; LDAP note
- [ ] ssrf: DNS-rebinding mitigation, IP-encoding bypass, non-Hetzner metadata
- [ ] auth-sessions: MFA/OTP replay + single-use/expiry
- [ ] crypto-transport: token TTL/single-use/hash-at-rest; key rotation
- [ ] web-nextjs: DOMPurify snippet; RSC prop/secret leakage
- [ ] edge-proxy: concrete smuggling; rate-limit-at-edge backstop
- [ ] mobile-expo: cert pinning + WebView hardening (file is snippet-free)
- [ ] webhooks-payments: timestamp-tolerance / replay window
- [ ] storage-r2: SVG sanitization; AV/malware scan
- [ ] ai-openrouter: cross-user context isolation
- [ ] secrets-config: secret-scanning CI gate; KMS/vault
- [ ] logging-privacy: retention/access + append-only audit
- [ ] dependencies: dependency-confusion/typosquat; postinstall blocking
- [ ] dos-limits: promote money-loss cap to BLOCKER; slowloris/concurrency

## Verification Commands
- `wc -l security/*.md`; grep each new rule has a `Verify:` line and a tag.
- Router index table row count matches files in security/.

## Attempts
- Approach: distilled the defending-code skill suite via two Explore agents
  (methodology digest + current-folder gap audit), then wrote the changes to
  match house style exactly.
- Evidence: `wc`/fence-parity/tag/Verify counts pass on all 17 files; all fences
  balanced; every rule has a Verify line (per-framework 0 by design).

## Review
- Changes made:
  - security.md: threat-vs-vuln litmus + coverage invariant in the ritual;
    review pass reframed as recon->scan(recall)->triage(precision, disprove-first
    N-vote)->patch->verify; severity = preconditions x access rubric; 8 numbered
    do-not-report FP classes; AI-review injection-hygiene note; D&R pointer; index
    row for incident-response.
  - New security/incident-response.md (runtime D&R: 5 hard rules, hunt baseline +
    hypothesis ledger, respond blast-radius, response-plan format).
  - CLAUDE.md: incident-response.md added to the security list.
  - 15 per-class gap fills (see Plan), all house-style.
- Verification evidence: fence parity OK x17; tag/Verify counts per file sane.
- Skipped checks: none deterministic beyond structure (docs, no test suite).
- Advisor (Sol) review: 20 findings relayed to user, "Hepsi" approved. All 20
  applied in a second pass:
  - Correctness (9): SSRF undici connect.lookup pin (SNI preserved); crypto atomic
    DELETE ... RETURNING + high-entropy-vs-OTP hashing nuance; edge dropped the
    byte-broken smuggling illustration for "reject both CL+TE"; NoSQL query-by-email
    then verify hash; IR verdict ladder split into activity/vulnerable/impact +
    corroborated "did it succeed"; severity reframed exploitability × impact with
    tag-selection rule; MFA separated TOTP timestep-replay from email/SMS challenge
    lifecycle, dropped account-lock DoS; audit "tamper-resistant" vs true
    tamper-evidence; AI-review overclaim softened (all repo bytes untrusted).
  - DISCUSS/OVER-ENG (framework-neutral, no web search): access-control BFLA now
    requirePermission(); mobile pinning made threat-model-dependent + backup pin;
    dos cost-cap BLOCKER made conditional (unauth/automated → BLOCKER else HARDEN);
    N-vote softened to 1 verifier default, escalate to 3 on highest-stakes;
    propose-until-approved with reversible-vs-destructive split; dedup (OTP lifecycle
    in auth, crypto storage in crypto; slowloris authoritative in dos, edge refs it).
  - STYLE sweep: IR schema-exemption note; added // wrong to storage-SVG and
    web-nextjs DOMPurify; imperative rule titles; one-sentence rationales; runnable
    Verify lines (logging grep, secrets scanner fixture, edge red-team).
- Post-fix verification: fence parity balanced ×17; rules==Verify on every file
  (per-framework exempt reference matrix; ssrf's +1 is the image-optimizer surface).
- Remaining blockers: none.
