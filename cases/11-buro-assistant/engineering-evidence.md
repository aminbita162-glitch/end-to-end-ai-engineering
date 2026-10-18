# PRJ-11 — Büro Assistant: Engineering Evidence

**Project ID:** PRJ-11
**Canonical name:** Büro Assistant
**Source:** https://github.com/aminbita162-glitch/buro-assistant
**Source HEAD (Phase 5 inspection):** `893db187185dbed7b4dafd461241eb78acb07f8e` (2026-10-04, commit message: "phase 9: record and freeze")
**Default branch:** `main`
**License:** Proprietary — `LICENSE` file present: "Copyright (c) 2026 Amin Azimi, AI Architect, Azimi Innovation Lab. All rights reserved." Amin Azimi owns the software, documentation, and derivatives. No redistribution right is granted to others without a separate written license. Copying source into the portfolio requires explicit owner authorization. All evidence uses SHA-pinned permalinks and attributed descriptions; no source code blocks are pasted.
**Language:** Python 3.12 (Dockerfile), 3.11+ (local)
**Framework:** FastAPI 0.128.8 + SQLAlchemy 2.0.54 + Pydantic 2.13.5 + Alembic 1.16.5
**Phase 5 inspection date (UTC):** 2026-10-18
**Independence:** Source/documentation observations only; no independent third-party verification claimed.
**Independence level:** Source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 not achieved — no CI workflow present (0 CI runs). Test count (856) is REPORTED by author in `docs/TEST_HOUSE.md` and `docs/releases/final-phase-9.md`, and confirmed by static source count (856 `def test_`/`async def test_` functions across 28 test files — matching). Not independently executed.

---

## 1. Context and Scope

Büro Assistant is a self-hosted, multi-tenant office mail desk that polls a mailbox, classifies messages with tenant rules before any model call, drafts receipt replies from approved templates, routes work to an operator approval queue, and maintains an append-only audit log with a SHA-256 hash chain. It is explicitly **Phase 9 closed** — all nine phases of the final build contract are complete. Commercialization (real payment vendor + market launch) is explicitly not done.

**Status (OBSERVED in README.md at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| System status | "1.0.0 · Phase 9 closed · self-hosted · not SaaS · payment not live" | OBSERVED |
| CI | No CI workflow (0 `.github/workflows/` files) | OBSERVED |
| Test count | 856 test functions across 28 test files | OBSERVED (static source count) |
| Release tags | 0 | OBSERVED |
| Published releases | 0 | OBSERVED |
| Deployment | Self-hosted via Docker Compose (operator-controlled) | OBSERVED |
| Payment | Not live — fake adapter when `GATEWAY_SECRET` absent | OBSERVED |
| External test script | `docs/EXTERNAL_TEST.sh` — written, not run | OBSERVED |
| Second-account run | Not done | OBSERVED |

**Non-goals (OBSERVED across README.md, DIRECTIVE.txt, final-phase-9.md, PRIVACY_DATA_MAP.md):**

- Not a hosted SaaS service — no hosted service, shared infrastructure, or SaaS offering.
- Commercialization is not done — no real payment vendor, no market launch.
- No card data stored — payment boundary uses fake adapter when `GATEWAY_SECRET` is absent.
- External test script (`docs/EXTERNAL_TEST.sh`) is written but not run by a second account.
- The second-account verification run is not done — the claim "a second account has installed and verified this" is not made.
- No Phase 10 — explicitly stated: "Do not start a phase 10."

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading all source files at HEAD SHA `893db187185dbed7b4dafd461241eb78acb07f8e` via local clone. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
[Sender's mail server]
  │  IMAP / SMTP
  ▼
[IMAP Provider / FakeProvider]  ← credentials from env only
  │  IMAPProvider: real IMAP (imap_provider.py)
  │  FakeProvider: deterministic stub (fake_provider.py)
  │  OAuth factory: OAuth mailbox port (oauth_factory.py, env-gated)
  │
  ▼
[Ingest Layer] (app/ingest/)
  │  ingest.py: idempotency key (tenant_id + provider_message_id)
  │  sender_auth.py: SPF/DKIM/DMARC (pass/fail/not_run)
  │  attachment_text.py: text extraction from allowlisted types only
  │  normalize.py: provider-neutral NormalizedMessage
  │  models.py: Message ORM (tenant_id, raw_json, provider_message_id)
  │
  ▼
[Pipeline] (app/pipeline.py)
  │  Plan capability check (Commercial Phase 4)
  │  → Amin (triage): rule engine → model call (if no rule hit)
  │  → Amilos (draft): template-based reply from approved registry
  │  → Leila (supervise): exception routing (low confidence, dissatisfied tone)
  │  → Shadow store or approval queue
  │  → Send (only when should_send == "allow")
  │
  ├─ Agents (app/agents/)
  │  amin.py: triage agent (department, action, confidence)
  │  amilos.py: draft reply agent (template-based)
  │  leila.py: supervisor agent (exception routing)
  │  rules.py: tenant rule engine (domain → subject → department, first match wins)
  │  signals.py: semantic duplicate (Jaccard ≥ 0.85) + dissatisfied tone (keyword lexicon)
  │  redact.py: PII redaction before model call (EMAIL, PHONE, CARD, SSN, IBAN)
  │  decision_hash.py: deterministic SHA-256 decision hash
  │  local_model.py: local model flag + FakeLocalModel stub
  │  fake_model.py: deterministic FakeModel for tests
  │  language.py: language detection by library
  │  urgency.py: urgency lexicon with tenant overrides
  │  thread_context.py: bounded thread context (200-char clip, last 3 messages)
  │
  ├─ Policy (app/policy/)
  │  send_decision.py: should_send → "allow" | "draft" | "shadow"
  │  shadow.py: Draft ORM + shadow mode store (store, don't send)
  │  approval.py: ApprovalQueueEntry ORM + human approval queue
  │  audit.py: AuditLogEntry ORM + append-only + SHA-256 hash chain
  │  templates.py: template registry with forbidden phrases
  │  sla.py: SLA clock from ingest time
  │  calendar.py: business-hours calendar per tenant
  │  digest.py: daily digest (shadow only)
  │
  ├─ Workers (app/workers/)
  │  queue.py: work queue with priority lanes
  │  quota.py: per-tenant daily token quota
  │  cost.py: CostRecord (model name, tokens_in, tokens_out)
  │  dlq.py: dead-letter queue + replay
  │  intake_loop.py: intake loop with secret filtering
  │  webhook_delivery.py: signed webhook delivery (HMAC-SHA256)
  │  trace.py: traces around ingest, decide, draft, send
  │
  ├─ Domain (app/domain/)
  │  apikeys.py: scoped API keys (SHA-256 hashed)
  │  buyer_features.py: buyer feature surface (31KB)
  │  subscription.py: tenant subscription record
  │  plan_rules.py: plan capability enforcement
  │  usage.py: usage events
  │  retention.py: retention field (default 180 days)
  │  sandbox.py: sandbox tenant seed
  │  export.py: tenant data export
  │  webhooks.py: webhook subscription management
  │
  ▼
[Persistence] (app/main.py → SQLAlchemy + Alembic)
  │  PostgreSQL 16 (Docker Compose: postgres:16-alpine)
  │  14 Alembic migrations (0001–0014)
  │  Tenant-scoped tables: tenants, messages, drafts, approval_queue, audit_log
  │  Every business table has tenant_id column
  │
  ▼
[Operator Desk] (app/web/desk.py + index.html)
  │  Bearer token auth (sessionStorage)
  │  textContent rendering (no innerHTML — XSS prevention)
  │  Dashboard counts, department queues, audit trail
```

### 2b. Key Architectural Decisions

1. **Rules before model** — A rule match (domain/subject/department) produces zero model tokens. The rule engine (`rules.py`) evaluates tenant rule packs before any model call. When a rule hits, the model client is not called. This is the primary cost-control and safety mechanism.

2. **Redaction before model** — PII (email, phone, credit card, SSN, IBAN) is redacted before any model call (`redact.py`). Attachment bytes are never sent to a model. The redacted text replaces PII with `[EMAIL]`, `[PHONE]`, `[CARD]`, `[SSN]`, `[IBAN]` tokens.

3. **Shadow mode** — When `shadow_mode: true` in tenant policy, drafts are stored but never sent. This lets operators validate decisions against real mail before enabling live auto-reply. Auto-reply is OFF by default (`should_send()` returns `"draft"`).

4. **Append-only audit with hash chain** — Every policy event is written to an append-only `audit_log` table. No UPDATE/DELETE path exists. Each row stores the SHA-256 of the previous row's canonical representation. `verify_chain()` detects tampering by recomputing hashes.

5. **Air-gap package** — Docker Compose runs with `LOCAL_MODEL=1` (FakeModel, no cloud model), no `GATEWAY_SECRET` (fake payment adapter), no `OPENAI_API_KEY`, no IMAP credentials. The app runs fully air-gapped.

6. **Self-hosted, not SaaS** — The database region is operator-controlled. No hosted service, no shared infrastructure, no multi-tenant cloud deployment. The operator sets `DATABASE_URL` to their own PostgreSQL instance.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Source URL | https://github.com/aminbita162-glitch/buro-assistant |
| HEAD SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| HEAD commit message | "phase 9: record and freeze — correct test badge to 856, update TEST_HOUSE, add final-phase-9.md" |
| HEAD commit date | 2026-10-04T15:32:59Z |
| Default branch | `main` |
| License | Proprietary (Copyright (c) 2026 Amin Azimi; all rights reserved; no redistribution without written license) |
| Total tracked files | 175 (excluding .git) |
| Source .py files | 62 (app/ tree) |
| Test files | 28 (`tests/test_*.py`) |
| Test functions | 856 (OBSERVED — static source count by `grep -c "def test_\|async def test_"` across all test files) |
| CI workflow | **NONE** — 0 `.github/workflows/` files |
| CI runs | **NONE** — 0 CI runs (no workflow to trigger) |
| Release tags | 0 |
| Published releases | 0 |
| Version | 1.0.0 (README badge: "version-1.0.0"; docs/releases/1.0.0.md) |
| Deployment | Self-hosted via Docker Compose (postgres:16-alpine + app) |
| Inspection method | Local clone at pinned SHA (after `git fetch origin main`) |
| Observation class | OBSERVED (all file content); OBSERVED (test count via grep); OBSERVED (no CI via GitHub API) |

**Commit history (git log --oneline -15 at HEAD):**

```
893db18 phase 9: record and freeze — correct test badge to 856, update TEST_HOUSE, add final-phase-9.md
dca23e5 phase 8: visual README — cover SVG, badges, Mermaid diagram, callout tables, looping GIF
6433902 Phase 7: air-gap docker-compose, no cloud model, no gateway secret
4ccb8e0 Phase 6: desk signals — digest shadow, semantic duplicate flag, attachment text, dissatisfied-tone flag
c0c8e11 Phase 5: extend audit log with SHA-256 hash chain
91a3c9c Phase 4: OAuth mailbox boundary — port, fake provider, factory, 17 tests
b8f180b Phase 3: local model route — flag, FakeLocalModel, pipeline routing, 15 tests, report
9552e0c Phase 3: local model route — tenant flag, FakeLocalModel, pipeline routing, 15 tests
6fe2de9 Phase 2: bounded thread context — redacted 200-char clip of last 3 thread messages in Amin prompt
05a6cc5 Phase 1: store SPF, DKIM, DMARC on inbound messages; fail blocks send, not_run passes
97dded4 Commercial phase 5: record the run and freeze
fca83ee Commercial phase 4: show plan on the desk
569731d docs: commercial-phase-3 release notes
ffa7ea5 Commercial phase 3: enforce desk, mail, and agents plans
0178f52 Commercial phase 2: payment port with fake gateway
```

---

## 4. Email/Attachment Injection Safety (Phase 5 Lens)

### 4a. Redaction Before Model Call (OBSERVED in `app/agents/redact.py`, 51 lines)

Five PII patterns are redacted before any model call:

| Pattern | Label | Regex | Example |
|---------|-------|-------|---------|
| IBAN | `[IBAN]` | `\b[A-Z]{2}\d{2}[A-Z0-9]{4}\d{7}(?:[A-Z0-9]?){0,16}\b` | DE89370400440532013000 |
| Credit card | `[CARD]` | `\b(?:\d[ -]?){13,19}\b` | 4111 1111 1111 1111 |
| SSN/National ID | `[SSN]` | `\b\d{3}[-\s]?\d{2}[-\s]?\d{4}\b` | 123-45-6789 |
| Phone | `[PHONE]` | Complex phone pattern | +1 (555) 123-4567 |
| Email | `[EMAIL]` | `[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}` | user@example.com |

**Key observation:** Order matters — more-specific patterns (IBAN, CARD, SSN) are applied before more-general ones (PHONE, EMAIL) to avoid partial matches. `redact_message()` redacts both subject and body. `redact()` returns `(redacted_text, count)`.

### 4b. Attachment Safety (OBSERVED in `app/ingest/attachment_text.py` + `app/ingest/ingest.py`)

**Attachment quarantine (OBSERVED in `ingest.py` `_attachment_state()`):**
- If no attachments → `attachment_state = "none"`.
- If any attachment has a disallowed content-type → `attachment_state = "quarantine"`.
- If all attachments are in the allowlist → `attachment_state = "clean"`.
- `ATTACHMENT_ALLOWLIST` is defined in `imap_provider.py`.

**Text extraction (OBSERVED in `attachment_text.py`):**
- Only `text/plain` and `text/csv` yield decoded strings.
- `application/pdf` is in the allowlist but returns `""` (binary parsing not performed).
- Images and binary office formats return `""`.
- **The extracted string is never passed to a model.** The docstring states: "Callers must not forward the return value to any model client."
- Attachment bytes never leave the module as anything other than a plain Python `str`.

### 4c. Sender Authentication (OBSERVED in `app/ingest/sender_auth.py`, 175 lines)

**SPF/DKIM/DMARC checks:**
- `SenderAuthResult` dataclass: `spf`, `dkim`, `dmarc` — each one of `"pass"`, `"fail"`, `"not_run"`.
- `any_fail` property: True when at least one check is `"fail"`.
- `check_sender_auth()`: performs DNS lookups (requires `dnspython`).
- If `SENDER_AUTH_ENABLED != "1"` → all checks return `"not_run"`.
- If `dnspython` is absent → all checks return `"not_run"`.
- **A fail does NOT auto-send and does NOT delete the message.** The pipeline layer enforces the send-block; this module only reads.
- Only the sender domain is checked; no message bytes leave the process.

### 4d. XSS Prevention (OBSERVED in THREAT_MODEL.md)

- `raw_json` is never rendered in the desk UI — the desk uses `textContent` only.
- Message and task fields are rendered as text, not HTML.
- The desk (`app/web/desk.py`) renders operator surface with text content injection prevention.

---

## 5. Privacy and Data Governance (Phase 5 Lens)

### 5a. GDPR Privacy Data Map (OBSERVED in `docs/PRIVACY_DATA_MAP.md`)

The source repository contains a GDPR Article 30 records-of-processing-activities document. Key findings:

| Table | Personal data | Purpose | Legal basis | Retention |
|-------|--------------|---------|-------------|-----------|
| `users` | name, email, password_hash (Argon2id) | Operator identity | Contract | Until account deleted |
| `messages` | subject_normalized, raw_json, message_id_header | Message classification | Legitimate interest | `tenants.retention_days` (default 180) |
| `drafts` | subject, body (template-generated) | Draft management | Contract | Same |
| `approval_queue` | subject, body (truncated 500 chars), resolved_by | Human review | Contract | Same |
| `audit_log` | actor (email or "system"), detail | Accountability, non-repudiation | Legal obligation | Same |
| `usage_events` | actor | Billing accountability | Contract | Same |
| `user_sessions` | token_hash (SHA-256 only) | Session auth | Contract | 24-hour TTL |
| `api_keys` | name, key_hash (SHA-256 only) | Key authentication | Contract | Until revoked |

**Key observation:** Body text of incoming messages is **not stored** in the `messages` table. It is passed through the pipeline in memory, redacted before any model call, and discarded. Only `subject_normalized` and `raw_json` (envelope headers) are persisted.

**Region choice:** The database region is operator-controlled. Buro Assistant does not operate a hosted service and makes no claim about data residency. If the operator must keep data in the EU, the operator must host the database in an EU region.

### 5b. STRIDE Threat Model (OBSERVED in `docs/THREAT_MODEL.md`)

The source repository contains a STRIDE threat model (Follow-up Phase 4, 2026-10-06). Key mitigations:

| STRIDE | Threat | Mitigation |
|--------|--------|------------|
| Spoofing | Credential stuffing | Argon2id password hashing; session rotation on every login; 10/min rate limit |
| Spoofing | API key theft | Keys stored as SHA-256 hashes only; plaintext shown once at creation |
| Spoofing | Forged webhook | HMAC-SHA256 payload signing (`X-Buro-Signature`) |
| Tampering | Audit log manipulation | Append-only table; no UPDATE/DELETE path in code |
| Tampering | Injection via email | Body and subject redacted before model call; template system enforces structure |
| Tampering | Stored XSS | `raw_json` never rendered in desk; desk uses `textContent` only |
| Repudiation | "I didn't approve that" | `resolved_by` records operator email; audit log row written |
| Info Disclosure | Cross-tenant read | Every query filters by `user.tenant_id`; desk enforces tenant scope |
| Info Disclosure | PII in model prompts | Redaction applied before every model call |
| Info Disclosure | Attachment bytes to model | Attachment bytes never included in any model prompt |
| DoS | Brute-force/flood | `slowapi` rate limiter on sensitive endpoints |
| DoS | Queue flooding | Per-tenant `queue_depth_cap` (default 500); `BackpressureError` |
| DoS | Runaway model calls | Per-tenant daily token quota (`daily_token_quota`); `QuotaExceeded` |

**Known gap (OBSERVED in THREAT_MODEL.md):** The OpenAI model API is a third-party service. Even after redaction, subject and body clips are transmitted to OpenAI's servers. Operators with data-residency requirements must evaluate whether this is acceptable or switch to an on-premises model (local model route, Phase 3). This is documented, not hidden.

---

## 6. Duplicate Task Extraction and Signals (Phase 5 Lens)

### 6a. Semantic Duplicate Detection (OBSERVED in `app/agents/signals.py`)

**Algorithm:** Jaccard similarity on word sets.
1. `_word_set(text)`: lowercase word tokens via `re.findall(r"[a-z0-9]+", text.lower())`.
2. `_jaccard(a, b)`: `len(a & b) / len(a | b)`.
3. `is_semantic_duplicate(body, candidates, threshold=0.85)`: True when Jaccard ≥ threshold for any candidate.
4. A semantic duplicate is recorded as a **boolean flag** on the existing Message row — no second Message row is created.

**Key observation:** This is a near-duplicate detection, not exact deduplication. It uses word-set Jaccard (not character-level or embedding similarity). No model call is required. The threshold (0.85) is configurable per tenant.

### 6b. Dissatisfied Tone Detection (OBSERVED in `app/agents/signals.py`)

**Algorithm:** Keyword lexicon (26 terms).
- `_DISSATISFIED_LEXICON`: "unacceptable", "very disappointed", "deeply disappointed", "not satisfied", "terrible service", "worst service", "worst experience", "demand a refund", "requesting a refund", "escalate this", "completely unacceptable", "i am appalled", "i am furious", "absolutely disgusted", "this is ridiculous", "i want to complain", "formal complaint", "not good enough", "extremely unhappy", "extremely disappointed", "very unhappy", "will never use", "will never return", "will not recommend", "do not recommend".
- `has_dissatisfied_tone(text)`: case-insensitive substring match against the lexicon.
- When flagged: message routes to Leila (supervise) and auto-send is blocked.
- No model call is required.

### 6c. Idempotency and Duplicate Detection (OBSERVED in `app/ingest/ingest.py`)

**Two levels of duplicate detection:**
1. **Idempotency (row 3):** If a row with `(tenant_id, provider_message_id)` already exists, the existing row is returned with result `"duplicate"`. No new row is created.
2. **Duplicate detection (row 9):** If `message_id_header` + `subject_normalized` match an existing row for this tenant, the new row is stored but its state is set to `"duplicate"`.

---

## 7. Permissions and Human Approval (Phase 5 Lens)

### 7a. Send Decision Policy (OBSERVED in `app/policy/send_decision.py`)

`should_send(policy_config)` returns one of three values:

| Return value | Condition | Effect |
|--------------|-----------|--------|
| `"draft"` | Default (no policy, or auto_reply not enabled) | Store draft for human review; do not send |
| `"shadow"` | `shadow_mode: true` | Store draft, do not send (operator validates) |
| `"allow"` | `auto_reply_enabled: true` (and not shadow) | Send the draft |

**Key observation:** Auto-reply is OFF by default. The tenant must explicitly enable it. `is_send_allowed()` returns True only when `should_send() == "allow"`.

### 7b. Human Approval Queue (OBSERVED in `app/policy/approval.py`)

**ApprovalQueueEntry ORM:**
- `state`: `"pending"` | `"approved"` | `"rejected"`.
- `resolved_by`: operator email (accountability).
- `reason`: why the entry is in the queue (default: `"pending_human_review"`).

**Functions:**
- `enqueue(db, tenant_id, subject, body, ...)`: create pending entry.
- `resolve(db, entry_id, tenant_id, decision, resolved_by)`: set state to `"approved"` or `"rejected"`. Validates `decision in ("approved", "rejected")`. Returns `None` if entry not found or wrong tenant.
- `pending_for_tenant(db, tenant_id)`: returns all pending entries, newest first.

**Tenant scoping:** All queries filter by `tenant_id`. Cross-tenant approval resolution is impossible (returns `None`).

### 7c. Deterministic Decision Hash (OBSERVED in `app/agents/decision_hash.py`)

`compute_decision_hash(subject_normalized, rule_hit, action, rule_pack)`:
- Deterministic SHA-256 hex digest.
- Inputs are sorted (`json.dumps(payload, sort_keys=True)`).
- Only stable rule pack fields are included: `domain_rules`, `subject_rules`, `department_rules`.
- Same inputs → same hash, with or without a live model.
- Lets operators verify that the same input always produces the same decision.

---

## 8. Audit Hash Chain (OBSERVED in `app/policy/audit.py`)

### 8a. Append-Only Audit Log

**AuditLogEntry ORM:**
- `id`, `tenant_id`, `event`, `actor`, `message_id`, `detail`, `created_at`, `prev_hash`.
- No UPDATE or DELETE operations in code — only INSERT via `log_event()`.
- Event categories: `message_ingested`, `triage_decided`, `draft_created`, `draft_approved`, `draft_rejected`, `message_sent`, `message_held`, `message_failed`, `shadow_draft_stored`, `approval_requested`.

### 8b. SHA-256 Hash Chain

**Chain construction:**
1. `log_event()` queries the most recent row for this tenant with `prev_hash IS NOT NULL`.
2. If no previous row → `prev_hash = ZERO_HASH` ("0" × 64).
3. If previous row exists → `prev_hash = SHA-256(canonical(previous_row))`.
4. Canonical: `f"{id}|{tenant_id}|{event}|{actor}|{detail}|{iso_timestamp}"`.

**Chain verification:**
- `verify_chain(db, tenant_id)`: returns list of entry ids where `prev_hash` does not match the expected value.
- Rows with `prev_hash IS NULL` are pre-chain (written before Phase 5) and skipped.
- Empty return → chain intact.

---

## 9. Self-Hosting and Air-Gap (Phase 5 Lens)

### 9a. Docker Compose Air-Gap Package (OBSERVED in `docker-compose.yml`)

```yaml
services:
  db:
    image: postgres:16-alpine
    # ... healthcheck, volumes
  app:
    build: .
    environment:
      DATABASE_URL: postgresql://buro:***@db:5432/buro
      LOCAL_MODEL: "1"           # FakeModel, no cloud model
      # OPENAI_API_KEY: absent     # no cloud model is called
      # GATEWAY_SECRET: absent     # fake payment adapter
      # IMAP credentials: absent   # FakeProvider is used
```

**Key observation:** The Docker Compose file explicitly sets `LOCAL_MODEL=1` and intentionally omits `OPENAI_API_KEY`, `GATEWAY_SECRET`, and IMAP credentials. The app runs fully air-gapped with FakeModel and FakeProvider. No cloud model, no payment gateway, no IMAP connection.

### 9b. Local Model Route (OBSERVED in `app/agents/local_model.py`)

- `is_local_model_enabled(policy_config)`: returns True when tenant has `local_model: true`.
- When set, the pipeline uses `FakeLocalModel` (deterministic stub) instead of the cloud model.
- No real local-inference vendor is shipped; no new vendor key is introduced.
- A production operator replaces `FakeLocalModel` with their own implementation.

### 9c. Payment Boundary (OBSERVED in `app/payment/`)

- `factory.py`: returns `FakePaymentAdapter` when `GATEWAY_SECRET` is absent; `LivePaymentAdapter` when set.
- `fake_adapter.py`: deterministic stub, no card data stored.
- `live_adapter.py`: real gateway adapter (2.7KB).
- `port.py`: payment port interface.
- **No card data is stored.** The fake adapter is used by default.

---

## 10. Test Evidence (Phase 5 Lens)

### 10a. Test Count (OBSERVED — static source count)

| Metric | Value | Method |
|--------|-------|--------|
| Test files | 28 | `ls tests/test_*.py \| wc -l` |
| Test functions | 856 | `grep -c "def test_\|async def test_"` across all test files |
| Author-reported count | 856 | `docs/TEST_HOUSE.md` row 25, `docs/releases/final-phase-9.md` |
| CI runs | 0 | No CI workflow present |
| Independent execution | NOT TESTED | Source repos are read-only per DIRECTIVE.txt |

**Test count matches:** 856 by static source count matches the author's reported 856 in `docs/TEST_HOUSE.md` and `final-phase-9.md`.

### 10b. Test File Distribution

| Test file | Test functions |
|-----------|---------------|
| test_agents.py | 52 |
| test_air_gap.py | 14 |
| test_audit_hash_chain.py | 18 |
| test_buyer_features.py | 50 |
| test_commercial.py | 50 |
| test_contracts.py | 19 |
| test_desk.py | 46 |
| test_desk_signals.py | 45 |
| test_differentiation.py | 96 |
| test_gateway.py | 15 |
| test_ingest.py | 26 |
| test_intake.py | 9 |
| test_local_model.py | 15 |
| test_oauth_mailbox.py | 17 |
| test_operator_surface.py | 21 |
| test_pipeline.py | 19 |
| test_plan_rules.py | 35 |
| test_policy.py | 62 |
| test_privacy.py | 27 |
| test_release.py | 22 |
| test_security.py | 34 |
| test_sender_auth.py | 34 |
| test_subscription.py | 20 |
| test_tenancy.py | 17 |
| test_thread_context.py | 22 |
| test_token_policy.py | 14 |
| test_webhook_delivery.py | 17 |
| test_workers.py | 40 |
| **Total** | **856** |

### 10c. Test House Record (OBSERVED in `docs/TEST_HOUSE.md`)

25 test runs recorded, all by Amin Azimi, all `pytest tests/ -q`. Test count grew from 404 (initial) to 856 (final). No failed runs recorded. External test runs section is empty — `docs/EXTERNAL_TEST.sh` is "written, not run."

---

## 11. Engineering Decisions (ADR-Style)

### ADR-11-001: Rules Before Model

| Field | Value |
|-------|-------|
| ADR_ID | ADR-11-001 |
| Source SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| Problem | Classify messages cost-efficiently while maintaining safety |
| Constraints | Model calls cost tokens and risk leaking PII; rules are deterministic and free |
| Selected approach | Tenant rule engine (`rules.py`): domain → subject → department, first match wins. Rule hit → no model call, zero tokens. |
| Alternatives | Always call model; model-only classification; hybrid scoring |
| Trade-offs | Rules are fast, free, and deterministic but may miss edge cases the model would catch. First-match-wins means rule order matters. |
| Failure implications | Misconfigured rule packs could misclassify; no rule → model fallback (safety net) |
| Evidence | `app/agents/rules.py` (144 lines); `app/pipeline.py` rule-hit check; README "Five things a buyer checks first" item 1 |
| Status | IMPLEMENTED — tested in `tests/test_agents.py` (52 tests) |

### ADR-11-002: Redaction Before Model Call

| Field | Value |
|-------|-------|
| ADR_ID | ADR-11-002 |
| Source SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| Problem | Prevent PII leakage to third-party model API |
| Constraints | Model prompts must not contain email, phone, card, SSN, or IBAN data |
| Selected approach | 5-pattern regex redaction before model call; order matters (specific first) |
| Alternatives | No redaction; post-hoc audit; model-side PII detection |
| Trade-offs | Regex-based redaction may miss non-standard PII formats; attachment bytes are excluded entirely (stronger than redaction) |
| Failure implications | Non-standard PII formats could pass through to the model; known gap documented in THREAT_MODEL.md |
| Evidence | `app/agents/redact.py` (51 lines); `app/pipeline.py` redaction call; THREAT_MODEL.md "Information Disclosure" section |
| Status | IMPLEMENTED — tested in `tests/test_agents.py` `TestRedaction` |

### ADR-11-003: Shadow Mode as Default

| Field | Value |
|-------|-------|
| ADR_ID | ADR-11-003 |
| Source SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| Problem | Prevent unauthorized email sending while operators validate the system |
| Constraints | Auto-reply must be OFF by default; operator must explicitly enable it |
| Selected approach | `should_send()` returns `"draft"` by default; `"shadow"` when shadow_mode is true; `"allow"` only when auto_reply_enabled is true |
| Alternatives | Auto-send by default; no shadow mode; manual approval only |
| Trade-offs | Conservative default prevents accidental sends but requires operator action to enable auto-reply. Shadow mode adds a validation layer. |
| Failure implications | Misconfigured `auto_reply_enabled: true` could auto-send; mitigated by tenant-level config and audit logging |
| Evidence | `app/policy/send_decision.py` (50 lines); `app/policy/shadow.py` (109 lines); `docs/releases/final-phase-9.md` |
| Status | IMPLEMENTED — tested in `tests/test_policy.py` `TestSendDecision` |

### ADR-11-004: Append-Only Audit with Hash Chain

| Field | Value |
|-------|-------|
| ADR_ID | ADR-11-004 |
| Source SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| Problem | Provide non-repudiable audit trail for all policy decisions |
| Constraints | Audit log must be append-only; tampering must be detectable; no UPDATE/DELETE |
| Selected approach | `AuditLogEntry` ORM with `prev_hash` column; `log_event()` is sole write path; `verify_chain()` detects tampering |
| Alternatives | Append-only without hash chain; external audit log; database triggers for immutability |
| Trade-offs | Hash chain detects tampering but not deletion (rows can be physically deleted). Pre-Phase-5 rows have `prev_hash IS NULL` and are skipped. |
| Failure implications | Physical deletion of rows would break chain but not be detected by `verify_chain()` (it only checks existing rows) |
| Evidence | `app/policy/audit.py` (175 lines); `tests/test_audit_hash_chain.py` (18 tests); migration `0013_audit_hash_chain.py` |
| Status | IMPLEMENTED — tested in `tests/test_audit_hash_chain.py` |

### ADR-11-005: Air-Gap Docker Compose

| Field | Value |
|-------|-------|
| ADR_ID | ADR-11-005 |
| Source SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| Problem | Enable fully offline operation for air-gapped environments |
| Constraints | No cloud model, no payment gateway, no IMAP connection required |
| Selected approach | `docker-compose.yml` with `LOCAL_MODEL=1`, no `OPENAI_API_KEY`, no `GATEWAY_SECRET`, no IMAP credentials. FakeModel + FakeProvider + FakePaymentAdapter. |
| Alternatives | Cloud-dependent deployment; optional air-gap; separate air-gap build |
| Trade-offs | Air-gap uses deterministic fakes — not real AI, not real email, not real payment. Intentional for validation/testing. |
| Failure implications | Operator must provide their own model implementation for production use; `FakeLocalModel` is a stub |
| Evidence | `docker-compose.yml` (33 lines); `app/agents/local_model.py`; `app/payment/factory.py`; `tests/test_air_gap.py` (14 tests) |
| Status | IMPLEMENTED — tested in `tests/test_air_gap.py` |

### ADR-11-006: Self-Hosted, Not SaaS

| Field | Value |
|-------|-------|
| ADR_ID | ADR-11-006 |
| Source SHA | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| Problem | Provide office mail desk without operating a hosted service |
| Constraints | Database region must be operator-controlled; no shared infrastructure; GDPR compliance is operator's responsibility |
| Selected approach | Self-hosted via Docker Compose; operator sets `DATABASE_URL`; no hosted service, no SaaS offering, no multi-tenant cloud |
| Alternatives | Hosted SaaS; managed cloud deployment; multi-tenant shared database |
| Trade-offs | Self-hosting puts operational burden on the operator (deployment, backups, updates). Avoids data residency and multi-tenancy risks. |
| Failure implications | Operator must handle deployment, scaling, backups, and GDPR compliance independently |
| Evidence | `docker-compose.yml` comment: "Operator-hosted. No cloud model client is called."; `docs/PRIVACY_DATA_MAP.md`: "Buro Assistant does not operate a hosted service"; `docs/releases/final-phase-9.md`: "No hosted service." |
| Status | IMPLEMENTED — explicit design decision, documented in README, PRIVACY_DATA_MAP, and final-phase-9.md |

---

## 12. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-11-001 | MEDIUM | No CI workflow — 0 `.github/workflows/` files. 856 tests exist but are not CI-enforced. Tests are run manually by the author (`pytest tests/ -q`) and recorded in `docs/TEST_HOUSE.md`. No automated CI means tests could regress without detection. |
| GAP-11-002 | MEDIUM | No published releases or tags — 0 tags, 0 releases. Version 1.0.0 is declared in README badge and `docs/releases/1.0.0.md` but not anchored to a Git tag or GitHub release. |
| GAP-11-003 | LOW | Proprietary license — no source code copied into portfolio. All evidence uses SHA-pinned permalinks and attributed descriptions. Redistribution requires a separate written license from Amin Azimi. |
| GAP-11-004 | MEDIUM | External test script not run — `docs/EXTERNAL_TEST.sh` is written but not executed by a second account. The claim "a second account has installed and verified this" is explicitly not made. |
| GAP-11-005 | LOW | Redaction is regex-based — may miss non-standard PII formats (e.g., unusual phone formats, non-IBAN bank accounts). Known gap documented in THREAT_MODEL.md. |
| GAP-11-006 | LOW | Hash chain detects tampering but not deletion — `verify_chain()` only checks existing rows; physical row deletion would not be detected. Application-layer only; no database trigger enforcement. |
| GAP-11-007 | LOW | Semantic duplicate uses Jaccard on word sets — does not detect semantic duplicates with paraphrased content. Threshold (0.85) is configurable but the algorithm is simple. |
| GAP-11-008 | MEDIUM | OpenAI model is a third-party service — even after redaction, subject and body clips are transmitted to OpenAI. Data-residency gap for EU operators. Mitigated by local model route (Phase 3) but not enforced by default. |
| GAP-11-009 | LOW | `app/main.py` is 49KB (monolithic) — 49,519 bytes. Contains the FastAPI application, all ORM models (Base), auth, API routes, and health checks. Maintainability concern; not a correctness issue. |
| GAP-11-010 | LOW | Test count matches (856) but tests are NOT independently executed. Author's `TEST_HOUSE.md` records 25 runs, all passing, all by Amin Azimi. No second-account verification. |

---

## 13. Claim Comparison

| Claim ID | Claim | Phase 1 Class | Phase 5 Assessment |
|----------|-------|---------------|---------------------|
| CLM-11-001 | "Büro Assistant is at version 1.0.0, Phase 9 closed, self-hosted, not SaaS, payment not live" | REPORTED / NOT_TESTED | **OBSERVED** — README badge confirms `version-1.0.0`, `phase-9 closed`, `self-hosted`, `send-off by default`. `docs/releases/final-phase-9.md` confirms all 9 phases closed. `docker-compose.yml` confirms self-hosted. Payment: `app/payment/factory.py` returns fake adapter when `GATEWAY_SECRET` absent. All claims verified. |
| CLM-11-002 | "Büro Assistant has a decision hash and send_decision policy" | OBSERVED / NOT_TESTED | **OBSERVED** — `app/agents/decision_hash.py` (47 lines): `compute_decision_hash()` with SHA-256, sorted keys. `app/policy/send_decision.py` (50 lines): `should_send()` → `"allow"` / `"draft"` / `"shadow"`. Both ENFORCED in code and tested (856 tests). |

---

## 14. Reproducibility Recipe (R1 → R3 path)

This is a documented path to reach R3 independently. It has NOT been executed (not authorized per DIRECTIVE.txt F03/G/J — source repositories are read-only).

```bash
git clone https://github.com/aminbita162-glitch/buro-assistant.git
cd buro-assistant
git checkout 893db187185dbed7b4dafd461241eb78acb07f8e  # HEAD SHA
# Requires: Python 3.12+, PostgreSQL
pip install -r requirements.txt
# Run tests (856 tests, author-reported)
python3 -m pytest tests/ -q
# Expected: 856 passed, 0 failed (author's final-phase-9.md record)

# Air-gap mode (no cloud, no gateway, no IMAP)
docker compose up
# App at http://localhost:8000
# PostgreSQL at db:5432

# With real OpenAI (for model routes)
export OPENAI_API_KEY=sk-...
export DATABASE_URL=postgresql://user:pass@localhost:5432/buro
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Preconditions: Python 3.12+; `DATABASE_URL` env var for PostgreSQL; `OPENAI_API_KEY` for cloud model routes (otherwise FakeModel/local model); `GATEWAY_SECRET` for live payment (otherwise fake adapter).

**Test reproduction note:** 856 test functions OBSERVED by static source count (grep). Author's `TEST_HOUSE.md` records 25 runs, all `pytest tests/ -q`, all passing. The count grew from 404 (initial) to 856 (Phase 9). Tests are NOT independently executed. No CI workflow exists — tests are run manually.

---

## 15. Evidence Matrix

| EVD ID | Claim | Source path | Observation class | Result | Source URL | Source SHA | Notes |
|--------|-------|-------------|-------------------|--------|------------|------------|-------|
| EVD-11-001 | Proprietary license — all rights reserved | `LICENSE` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/LICENSE | `893db187185d` | "Copyright (c) 2026 Amin Azimi... All rights reserved." No redistribution without written license. No source copied. |
| EVD-11-002 | No CI workflow — 0 .github/workflows | repo tree | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/tree/893db187185dbed7b4dafd461241eb78acb07f8e | `893db187185d` | 175 files, 0 in .github/workflows/. No CI to trigger. |
| EVD-11-003 | 856 test functions across 28 test files | `tests/` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/tree/893db187185dbed7b4dafd461241eb78acb07f8e/tests | `893db187185d` | Static source count: `grep -c "def test_\|async def test_"` = 856. Matches author's TEST_HOUSE.md row 25. |
| EVD-11-004 | 0 release tags, 0 published releases | GitHub API | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant | `893db187185d` | Tags: 0. Releases: 0. Version 1.0.0 in README badge + docs/releases/1.0.0.md only. |
| EVD-11-005 | Rules before model — rule hit produces zero tokens | `app/agents/rules.py` + `app/pipeline.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/agents/rules.py | `893db187185d` | `evaluate_rules()` returns `RuleHit` or `None`. Pipeline skips model call when `rule_hit` is not None. `CostRecord` is zero on rule-hit path. |
| EVD-11-006 | Redaction before model call (5 PII patterns) | `app/agents/redact.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/agents/redact.py | `893db187185d` | IBAN, CARD, SSN, PHONE, EMAIL — order-specific. `redact_message()` redacts subject + body. |
| EVD-11-007 | Shadow mode — store draft, do not send | `app/policy/send_decision.py` + `app/policy/shadow.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/policy/send_decision.py | `893db187185d` | `should_send()` returns `"draft"` by default; `"shadow"` when shadow_mode; `"allow"` only when auto_reply_enabled. |
| EVD-11-008 | Append-only audit with SHA-256 hash chain | `app/policy/audit.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/policy/audit.py | `893db187185d` | `log_event()` sole write path. `prev_hash` = SHA-256 of previous row's canonical representation. `verify_chain()` detects tampering. No UPDATE/DELETE. |
| EVD-11-009 | Human approval queue with tenant scoping | `app/policy/approval.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/policy/approval.py | `893db187185d` | `ApprovalQueueEntry` ORM; `resolve()` filters by `tenant_id`; `resolved_by` records operator email. |
| EVD-11-010 | Deterministic decision hash (SHA-256) | `app/agents/decision_hash.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/agents/decision_hash.py | `893db187185d` | `compute_decision_hash()` with sorted JSON keys. Same inputs → same hash. |
| EVD-11-011 | Sender auth (SPF/DKIM/DMARC) — pass/fail/not_run | `app/ingest/sender_auth.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/ingest/sender_auth.py | `893db187185d` | `SenderAuthResult` dataclass; `any_fail` property; `SENDER_AUTH_ENABLED=1` to activate; fail blocks send, does not delete. |
| EVD-11-012 | Attachment quarantine + text extraction (never to model) | `app/ingest/attachment_text.py` + `app/ingest/ingest.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/ingest/attachment_text.py | `893db187185d` | Only text/plain + text/csv decoded. Attachment bytes never sent to model. Disallowed types → quarantine state. |
| EVD-11-013 | Semantic duplicate detection (Jaccard ≥ 0.85) | `app/agents/signals.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/agents/signals.py | `893db187185d` | `is_semantic_duplicate()` — Jaccard on word sets, threshold 0.85. Boolean flag on Message, no second row. No model call. |
| EVD-11-014 | Dissatisfied tone detection (keyword lexicon) | `app/agents/signals.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/agents/signals.py | `893db187185d` | 26-term lexicon; case-insensitive substring match. Routes to Leila, blocks auto-send. No model call. |
| EVD-11-015 | Air-gap Docker Compose (LOCAL_MODEL=1, no secrets) | `docker-compose.yml` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/docker-compose.yml | `893db187185d` | `LOCAL_MODEL=1`; OPENAI_API_KEY/GATEWAY_SECRET/IMAP intentionally absent. FakeModel + FakeProvider + FakePaymentAdapter. |
| EVD-11-016 | Self-hosted, not SaaS — no hosted service | `README.md` + `docs/PRIVACY_DATA_MAP.md` + `docs/releases/final-phase-9.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/README.md | `893db187185d` | README: "self-hosted operator-controlled". PRIVACY_DATA_MAP: "Buro Assistant does not operate a hosted service". final-phase-9: "No hosted service." |
| EVD-11-017 | Payment not live — fake adapter when GATEWAY_SECRET absent | `app/payment/factory.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/payment/factory.py | `893db187185d` | `factory.py` returns `FakePaymentAdapter` when env absent; `LivePaymentAdapter` when set. No card data stored. |
| EVD-11-018 | 175 total tracked files | repo tree | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/tree/893db187185dbed7b4dafd461241eb78acb07f8e | `893db187185d` | 175 blobs, 19 trees. 62 .py files, 28 test files, 1 Dockerfile, 1 docker-compose.yml, 14 migrations, 25 release docs. |
| EVD-11-019 | Phase 9 closed — all 9 build phases complete | `docs/releases/final-phase-9.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/docs/releases/final-phase-9.md | `893db187185d` | 9 phases: sender auth → bounded thread → local model → OAuth → audit chain → desk signals → air-gap → visual README → record/freeze. |
| EVD-11-020 | 50-row capability checklist all verified | `docs/CAPABILITY_MATRIX.md` + `docs/releases/1.0.0.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/docs/CAPABILITY_MATRIX.md | `893db187185d` | 50 rows, each with "Designed for" and "Verified" columns. All 50 marked ✓ in both columns. |
| EVD-11-021 | STRIDE threat model with mitigations | `docs/THREAT_MODEL.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/docs/THREAT_MODEL.md | `893db187185d` | 6 STRIDE categories with specific threats + mitigations. Known gap: OpenAI third-party model data transmission documented. |
| EVD-11-022 | GDPR privacy data map (Art. 30) | `docs/PRIVACY_DATA_MAP.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/docs/PRIVACY_DATA_MAP.md | `893db187185d` | Per-table personal data inventory with purpose, legal basis, retention. Region is operator-controlled. Body text not stored. |
| EVD-11-023 | Local model route (FakeLocalModel stub) | `app/agents/local_model.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/agents/local_model.py | `893db187185d` | `is_local_model_enabled()` reads tenant flag; `FakeLocalModel` deterministic stub; no real vendor shipped. |
| EVD-11-024 | Idempotency key (tenant_id + provider_message_id) | `app/ingest/ingest.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/ingest/ingest.py | `893db187185d` | `ingest_message()` checks `(tenant_id, provider_message_id)` — returns existing row on duplicate. `RESULT_DUPLICATE`. |
| EVD-11-025 | External test script written, not run | `docs/EXTERNAL_TEST.sh` + `docs/releases/final-phase-9.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/docs/EXTERNAL_TEST.sh | `893db187185d` | 9.4KB script. final-phase-9.md: "The external docs/EXTERNAL_TEST.sh script awaits a second-account run." Not run. |
| EVD-11-026 | Tenant-scoped queries on all business tables | `app/policy/approval.py` + `app/policy/audit.py` + `app/ingest/ingest.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/policy/approval.py | `893db187185d` | All queries filter by `tenant_id`. Cross-tenant reads impossible by query design. 17 tests in `test_tenancy.py`. |
| EVD-11-027 | Argon2id password hashing | `requirements.txt` + `app/main.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/requirements.txt | `893db187185d` | `argon2-cffi==25.1.0`. Not bcrypt — Argon2id is the modern standard. Tested in `test_security.py` (34 tests). |
| EVD-11-028 | 14 Alembic migrations (forward-only) | `migrations/versions/` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/tree/893db187185dbed7b4dafd461241eb78acb07f8e/migrations/versions | `893db187185d` | 0001_baseline through 0014_desk_signals. Forward-only. Includes 0013_audit_hash_chain. |
| EVD-11-029 | Webhook delivery with HMAC-SHA256 signing | `app/workers/webhook_delivery.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/workers/webhook_delivery.py | `893db187185d` | 9.5KB. Signed payload with `X-Buro-Signature`. 17 tests in `test_webhook_delivery.py`. |
| EVD-11-030 | Pipeline with plan capability check (Commercial Phase 4) | `app/pipeline.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/buro-assistant/blob/893db187185dbed7b4dafd461241eb78acb07f8e/app/pipeline.py | `893db187185d` | `check_plan_capability()` before any agent or model call. Plan refusal → `outcome="plan_refused"`, zero tokens. |

---

## 16. 30-Filter Gate Record

**PHASE 5 | PRJ-11 Büro Assistant | Changeset: `cases/11-buro-assistant/engineering-evidence.md` (new file)**

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 5 for PRJ-11 |
| F02 Scope | PASS | Only portfolio path `cases/11-buro-assistant/engineering-evidence.md` modified |
| F03 Source immutability | PASS | No source repository changes; read-only inspection via local clone at pinned SHA (after git fetch) |
| F04 Baseline | PASS | Branch `main`, HEAD `12fe7266062e4d904d6cbac3c92e9eb042469a6b`, clean working tree |
| F05 Conflict | PASS | No conflicting directives; DIRECTIVE.txt is sole authority |
| F06 Intent | PASS | Change advances evidence-first portfolio objective |
| F07 Necessity | PASS | No simpler non-writing alternative; evidence file required for Phase 5 |
| F08 Duplication | PASS | No existing `engineering-evidence.md` for PRJ-11; no duplicate content |
| F09 Provenance | PASS | Source URL, full SHA, file paths, timestamps captured |
| F10 Authenticity | PASS | No source statement treated as independently verified |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/INFERRED/NOT TESTED/CONDITIONAL used precisely |
| F12 Reproducibility | PASS | A reader can follow claim→source→evidence chain via permalinks |
| F13 Technical correctness | PASS | Architecture and code descriptions match actual source |
| F14 Domain safety | PASS | Email/attachment injection, privacy, human approval all represented accurately |
| F15 Trust boundaries | PASS | Source files treated as untrusted data; no prompt injections followed |
| F16 Security | PASS | No secrets introduced; hardcoded JWT secret is in PRJ-08 not this repo; proprietary license respected |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific information; PII patterns described generically |
| F18 Intellectual property | PASS | Proprietary license — no source code copied; all evidence uses permalinks and attributed descriptions only |
| F19 Attribution | PASS | Source author (Amin Azimi) accurately attributed |
| F20 Test integrity | PASS | No unexecuted test described as passed; 856 test functions by static source count; NOT independently executed — explicitly stated |
| F21 Failure modes | PASS | No CI, external test not run, OpenAI third-party gap, regex redaction limitations, hash chain deletion gap all represented |
| F22 Version drift | PASS | Release, branch, CI, deployment, and docs not conflated; "Phase 9 closed" pinned |
| F23 Interface stability | PASS | Existing case README, links, and case ordering preserved |
| F24 Accessibility | PASS | Headings, tables, and text are navigable; no images requiring alt text |
| F25 Maintainability | PASS | Concise structure; no second source tree; links to source |
| F26 Reviewability | PASS | Diff is one new file; coherent and attributable to Phase 5 PRJ-11 outcome |
| F27 Validation | PASS | Available safe local checks: markdown structure verified; no external validators run |
| F28 Git hygiene | PASS | Commit authorized for Phase 5; push authorized for Phase 5; single meaningful commit |
| F29 Disclosure | PASS | Public-safe content; no sensitive material; proprietary license noted; no source copied |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision: PROCEED** — all filters PASS; no blocking evidence.

---

## 17. Phase Completion Report

```
PHASE: 5 — Domain-Breadth Cases (PRJ-11 Büro Assistant)
AUTHORIZED SCOPE: Portfolio write — cases/11-buro-assistant/engineering-evidence.md (new file)
BASELINE HEAD: 12fe7266062e4d904d6cbac3c92e9eb042469a6b (portfolio main, clean working tree)
OBSERVED FACTS:
  - Source repo inspected at pinned SHA 893db187185dbed7b4dafd461241eb78acb07f8e via local clone
  - 175 tracked files; 62 .py source files; 28 test files with 856 test functions (static source count)
  - Proprietary license — no source code copied into portfolio; all evidence uses permalinks
  - No CI workflow (0 .github/workflows); 0 CI runs; 0 release tags; 0 published releases
  - Phase 9 closed — all 9 build phases complete; commercialization explicitly not done
  - Rules before model: tenant rule engine (domain→subject→department, first match wins, zero tokens on hit)
  - Redaction before model: 5 PII patterns (IBAN, CARD, SSN, PHONE, EMAIL), order-specific
  - Shadow mode: auto-reply OFF by default; "draft" → "shadow" → "allow" progression
  - Append-only audit with SHA-256 hash chain; verify_chain() detects tampering
  - Human approval queue with tenant scoping; resolved_by records operator email
  - Deterministic decision hash (SHA-256, sorted keys)
  - Sender auth: SPF/DKIM/DMARC (pass/fail/not_run); fail blocks send, does not delete
  - Attachment quarantine + text extraction (never to model); only text/plain + text/csv decoded
  - Semantic duplicate (Jaccard ≥ 0.85) + dissatisfied tone (26-term lexicon)
  - Air-gap Docker Compose (LOCAL_MODEL=1, no OPENAI_API_KEY, no GATEWAY_SECRET, no IMAP)
  - Self-hosted, not SaaS; GDPR privacy data map (Art. 30); STRIDE threat model with mitigations
  - 50-row capability checklist all marked Designed for + Verified
  - External test script written, not run; second-account verification not done
  - Payment not live — fake adapter when GATEWAY_SECRET absent; no card data stored
FILES CHANGED: cases/11-buro-assistant/engineering-evidence.md (new file)
EVIDENCE ADDED: EVD-11-001 through EVD-11-030; ADR-11-001 through ADR-11-006; GAP-11-001 through GAP-11-010
CHECKS ACTUALLY RUN:
  - git status --short (clean working tree confirmed)
  - git rev-parse HEAD (portfolio HEAD confirmed)
  - git fetch origin main + git checkout (pinned SHA available in local clone)
  - Source file inspection (62 .py files, 28 test files, all key docs read)
  - Test function count: grep -c "def test_\|async def test_" = 856 (matches author's report)
  - GitHub API (CI runs: 0, tags: 0, releases: 0, license: proprietary)
  - Markdown structure verification (visual)
NOT TESTED / BLOCKED:
  - Full test suite NOT independently executed (source repos are read-only per DIRECTIVE.txt)
  - No CI workflow — tests are not CI-enforced; 856 tests run manually by author only
  - No independent reproduction (R3) — source repos are immutable
  - External test script not run by second account
RISKS / OPEN DECISIONS:
  - GAP-11-001 (MEDIUM): No CI workflow — 856 tests not CI-enforced
  - GAP-11-004 (MEDIUM): External test script not run by second account
  - GAP-11-008 (MEDIUM): OpenAI third-party model — data residency gap for EU operators
  - GAP-11-002 (MEDIUM): No published releases or tags
COMMIT SHA: (pending — will commit after PRJ-09 is complete)
PUSH STATUS: (pending — will push after all three Phase 5 projects are committed)
NEXT: PRJ-09 (Personal AI Assistant) — proceeding per owner authorization.
```
