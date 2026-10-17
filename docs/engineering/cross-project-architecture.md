# Cross-Project Architecture and Engineering Synthesis

**Phase:** 8 — Cross-Project Architecture and Engineering Synthesis (PORTFOLIO WRITE)
**Created:** 2026-10-18
**Phase 8 portfolio HEAD (pre-execution):** `2ab932c093c1aaf8bc5947c80355b06d8402d4ea`
**Reviewer:** Second-agent review (Hermes) — SECOND_AGENT_REVIEW; not an independent third-party review
**Independence:** All findings are this agent's read-only source inspections across 13 projects. No source code was executed. No source repositories were modified.
**Evidence maturity ceiling:** R1 (source inspected at pinned SHA) for 11 projects; R0 (live URL only) for PRJ-07 and PRJ-13. No project reached R3 (independent clean-room reproduction).

---

## How to Use This Document

This is the Phase 8 cross-project synthesis. It identifies **real differences** and **genuine common patterns** across the 13 projects — not an invented uniform architecture. Every comparative claim links to constituent case evidence (`cases/<NN>/engineering-evidence.md`) or the central evidence index (`docs/evidence/central-index.md`).

**Skeptical-reviewer navigation path:** claim → case evidence file → source permalink (SHA-pinned) → evidence ID (EVD-NN-NNN) → limitation or gap (GAP-NN-NNN). If a claim cannot be traced through this chain, it is labeled INFERRED or NOT VERIFIED.

---

## 1. Multi-Agent Orchestration and Human Authority

Five of 13 projects implement multi-agent coordination with a human authority gate. The architectural approaches differ materially — this is not a shared framework.

| Project | Agent count | Routing mechanism | Human authority gate | Evidence |
|---------|-------------|-------------------|---------------------|----------|
| PRJ-01 AILORA | 1 (Oya, disabled) | `max_requested_tools=0` — tool execution forbidden | Advisory-only boundary; no spacecraft command path | [EVD-01-005](../cases/01-ailora/engineering-evidence.md#16-evidence-matrix) · [ADR-01-001](../cases/01-ailora/engineering-evidence.md#9-adr-01-001--advisory-only-platform-with-no-spacecraft-command-path) |
| PRJ-02 Aegis | 6 specialists | Keyword-based routing (`resolve_agent_name`) | 4-layer: risk classification (L0–L3) → HumanAuthority evaluator → hash-bound SHA-256 approve → tenant binding | [EVD-02-001](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) · [ADR-02-002](../cases/02-aegis-agent/engineering-evidence.md#adr-02-002-hash-bound-approve-as-tamper-proofing) |
| PRJ-04 AlinaMatrix | 4 agents (frozen capabilities) | M03 pipeline (AlinaArchitect/Guard/DocEngine/Optimizer) | Four-eyes approval at API boundary; 8-condition release gate | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-05 Kian | 4 agents + risk kernel | Execution engine + graduated autonomy L0–L5 | `live_trading_authorized=False` invariant; `OperatingModeConfig` simulation-only | [central-index §PRJ-05](../docs/evidence/central-index.md#prj-05-kian-trading-intelligence--phase-4-complete) |
| PRJ-07 NimaOS | 5 agents (REPORTED) | Unknown — no source repo | "ask before_action" security (REPORTED, no source) | [EVD-07-001](../cases/07-nimaos-ai-agent/engineering-evidence.md) — R0, no source |

**Key contrast:** PRJ-02 uses a hash-bound SHA-256 digest to prevent payload mutation between review and execution — a tamper-proofing mechanism not present in any other project. PRJ-04 enforces four-eyes approval at the API boundary. PRJ-05 uses a deterministic risk kernel with 9 sequential safety checks. PRJ-01's agent is intentionally disabled (`max_requested_tools=0`). These are distinct engineering decisions, not variations of one pattern.

**Limitation:** No project has runtime instrumentation proving the approval gate is the sole execution path. PRJ-02's risk evaluation is a heuristic (keyword + length), not a formal policy engine ([GAP-02-004](../cases/02-aegis-agent/engineering-evidence.md#9-known-limitations-and-gaps)). PRJ-07's agent architecture is INFERRED from live URL content only — no source evidence (R0).

---

## 2. Tenant Isolation Patterns

Six projects implement tenant isolation. The approaches span from database-enforced Row Level Security to application-layer query scoping.

| Project | Isolation layer | Enforcement | Evidence |
|---------|-----------------|-------------|----------|
| PRJ-01 AILORA | PostgreSQL RLS + application context injection | `@event.listens_for` sets `app.current_tenant_id` per transaction; ADR-003 `ACCEPTED_PARTIAL` | [EVD-01-006](../cases/01-ailora/engineering-evidence.md#16-evidence-matrix) · [ADR-01-003](../cases/01-ailora/engineering-evidence.md#11-adr-01-003--append-only-audit-chain-with-sha-256-hash-integrity) |
| PRJ-03 IBM | JWT dependency injection | `resolve_org_id()` — non-admin users get their own `org_uuid` regardless of request params | [EVD-03-006](../cases/03-supply-chain-challenge-ibm-2026/engineering-evidence.md#12-evidence-matrix) |
| PRJ-04 AlinaMatrix | PostgreSQL 16 RLS with FORCE + fail-closed | Unset `tenant_id` → no rows; server-derived UUID-validated context | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-08 Lieferkette | Application-layer (`owner_id` filtering) | User-scoped queries; no DB-level RLS | [EVD-08-001](../cases/08-lieferkette-optimierungsplattform/engineering-evidence.md) · [GAP-08-001](../cases/08-lieferkette-optimierungsplattform/engineering-evidence.md) |
| PRJ-10 Restaurant | Application-layer query scoping | `X-Api-Key` + optional `X-Tenant-Token`; per-tenant rate limit (5 req/60s) | [central-index §PRJ-10](../docs/evidence/central-index.md#prj-10-restaurant-ai-platform--phase-4-complete) · [GAP-10-005](../cases/10-restaurant-ai-platform/engineering-evidence.md) |
| PRJ-11 Büro | Tenant-scoped queries | Approval queue with tenant scoping; tenant rule engine (first match wins) | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) |

**Key contrast:** PRJ-04 has the strongest tenant isolation — PostgreSQL RLS with FORCE and fail-closed, meaning an unset tenant context returns zero rows rather than all rows. PRJ-01's RLS is `ACCEPTED_PARTIAL` — foundation present but full multi-tenant defense is ongoing. PRJ-08 and PRJ-10 rely on application-layer filtering only, with no database-level enforcement. PRJ-08 has a hardcoded JWT secret in 4 files ([GAP-08-002](../cases/08-lieferkette-optimierungsplattform/engineering-evidence.md)) — a material security limitation.

**Limitation:** No project has independently verified tenant isolation through runtime penetration testing. PRJ-01's ADR-003 is explicitly partial. PRJ-08 and PRJ-10's application-layer isolation could be bypassed by a query that omits the `owner_id`/tenant filter.

---

## 3. Deterministic vs Live Model Separation

Three projects implement explicit deterministic/fake provider separation with env-gated live providers. This is a shared engineering pattern — but implemented differently in each.

| Project | Default provider | Live provider | Env-gating | Budget enforcement | Evidence |
|---------|-----------------|---------------|------------|-------------------|----------|
| PRJ-02 Aegis | EchoProvider (offline echo) | HttpProvider | `AEGIS_LLM_PROVIDER=http` + `AEGIS_LLM_BASE_URL` + `AEGIS_LLM_API_KEY` | `AEGIS_LLM_BUDGET_EXHAUSTED=1`; quota ledger | [EVD-02-006](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) · [ADR-02-003](../cases/02-aegis-agent/engineering-evidence.md#adr-02-003--echoprovider-as-default-llm-explicit-offline-first) |
| PRJ-04 AlinaMatrix | DeterministicFakeProvider (pure function, SHA-256 seed) | PinnedModelProvider + LiveModelProvider | Environment-gated | Budget-enforced + schema-validated; 39 pinned-model tests | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) · [GAP-04-001 RESOLVED](../cases/04-alina-matrix-enterprise/engineering-evidence.md) |
| PRJ-05 Kian | Simulation mode (deterministic) | LIVE mode | `OperatingModeConfig` singleton defaults to SIMULATION; `live_authorized=False` | `set_mode(LIVE)` without `live_authorized` raises `PermissionError` | [central-index §PRJ-05](../docs/evidence/central-index.md#prj-05-kian-trading-intelligence--phase-4-complete) · [GAP-05-001 RESOLVED](../cases/05-kian-trading-intelligence/engineering-evidence.md) |

**Key contrast:** PRJ-04's DeterministicFakeProvider is a pure function with no network path and a SHA-256 seed from pinned hashes — the most formally rigorous separation. PRJ-02's EchoProvider is a simpler offline echo with optional HTTP adapter gated by three env vars. PRJ-05's separation is at the operating-mode level, not the provider level — `live_trading_authorized=False` is an invariant, not a provider choice.

**Shared pattern:** All three default to deterministic/offline operation and require explicit environmental activation for live behavior. This prevents silent live-mode activation and enables deterministic testing.

**Limitation:** PRJ-04 has 2 skipped live-model tests (no API key — [GAP-04-002](../cases/04-alina-matrix-enterprise/engineering-evidence.md)). PRJ-05 has no published releases. PRJ-02's quota is in-memory (not persisted across restarts).

---

## 4. Audit and Provenance Architecture

Three projects implement cryptographic audit chains. The mechanisms differ in construction and verification.

| Project | Mechanism | Hash algorithm | Tamper detection | Evidence |
|---------|-----------|---------------|-----------------|----------|
| PRJ-01 AILORA | SHA-256 hash chain (length-prefixed canonical payload) | SHA-256 | `AuditIntegrityError` on discontinuity or hash mismatch; append-only DB constraints (migration 0014) | [EVD-01-008](../cases/01-ailora/engineering-evidence.md#16-evidence-matrix) · [ADR-01-003](../cases/01-ailora/engineering-evidence.md#11-adr-01-003--append-only-audit-chain-with-sha-256-hash-integrity) |
| PRJ-04 AlinaMatrix | Merkle provenance ledger (SHA-256, sorted leaves) | SHA-256 | Tamper detection via Merkle root verification; revocation as event (not byte mutation) | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-11 Büro | Append-only audit with SHA-256 hash chain | SHA-256 | Hash chain continuity; does NOT detect deletion | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) · [GAP-11-006](../cases/11-buro-assistant/engineering-evidence.md) |

**Key contrast:** PRJ-04 uses a Merkle tree (sorted leaves, root hash verification) — the most structurally rigorous approach, enabling efficient proof-of-inclusion. PRJ-01 uses a linear hash chain with length-prefixed canonical serialization shared with PostgreSQL (enabling server-side verification). PRJ-11 uses a simpler SHA-256 hash chain that does not detect deletion ([GAP-11-006](../cases/11-buro-assistant/engineering-evidence.md)).

**Shared pattern:** All three use SHA-256 for tamper-evidence. All three store `previous_hash` to form a chain. No project has independently verified the chain's integrity under adversarial conditions (no penetration testing).

**Limitation:** PRJ-01's chain is a software-layer control; WORM/database-level constraints remain on the qualification roadmap ([GAP-01-007](../cases/01-ailora/engineering-evidence.md#17-open-gaps-after-phase-3)). PRJ-04's replay test is NOT TESTED ([GAP-04-004](../cases/04-alina-matrix-enterprise/engineering-evidence.md)). PRJ-11's hash chain does not detect row deletion.

---

## 5. Human Approval Gates — Comparative Depth

| Project | Gate mechanism | Binding | Re-approval on mutation | Evidence |
|---------|---------------|---------|--------------------------|----------|
| PRJ-02 Aegis | 4-layer (risk → authority → hash → tenant) | SHA-256 canonical envelope digest | Yes — digest mismatch rejects | [EVD-02-002](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |
| PRJ-04 AlinaMatrix | Four-eyes at API boundary | Release gate (8 conditions) | Yes — revocation as event | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-05 Kian | `live_authorized=False` invariant | OperatingModeConfig singleton | N/A — LIVE mode cannot be activated | [central-index §PRJ-05](../docs/evidence/central-index.md#prj-05-kian-trading-intelligence--phase-4-complete) |
| PRJ-11 Büro | Approval queue with tenant scoping | Deterministic decision hash | Yes — `decision_hash` binding | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) |

**Key contrast:** PRJ-02's hash-bound approve is the strongest tamper-proofing — the canonical SHA-256 digest of the full action envelope (7 fields) prevents any payload mutation between review and execution. PRJ-11's decision hash is simpler. PRJ-04's four-eyes is enforced at the API boundary. PRJ-05's gate is a mode invariant, not an approval flow.

**Limitation:** PRJ-02's risk evaluation is heuristic (6 keywords + length factor) — not a formal policy engine ([GAP-02-004](../cases/02-aegis-agent/engineering-evidence.md#9-known-limitations-and-gaps)). PRJ-05's emergency stop deactivation is caller responsibility ([GAP-05-008](../cases/05-kian-trading-intelligence/engineering-evidence.md)).

---

## 6. Security and Trust Boundary Patterns

| Pattern | Projects | Implementation | Evidence |
|---------|----------|---------------|----------|
| Tool allowlists | PRJ-02 | `ALLOWED_TOOLS` (3 items) + forbidden-claims rejection (4 items) + secret redaction | [EVD-02-005](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |
| Redaction before model | PRJ-11 | 5 PII patterns (IBAN/CARD/SSN/PHONE/EMAIL) redacted before any model call | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) |
| Attachment quarantine | PRJ-11 | Attachment bytes never sent to model; only text/plain and text/csv decoded | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) |
| Non-medical boundary | PRJ-12 | Disclaimer-based, not code-enforced | [GAP-12-008](../cases/12-patch-auri/engineering-evidence.md) |
| Supply-chain admission | PRJ-01 | Ed25519-signed evidence manifest; "EVIDENCE_VERIFIED_NOT_DEPLOYMENT_AUTHORIZATION" | [EVD-01-012](../cases/01-ailora/engineering-evidence.md#16-evidence-matrix) |
| Untrusted ingest | PRJ-02 | Ingested content never treated as trusted instruction; passes through `complete_safe` cage | [EVD-02-019](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |

**Key contrast:** PRJ-02's `complete_safe` gateway is the most comprehensive single-point security control — combining tool allowlisting, forbidden-claims rejection, secret redaction, quota enforcement, and budget exhaustion in one function. PRJ-11's redaction-before-model is domain-specific (email PII). PRJ-12's non-medical boundary is disclaimer-based, not code-enforced — an honestly disclosed limitation.

**Shared pattern:** PRJ-01, PRJ-02, and PRJ-11 all enforce the principle that ingested content is untrusted data, not trusted instructions. PRJ-01 does this via `OyaSafetyPolicy` (prompt injection markers → REJECTED). PRJ-02 does it via `complete_safe`. PRJ-11 does it via rules-before-model + redaction-before-model.

---

## 7. CI and Testing Maturity — Honest Spectrum

| Maturity level | Projects | CI status | Test count | Independent execution |
|---------------|----------|-----------|------------|----------------------|
| R2 (CI artifact linked) | PRJ-01 | FAIL at HEAD; SUCCESS at qualifying SHA `9571b168` | 1058 REPORTED (91 files OBSERVED) | NOT TESTED |
| R2 partial (CI PASS at HEAD) | PRJ-04, PRJ-05, PRJ-08, PRJ-10 | PASS at HEAD | PRJ-04: 938 REPORTED (56 files); PRJ-05: 2003 REPORTED; PRJ-08: 4 tests (1 file); PRJ-10: 43 tests (11 files) | NOT TESTED |
| R1 (source inspected, no CI) | PRJ-02 | ALL 3 FAIL at HEAD | 725 REPORTED (269 files OBSERVED) | NOT TESTED |
| R1 (source inspected, no CI) | PRJ-03, PRJ-06, PRJ-09, PRJ-11, PRJ-12 | No CI | PRJ-03: 69 REPORTED; PRJ-06: 5 tests; PRJ-09: 0 tests; PRJ-11: 856 (independently counted); PRJ-12: 0 tests | NOT TESTED |
| R0 (no source) | PRJ-07, PRJ-13 | N/A | N/A | N/A |

**Key findings:**
- **5 of 13** projects have CI. Of those, **2 have failing CI at HEAD** (PRJ-01, PRJ-02).
- **No test count has been independently reproduced** (R3). PRJ-11's 856 functions were independently counted by this reviewer (matching the author's claim) but not independently executed.
- PRJ-10 has the most systematic test file structure outside the flagship cases (43 test functions, 11 files) but CI scope is smoke-only (4 of 43 tests CI-enforced — [GAP-10-001](../cases/10-restaurant-ai-platform/engineering-evidence.md)).
- PRJ-09 (Personal AI Assistant) has zero tests, zero CI, zero license — the weakest testing maturity among projects with source.
- PRJ-08 has only 1 test file (4 tests) for 31 source files — minimal coverage.

**Honest assessment:** CI maturity is not uniform across the portfolio. It ranges from R2 (linked CI artifact) to R0 (no source). No project has reached R3 (independent clean-room reproduction). A passing CI run at HEAD is not the same as independently verified test results.

---

## 8. Observability and Structured Logging

| Project | Logging approach | Correlation/tracing | Evidence |
|---------|-----------------|---------------------|----------|
| PRJ-01 AILORA | `structlog` + JSON; OpenTelemetry tracing (OTLP/HTTP) | `correlation_id` per request; workflow SLO metrics | [ADR-012](../cases/01-ailora/engineering-evidence.md#7-observability-architecture-observed-in-source) |
| PRJ-04 AlinaMatrix | Structured logging with `run_id` | `run_id` per pipeline execution | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-10 Restaurant | Structured logging (`run_id`, `step`, `duration_ms`) | `run_id` per pipeline run | [central-index §PRJ-10](../docs/evidence/central-index.md#prj-10-restaurant-ai-platform--phase-4-complete) |
| PRJ-11 Büro | SHA-256 hash chain audit | Deterministic decision hash per action | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) |
| PRJ-02 Aegis | JSONL audit with correlation IDs | `_get_or_create_correlation()` per event | [EVD-02-023](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |

**Key contrast:** PRJ-01 has the most sophisticated observability stack — `structlog` + OpenTelemetry + SLO/error budget contracts (ADR-012 `ACCEPTED_PARTIAL`). PRJ-04 and PRJ-10 use `run_id`-based structured logging. PRJ-11's observability is audit-chain-centric (SHA-256 hash per event). PRJ-02 uses JSONL audit with correlation IDs.

**Shared pattern:** PRJ-01, PRJ-04, and PRJ-10 all use `run_id` or `correlation_id` for request tracing. No project has production telemetry or longitudinal operational measurement (R5).

**Limitation:** PRJ-01's ADR-012 is `ACCEPTED_PARTIAL` — "vendor, region, queue product, monitoring backend remain provider-neutral." No project has production observability. No project has operational telemetry evidence.

---

## 9. Domain Safety Invariants

| Project | Domain | Safety invariant | Enforcement | Evidence |
|---------|--------|-----------------|-------------|----------|
| PRJ-01 AILORA | Space situational awareness | No spacecraft command path; advisory-only | `max_requested_tools=0`; `OYA_RELEASE_AUTHORITY=NOT_APPROVED` | [ADR-01-001](../cases/01-ailora/engineering-evidence.md#9-adr-01-001--advisory-only-platform-with-no-spacecraft-command-path) |
| PRJ-05 Kian | Financial trading | No real trading; no real mining; no guaranteed returns | `live_trading_authorized=False`; `OperatingModeConfig` simulation-only; 9-check risk kernel | [central-index §PRJ-05](../docs/evidence/central-index.md#prj-05-kian-trading-intelligence--phase-4-complete) |
| PRJ-12 Patch-Auri | Health-adjacent | Non-medical informational only; no medical diagnosis | Disclaimer-based, not code-enforced | [GAP-12-008](../cases/12-patch-auri/engineering-evidence.md) |
| PRJ-11 Büro | Email/office | Rules-before-model; redaction-before-model; attachment quarantine | Code-enforced (5 PII patterns, text-only decode) | [EVD-11-001](../cases/11-buro-assistant/engineering-evidence.md) |

**Key contrast:** PRJ-01 and PRJ-05 enforce domain safety invariants in code (`max_requested_tools=0`, `live_authorized=False`). PRJ-12's non-medical boundary is disclaimer-based — an honestly disclosed limitation, not a code-enforced control. PRJ-11's email safety is code-enforced (rules-before-model, redaction, quarantine).

**Shared pattern:** PRJ-01, PRJ-05, and PRJ-11 all enforce the principle that a domain-specific risk must be mitigated before any model or external action. The mechanisms differ: PRJ-01 forbids tools entirely; PRJ-05 forbids live mode; PRJ-11 redacts PII before model calls.

---

## 10. Failure Handling and Resilience Patterns

| Project | Failure mechanism | Retry strategy | Circuit breaking | Evidence |
|---------|------------------|---------------|-----------------|----------|
| PRJ-02 Aegis | `CircuitBreaker` (3 failures → open, 30s → half-open) | `RetryGuard` (max 2 retries) | Yes — `allow_request()` returns False when open | [EVD-02-011](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |
| PRJ-02 Aegis | `ReversibleWorkflowManager` (rollback stack) | Compensating undo in reverse order | N/A — rollback-based | [EVD-02-012](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |
| PRJ-01 AILORA | Durable workflow state machine; DLQ | Recovery with replay | Not observed | [§6](../cases/01-ailora/engineering-evidence.md#6-test-architecture-observed-in-source) |
| PRJ-06 Ahmad & Amin | Polling fallback (3000ms); service worker (network-first GET only) | `shouldPoll` pure function (5 tests) | No | [EVD-06-001](../cases/06-ahmad-amin-technology/engineering-evidence.md) |
| PRJ-10 Restaurant | `FOR UPDATE SKIP LOCKED` async job queue | Worker as separate process | No | [central-index §PRJ-10](../docs/evidence/central-index.md#prj-10-restaurant-ai-platform--phase-4-complete) |

**Key contrast:** PRJ-02 has the most explicit failure-handling architecture — circuit breaker, retry guard, reversible workflow with rollback stack, and quota exhaustion handling. PRJ-01 has durable workflows with DLQ and replay. PRJ-06 has a polling fallback with a tested pure function. PRJ-10 uses Postgres `FOR UPDATE SKIP LOCKED` for async job processing.

**Shared pattern:** PRJ-02 and PRJ-01 both implement durable/reversible workflows. No project has independently verified failure recovery under load.

---

## 11. Data Flow and Persistence Patterns

| Project | Database | Migrations | ORM | Multi-tenancy persistence | Evidence |
|---------|----------|-----------|-----|--------------------------|----------|
| PRJ-01 AILORA | PostgreSQL (asyncpg) | 14 Alembic | SQLAlchemy async | RLS + tenant context injection | [EVD-01-010](../cases/01-ailora/engineering-evidence.md#16-evidence-matrix) |
| PRJ-03 IBM | PostgreSQL | 1 Alembic | SQLAlchemy | JWT org scoping | [EVD-03-006](../cases/03-supply-chain-challenge-ibm-2026/engineering-evidence.md#12-evidence-matrix) |
| PRJ-04 AlinaMatrix | PostgreSQL 16 | 14 forward-only | Prisma/TypeScript | RLS FORCE + fail-closed | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-08 Lieferkette | PostgreSQL | 14 Alembic | SQLAlchemy 2.0+ | User-scoped (`owner_id` filtering) | [central-index §PRJ-08](../docs/evidence/central-index.md#prj-08-lieferkette-optimierungsplattform--phase-5-complete) |
| PRJ-09 Personal AI | PostgreSQL | Runtime schema init | psycopg2 raw SQL | User-scoped queries | [central-index §PRJ-09](../docs/evidence/central-index.md#prj-09-personal-ai-assistant--phase-5-complete) |
| PRJ-10 Restaurant | PostgreSQL | N/A (Flask) | Raw SQL / SQLAlchemy | Application-layer query scoping | [central-index §PRJ-10](../docs/evidence/central-index.md#prj-10-restaurant-ai-platform--phase-4-complete) |
| PRJ-11 Büro | PostgreSQL | 14 Alembic | SQLAlchemy 2.0.54 | Tenant-scoped queries | [central-index §PRJ-11](../docs/evidence/central-index.md#prj-11-buro-assistant--phase-5-complete) |
| PRJ-12 Patch-Auri | SQLite | N/A | SQLAlchemy | No multi-tenancy | [central-index §PRJ-12](../docs/evidence/central-index.md#prj-12-patch-auri--phase-6-complete) |
| PRJ-02 Aegis | SQLite (local) | N/A | SQLAlchemy | Tenant-scoped SQLite | [EVD-02-007](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |

**Key contrast:** PRJ-01 and PRJ-04 both use PostgreSQL with 14 Alembic/Prisma migrations and RLS. PRJ-04's RLS is FORCE + fail-closed (strongest). PRJ-09 uses raw SQL with runtime schema init (`CREATE TABLE IF NOT EXISTS`) — no migration framework. PRJ-12 uses SQLite only (prototype). PRJ-02 uses local SQLite for a local-first product.

**Shared pattern:** 7 of 13 projects use PostgreSQL. 5 of those use Alembic or Prisma migrations. PRJ-01, PRJ-08, and PRJ-11 all have 14 migrations — suggesting a similar migration-first development approach, though implemented independently.

**Limitation:** PRJ-08 has a hardcoded JWT secret in 4 files ([GAP-08-002](../cases/08-lieferkette-optimierungsplattform/engineering-evidence.md)). PRJ-09 stores auth tokens as plaintext in the database ([GAP-09-005](../cases/09-personal-ai-assistant/engineering-evidence.md)). PRJ-12 uses unsalted SHA-256 for password hashing ([GAP-12-002](../cases/12-patch-auri/engineering-evidence.md)).

---

## 12. AI Evaluation and Model Architecture

| Project | AI integration | Model routing | Output validation | Evidence |
|---------|---------------|--------------|------------------|----------|
| PRJ-02 Aegis | EchoProvider (default) / HttpProvider (env-gated) | `TrustAwareModelRouter` (cheap vs expensive by task length) | `complete_safe` gateway (tool allow-list, forbidden claims, schema) | [EVD-02-005](../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix) |
| PRJ-04 AlinaMatrix | DeterministicFakeProvider / PinnedModelProvider / LiveModelProvider | 39 pinned-model tests | Schema validation + quote-lock + 4 soft rules | [central-index §PRJ-04](../docs/evidence/central-index.md#prj-04-alinamatrix-enterprise--phase-4-complete) |
| PRJ-09 Personal AI | OpenAI gpt-4o-mini (task extraction, smart decision, Whisper) | Direct OpenAI calls | None observed | [central-index §PRJ-09](../docs/evidence/central-index.md#prj-09-personal-ai-assistant--phase-5-complete) |
| PRJ-10 Restaurant | GPT insight (7 keys, 3 actions) | `method=dow_heuristic` (never `ml_model`) | AI output schema gate (degraded fallback to `method=rule`) | [central-index §PRJ-10](../docs/evidence/central-index.md#prj-10-restaurant-ai-platform--phase-4-complete) |
| PRJ-11 Büro | OpenAI (task extraction, semantic duplicate, tone analysis) | Rules-before-model (first match wins, zero tokens on hit) | Redaction before model (5 PII patterns) | [central-index §PRJ-11](../docs/evidence/central-index.md#prj-11-buro-assistant--phase-5-complete) |

**Key contrast:** PRJ-02's `TrustAwareModelRouter` routes by keyword heuristics to cheap vs expensive models — a cost-aware routing decision. PRJ-10 explicitly labels forecast method as `dow_heuristic` (never `ml_model`) — an honest non-AI-claim for the core forecasting logic. PRJ-09 has no output validation for AI responses. PRJ-11's rules-before-model means the model is never called if a tenant rule matches — a zero-token optimization.

**Shared pattern:** PRJ-02, PRJ-04, and PRJ-11 all gate model access through a safety/validation layer before the model is called. PRJ-09 does not — it's the least mature AI integration in the portfolio.

---

## 13. Reproducibility and Evidence Maturity — Cross-Project View

| Level | Projects | Evidence basis |
|-------|----------|---------------|
| R2 (CI artifact linked) | PRJ-01 | Qualifying CI run `33025990734` at SHA `9571b168` (SUCCESS) |
| R2 partial (CI PASS at HEAD) | PRJ-04, PRJ-05, PRJ-08, PRJ-10 | CI run SUCCESS observed via GitHub Actions API; not independently reproduced |
| R1 (source inspected at pinned SHA) | PRJ-02, PRJ-03, PRJ-06, PRJ-09, PRJ-11, PRJ-12 | Read-only source inspection; no CI artifact linked (or CI failing/not present) |
| R0 (no source artifact) | PRJ-07, PRJ-13 | Live URL content only; no source repository available |

**No project has reached R3** (independent clean-environment reproduction) or higher. The portfolio's evidence is entirely source-level (R1) or CI-linked (R2 partial). No test suite has been independently executed in a clean environment by this reviewer.

**Honest disclosure:** A second AI agent repeating an assertion is not independent reproduction. All findings are SECOND_AGENT_REVIEW (Hermes). No third-party review has been performed.

---

## 14. Engineering Decision Contrasts

These are genuine architectural differences across projects — not variations of one platform:

| Decision | PRJ-01 | PRJ-02 | PRJ-04 | PRJ-05 | PRJ-11 |
|----------|--------|--------|--------|--------|--------|
| **Audit mechanism** | SHA-256 linear chain | JSONL + correlation IDs | Merkle tree (sorted leaves) | N/A | SHA-256 linear chain |
| **Tenant isolation** | RLS (partial) | Tenant-scoped SQLite | RLS FORCE + fail-closed | N/A | Tenant-scoped queries |
| **Model default** | N/A (Oya disabled) | EchoProvider (offline) | DeterministicFakeProvider (pure) | Simulation mode | Rules-before-model |
| **Human gate** | Advisory-only (no tools) | Hash-bound SHA-256 | Four-eyes at API | `live_authorized=False` | Approval queue + tenant |
| **Risk evaluation** | `max_requested_tools=0` | Heuristic (6 keywords) | 8-condition release gate | 9-check risk kernel | First-match rule engine |
| **CI status** | FAIL at HEAD | ALL 3 FAIL | PASS | PASS (recently fixed) | No CI |
| **Test count** | 1058 REPORTED | 725 REPORTED | 938 REPORTED | 2003 REPORTED | 856 independently counted |
| **License** | NOASSERTION | NOASSERTION | NONE | PROPRIETARY | Proprietary |

**Key insight:** The portfolio demonstrates distinct engineering styles across projects — not a single deployed platform. Each project makes domain-specific trade-offs. The contrasts in audit mechanisms, tenant isolation depth, and risk evaluation rigor show genuine engineering judgment applied to different problem domains.

---

## 15. Skeptical-Reviewer Navigation Path

For any claim in this synthesis, a reviewer can follow:

```
Claim in this document
    ↓
Case evidence file (cases/<NN>/engineering-evidence.md)
    ↓
Evidence ID (EVD-NN-NNN) in evidence matrix
    ↓
Source permalink (SHA-pinned GitHub URL)
    ↓
Limitation or gap (GAP-NN-NNN)
    ↓
Central index (docs/evidence/central-index.md) for cross-reference
```

**If a claim cannot be traced through this chain**, it is labeled INFERRED or NOT VERIFIED in this document. No claim is presented as verified without a source permalink or evidence ID.

**What this synthesis does NOT claim:**
- It does not claim all projects share one architecture.
- It does not claim any project is production-ready.
- It does not claim independent reproduction (R3) of any test result.
- It does not claim uniform testing or CI maturity.
- It does not claim that observed patterns imply runtime behavior.

---

## 16. Open Cross-Project Gaps

| Gap | Description | Affects |
|-----|-------------|---------|
| XGAP-01 | No project has reached R3 (independent clean-room reproduction) | All 13 |
| XGAP-02 | No project has production observability or operational telemetry (R5) | All 13 |
| XGAP-03 | No project has undergone independent penetration testing | PRJ-01, PRJ-02, PRJ-04, PRJ-08, PRJ-10, PRJ-11 |
| XGAP-04 | No test count has been independently reproduced (R3) — PRJ-11 was independently counted but not executed | All with tests |
| XGAP-05 | CI maturity is not uniform — 5/13 have CI; 2 of those fail at HEAD | PRJ-01, PRJ-02 (failing); PRJ-03, PRJ-06, PRJ-09, PRJ-11, PRJ-12 (no CI) |
| XGAP-06 | License clarity varies — 4 projects have no license file; 3 have NOASSERTION; 2 have proprietary | PRJ-04, PRJ-08, PRJ-09, PRJ-12 (none); PRJ-01, PRJ-02, PRJ-06 (NOASSERTION); PRJ-05, PRJ-11 (proprietary) |
| XGAP-07 | 2 projects (PRJ-07, PRJ-13) have no source repository — all claims are R0/E0 | PRJ-07, PRJ-13 |

---

## 17. 30-Filter Gate Record (Phase 8)

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 8 in full + final consistency audit + README synchronization |
| F02 Scope | PASS | Only `docs/engineering/cross-project-architecture.md` (new), `docs/evidence/central-index.md` (update), `README.md` (update), `DIRECTIVE.txt` (replace after validation) |
| F03 Source immutability | PASS | No source repository modified; all 13 source projects remain read-only |
| F04 Baseline | PASS | Branch `main`, HEAD `2ab932c`, clean working tree verified before edits |
| F05 Conflict | PASS | No existing directive or owner work conflicts |
| F06 Intent | PASS | Cross-project synthesis advances evidence-first portfolio objective |
| F07 Necessity | PASS | Synthesis is the Phase 8 deliverable; no simpler non-writing alternative |
| F08 Duplication | PASS | Existing case evidence referenced, not duplicated; synthesis adds comparative analysis |
| F09 Provenance | PASS | All source SHAs pinned in case files; this document links to evidence IDs |
| F10 Authenticity | PASS | No source statement treated as independently verified; R3 explicitly NOT TESTED |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/INFERRED/NOT_TESTED used precisely throughout |
| F12 Reproducibility | PASS | Every claim links to case evidence or is labeled INFERRED/NOT VERIFIED |
| F13 Technical correctness | PASS | Architecture comparisons match actual source evidence in case files |
| F14 Domain safety | PASS | Financial, space, health, email safety boundaries respected |
| F15 Trust boundaries | PASS | Source files treated as untrusted data; no prompt injection followed |
| F16 Security | PASS | No injection, unsafe links, or exposed attack details introduced |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific information leaked |
| F18 IP | PASS | No source code copied; only attributed descriptions and SHA-pinned links |
| F19 Attribution | PASS | Source author (Amin Azimi, Azimi Innovation Lab) accurately attributed |
| F20 Test integrity | PASS | No unexecuted test described as passing; test counts labeled REPORTED where applicable |
| F21 Failure modes | PASS | Negative cases and known limitations represented per project |
| F22 Version drift | PASS | SHAs pinned at observation date; source drift acknowledged |
| F23 Interface stability | PASS | Existing README, case ordering, and links preserved |
| F24 Accessibility | PASS | Headings, tables, and text navigation are readable |
| F25 Maintainability | PASS | Concise structured document; no second source tree created |
| F26 Reviewability | PASS | Diff is one new synthesis file + central-index update + README update |
| F27 Validation | PASS | Source links point to pinned SHAs; evidence IDs referenced |
| F28 Git hygiene | PASS | One meaningful commit for Phase 8 synthesis (authorized) |
| F29 Disclosure | PASS | No public-safe content beyond evidence; no production claims |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision:** PROCEED — all 30 filters PASS; no blocking evidence.
