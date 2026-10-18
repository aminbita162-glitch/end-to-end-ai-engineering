# PRJ-01 — AILORA: Engineering Evidence

**Project ID:** PRJ-01\
**Canonical name:** AILORA — Azimi Innovation Lab Orbital Intelligence System\
**Source:** https://github.com/azimilab2025-ai/ailora\
**Source HEAD (Phase 3 inspection):** `46ca4a939bee0396e90abef91905b6d1b95d470c` (2026-08-31T12:06:12Z)\
**Freeze commit message:** `chore(freeze): lock judge surface 2026-08-31 14:00 Europe/Berlin`\
**Release tag (canonical):** `candidate-2026-08-29` (published 2026-08-29T11:55:25Z, non-prerelease)\
**Release tag target commit:** `33090bd47aa67bf917a3bf9d87e9e3836fa5edbf` (lightweight tag)\
**License:** NOASSERTION — license file present; not SPDX-recognized\
**Phase 3 inspection date (UTC):** 2026-10-17\
**Independence:** Source/documentation observations only; no independent third-party verification claimed.\
**Independence level:** Source/documentation observations only; no independent reproduction claimed.\
**Evidence maturity:** R1 — source and configuration inspected at immutable revision\
**Reproducibility ladder:** R1 (source inspected). R2 partially achieved — CI run artifact `33025990734` at SHA `9571b168080605b72a464d39d1fed5239242330d` linked by author (2026-08-27). CI run at HEAD `46ca4a939bee` FAILED (run `33390090146`, 2026-08-31). See §6.

---

## 1. Context and Scope

AILORA is an advisory-only, human-authority-first Space Situational Awareness (SSA) platform. It provides orbital scenario management, screening, risk assessment, and review workflows for a candidate candidate under active qualification. The domain is astrodynamics-informed SSA; the system is explicitly scoped to advisory results — it does not issue spacecraft commands.

**Status (OBSERVED in `docs/qualification/status-contract.md` and `AILORA_FREEZE_RECORD.txt`):**

| Field | Value |
|-------|-------|
| ENGINEERING_MATURITY | PRODUCTION_GRADE_CANDIDATE |
| QUALIFICATION_STATE | ACTIVE_QUALIFICATION |
| PRODUCTION_READY | FALSE |
| PRODUCTION_QUALIFIED | FALSE |
| RELEASE_AUTHORIZED | FALSE |
| CANDIDATE_RUNTIME_AUTH | HS256 (Render live path) |
| RS256_JWKS | FOUNDATION / NOT YET OPERATIONAL |
| OIDC_MFA | FOUNDATION / NOT YET OPERATIONAL |

**Non-goals (OBSERVED in `docs/governance/residual-risk-and-release-decision.md`):**

- Not production-ready or production-qualified.
- No spacecraft command, uplink, telecommand, or autonomous maneuver path.
- No live NASA data activated.
- In-repo Oya agent is a disabled library agent, not a hosted runtime.
- No scientific independence approval obtained (OPEN_EXTERNAL_GATE).
- No legal/privacy review completed (OPEN_EXTERNAL_GATE).

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading `docs/architecture/enterprise-adr-register.md`, source tree, and qualification documents at HEAD SHA `46ca4a939bee`. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
HTTP Request (TLS via Render)
     │
     ▼
┌─────────────────────────────────────────────────┐
│  FastAPI Routers  (src/ailora/api/routers/)     │
│  - Versioned /api/v1 + /health endpoints        │
│  - Request identity + correlation middleware     │
│  - Pydantic contract validation                 │
│  - FastAPI Depends() injection                  │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│  Security Layer  (src/ailora/security/)         │
│  - asymmetric_tokens.py — RS256 foundation      │
│  - auth.py — HS256 Render runtime path          │
│  - contextual_authorization.py — per-action     │
│  - workload_identity.py — short-lived svc IDs   │
│  - supply_chain.py — Ed25519 evidence admission │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│  Domain  (src/ailora/domain/)                   │
│  - identity/: User, Tenant, Membership, roles   │
│  - ssa/: scenario, screening, risk, review,     │
│          audit, audit_integrity                 │
│  - space_data/: contracts, models, repository   │
│  - workflows/: durable workflow state machine   │
│  - shared/: value objects (Epoch, CartesianState│
│              TemporalStamp — pure dataclasses)  │
└──────┬───────────────────────────────┬──────────┘
       │                               │
       ▼                               ▼
┌────────────────┐         ┌────────────────────────────────┐
│  Repositories  │         │  Services                      │
│  - AsyncSQLAlch│         │  - astrodynamics/: SGP4, TCA,  │
│  - Tenant-keyed│         │    covariance, frame transform, │
│  - PostgreSQL  │         │    independent verification     │
│  - RLS defense │         │  - space_data/: provider ingest │
└───────┬────────┘         │    resilience, governance       │
        │                  │  - oya/: disabled library agent │
        ▼                  │    safety.py, typed_plan.py     │
┌──────────────────┐       │  - workflows/: processor        │
│  PostgreSQL      │       └────────────────────────────────┘
│  14 migrations   │
│  RLS + tenant    │
│  context         │
└──────────────────┘
```

Source: [`docs/architecture/enterprise-adr-register.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/architecture/enterprise-adr-register.md) — OBSERVED.

**Design principle (OBSERVED in ADR register):** "Domain → service → API dependency direction with typed persistence" (ADR-002, `ACCEPTED_IMPLEMENTED`). Value objects are pure Python dataclasses with no ORM coupling — observed in `src/ailora/domain/shared/value_objects.py`.

### 2b. Tenant Isolation Architecture (OBSERVED in source)

Two-layer isolation enforced at both application and database levels:

**Application layer** (`src/ailora/db/tenant_context.py` — OBSERVED):

- `TenantDatabaseContext` immutable dataclass carries `tenant_id`, `actor_id`, `correlation_id` plus PostgreSQL timeout budgets (`statement_timeout_ms`, `lock_timeout_ms`, `idle_transaction_timeout_ms`).
- `context_from_request()` extracts `tenant_id` from path parameters (not request body); `actor_id` from middleware-set `request.state`.
- `@event.listens_for(Session, "after_begin")` fires `set_config()` for each transaction-local setting (`app.current_tenant_id`, `app.current_actor_id`, `app.correlation_id`, timeouts). PostgreSQL dialect check prevents silent SQLite bypass.

**Database layer** (migration `alembic/versions/0013_postgres_tenant_rls.py` — OBSERVED in tree):

- PostgreSQL Row Level Security policies apply `app.current_tenant_id` session variable.
- ADR-003 status: `ACCEPTED_PARTIAL` — tenant-scoped models and RLS foundation present; full multi-tenant defense continues in qualification commands.

```python
# src/ailora/db/tenant_context.py — OBSERVED at HEAD
@event.listens_for(Session, "after_begin")
def _apply_postgres_transaction_context(session, transaction, connection):
    context = session.info.get(SESSION_CONTEXT_KEY)
    if not isinstance(context, TenantDatabaseContext):
        return
    if connection.dialect.name != "postgresql":
        return  # no-op on SQLite test dialect
    statement = text("SELECT set_config(:setting_name, :setting_value, true)")
    for setting_name, setting_value in transaction_settings(context):
        connection.execute(statement, {"setting_name": setting_name, "setting_value": setting_value})
```

### 2c. Database Migrations (OBSERVED in tree)

14 Alembic migrations from `0001_baseline` through `0014_audit_integrity`:

```
0001_baseline
0002_identity
0003_identity_sessions
0004_ssa_scenarios
0005_ssa_screenings
0006_ssa_risk_assessments
0007_ssa_reviews
0008_ssa_audit_events
0009_space_data_contracts
0009a_expand_alembic_version
0010_space_data_provider_governance
0011_durable_workflows
0012_space_data_frame_transformations
0013_postgres_tenant_rls
0014_audit_integrity
```

Async Alembic environment (`alembic/env.py`) using `asyncpg` in production. `aiosqlite` in tests.

### 2d. Oya Agent Architecture (OBSERVED in source)

The in-repository Oya component is a **disabled library agent**. Its live integration is with the external `getoya.ai` service, which was tested during the qualification window (OBSERVED in real-system qualification record).

**In-repo boundary** (`src/ailora/services/oya/safety.py` — OBSERVED):

```python
class OyaSafetyPolicy:
    max_input_characters: int = 4096
    max_requested_tools: int = 0  # tool execution forbidden in candidate

    def evaluate(self, text, requested_tools=()):
        # Prompt injection markers → REJECTED + human_review_required=True
        # Authority override markers → REJECTED
        # Tool requests → REJECTED (TOOL_EXECUTION_FORBIDDEN)
        # All other → FALLBACK (OYA_DISABLED)
```

**Status contract** (OBSERVED in `docs/qualification/status-contract.md`):

- `OYA_REPOSITORY_COMPONENT_STATUS = LIBRARY_AGENT_PRESENT`
- `OYA_EXTERNAL_INTEGRATION_STATUS = TESTED`
- `OYA_QUALIFICATION_STATUS = PARTIAL`
- `OYA_RELEASE_AUTHORITY = NOT_APPROVED`

The `max_requested_tools = 0` enforces tool execution prohibition at the candidate stage. Advisory-only is a code-enforced boundary in the in-repo component.

---

## 3. SHA Register and Release Evidence

| Item | Value | Observation class |
|------|-------|-------------------|
| Source HEAD (Phase 3) | `46ca4a939bee0396e90abef91905b6d1b95d470c` | OBSERVED — GitHub API 2026-10-17 |
| Release tag (canonical) | `candidate-2026-08-29` | OBSERVED — GitHub API releases endpoint |
| Release tag target commit | `33090bd47aa67bf917a3bf9d87e9e3836fa5edbf` | OBSERVED — GitHub API refs endpoint |
| Release published | 2026-08-29T11:55:25Z | OBSERVED |
| Release pre-release flag | `false` (non-prerelease) | OBSERVED |
| Prerelease tag | `candidate-2026-08-30` | OBSERVED — `prerelease: true` |
| Earliest tag | `v0.1.0` (2026-08-06T03:31:03Z) | OBSERVED |
| Qualified CI run | `33025990734` at `9571b168080605b72a46` (2026-08-27) | OBSERVED — GitHub Actions API |
| Qualified CI conclusion | `success` | OBSERVED |
| HEAD CI run | `33390090146` at `46ca4a939bee` (2026-08-31) | OBSERVED |
| HEAD CI conclusion | `failure` | OBSERVED (GAP-01-001) |

**CI failure at HEAD (GAP-01-001):** The final commit (`chore(freeze): lock judge surface`) triggered a failing CI run. The qualification evidence cited in `docs/verification.md` references CI run `33025990734` at SHA `9571b168080605b72a46` (2026-08-27) with "1058 passing tests, 87.91% statement coverage". This qualified run SHA is different from the freeze HEAD. The HEAD CI failure was introduced by the freeze commit. Root cause of the specific test failure is NOT INSPECTED (no workflow log content accessed). The portfolio case must disclose this CI state clearly.

---

## 4. Authentication Architecture (OBSERVED in source)

AILORA implements a two-path authentication model: a production-target RS256/OIDC path (foundation-only, not operational) and a candidate runtime HS256 path (live on Render).

### RS256 Foundation (`src/ailora/security/asymmetric_tokens.py` — OBSERVED)

```python
_ALGORITHM: Final = "RS256"
_MINIMUM_RSA_BITS: Final = 2048

@dataclass(frozen=True, slots=True)
class AsymmetricTokenProfile:
    issuer: str       # must be explicit credential-free HTTPS URL
    audience: str     # must be explicit and non-empty
    access_token_ttl: timedelta  # 1 second to 1 hour
    algorithm: Literal["RS256"] = "RS256"
```

- Validates RSA key size (minimum 2048-bit) on key registration.
- Issuer must be a credential-free HTTPS URL (validated in `__post_init__`).
- Audience validation required on decode.
- **Status:** FOUNDATION — code exists and is tested; not the live Render login path.

### Candidate Runtime Auth (`src/ailora/security/auth.py` — OBSERVED in tree)

- HS256 symmetric path — used by Render deployment.
- Status contract explicitly states: `CANDIDATE_RUNTIME_AUTH = HS256 local/runtime path used by login and session on Render`.
- RS256 is explicitly NOT described as the live candidate login path.

### Contextual Authorization (`src/ailora/security/contextual_authorization.py` — OBSERVED)

Fine-grained permission model:

```python
class ContextualPermission(StrEnum):
    SCENARIO_READ = "scenario:read"
    SCENARIO_WRITE = "scenario:write"
    SCREENING_READ = "screening:read"
    SCREENING_SUBMIT = "screening:submit"
    REVIEW_READ = "review:read"
    REVIEW_DECIDE = "review:decide"
    AUDIT_READ = "audit:read"
    MEMBERSHIP_MANAGE = "membership:manage"
```

- `ActorKind` separates HUMAN and WORKLOAD identities.
- `DataClassification` (INTERNAL/SENSITIVE/RESTRICTED) constrains grants on sensitive resources.
- `_HUMAN_MAX_AUTH_AGE: timedelta(hours=8)` and `_WORKLOAD_MAX_AUTH_AGE: timedelta(minutes=15)` bound token lifetimes by actor kind.

### Supply-Chain Admission (`src/ailora/security/supply_chain.py` — OBSERVED)

Ed25519-signed evidence manifest verification. Decision string: `EVIDENCE_VERIFIED_NOT_DEPLOYMENT_AUTHORIZATION` — explicitly separates evidence verification from deployment authorization.

---

## 5. Audit Integrity Architecture (OBSERVED in source)

SHA-256 hash chain for audit events (`src/ailora/domain/ssa/audit_integrity.py` — OBSERVED):

```python
GENESIS_HASH = "0" * 64

def canonical_audit_payload(record: AuditChainRecord) -> str:
    """Return the length-prefixed canonical payload shared with PostgreSQL."""
    # Length-prefixed canonicalization of: tenant_id, id, actor_user_id,
    # event_type, resource_type, resource_id, outcome, correlation_id,
    # detail, combined_classification, advisory_only, timestamp_utc,
    # sequence_no, previous_hash
```

- Each audit event carries `previous_hash` forming a tamper-evident chain.
- Chain verification (`AuditIntegrityError`) detects discontinuity or cryptographic mismatch.
- Migration `0014_audit_integrity` applies append-only database constraints.
- Threat model entry THR-006: "append-only roles, content addressing, signed batches/WORM where applicable."

---

## 6. Test Architecture (OBSERVED in source)

Source: `tests/` at HEAD (91 test files) — OBSERVED in tree.

### Test File Categories (OBSERVED in tree)

| Category | Representative files | Coverage area |
|----------|---------------------|---------------|
| Astrodynamics | `test_astrodynamics_propagation.py`, `test_astrodynamics_tca.py`, `test_astrodynamics_covariance.py`, `test_astrodynamics_frame_transform.py`, `test_astrodynamics_verification_contracts.py` (10 files) | SGP4 propagation, TCA, covariance, frame/time transforms, verification contracts |
| Security | `test_asymmetric_tokens.py`, `test_auth.py`, `test_authorization_isolation.py`, `test_contextual_authorization.py`, `test_workload_identity.py`, `test_runtime_edge_security.py` | RS256 foundation, JWT auth, cross-tenant isolation, contextual permissions |
| Tenant/Identity | `test_identity_models.py`, `test_tenant_identity_management.py`, `test_tenant_repositories.py`, `test_isolation.py`, `test_postgres_tenant_defense.py` | Identity models, tenant isolation, RLS |
| SSA domain | `test_scenario_ingestion.py`, `test_screening.py`, `test_risk_assessment.py`, `test_review_state.py`, `test_audit.py`, `test_audit_integrity.py` | SSA workflow end-to-end |
| Space data | `test_space_data_contracts.py`, `test_space_data_provider_governance.py`, `test_space_data_provider_outages.py` | Provider contracts, governance, outage handling |
| Resilience | `test_durable_workflow_failures.py`, `test_durable_workflow_replay.py`, `test_recovery_failures.py`, `test_recovery_qualification.py` | Failure injection, replay, recovery |
| Observability | `test_observability.py`, `test_workflow_observability.py`, `test_workflow_slo.py` | Logging, tracing, SLO |
| CI / Infrastructure | `test_ci_baseline.py`, `test_docker_contracts.py`, `test_artifact_sbom.py`, `test_supply_chain_admission.py` | CI structural contracts, SBOM, supply-chain admission |
| Release evidence | `test_final_evidence_pack.py`, `test_final_release_evidence.py`, `test_release_decision_contract.py` | Release manifest integrity |
| Oya safety | `test_oya.py`, `test_oya_safety_qualification.py` | Oya safety policy, injection rejection |

### Qualification CI Evidence

| Item | Value | Source |
|------|-------|--------|
| Qualified CI run ID | `33025990734` | OBSERVED — GitHub Actions API |
| Qualified CI SHA | `9571b168080605b72a464d39d1fed5239242330d` | OBSERVED |
| Qualified CI date | 2026-08-27T00:11:21Z | OBSERVED |
| Qualified CI conclusion | `success` | OBSERVED |
| Claimed test count | 1058 passing | REPORTED in `docs/verification.md` PHASE_0 note |
| Claimed coverage | 87.91% statement | REPORTED in `docs/verification.md` PHASE_0 note |
| Coverage floor in CI | 85% measured (`uv run pytest tests/ -v`) | OBSERVED in `.github/workflows/ci.yml` |
| HEAD CI run (`46ca4a939bee`) | **FAILURE** (run `33390090146`, 2026-08-31) | OBSERVED |

**Test count assessment:** 1058-test and 87.91% coverage claims are REPORTED by author in `docs/verification.md`. CI run `33025990734` OBSERVED as `success` at SHA `9571b168080605b72a46` (2026-08-27). This confirms a qualifying CI run exists — but at an intermediate SHA, not the freeze HEAD. The freeze-commit HEAD CI run fails. Test count independent verification (counting 91 files × average per file) NOT TESTED. Evidence strength: E2 (CI run artifact observed and linked) for the qualifying run; E0 for the precise test count.

### Negative Case Coverage (OBSERVED in test tree)

- `test_isolation.py` — cross-tenant read attempts.
- `test_authorization_isolation.py` — permission boundary enforcement.
- `test_postgres_tenant_defense.py` — database-level tenant defense.
- `test_astrodynamics_verification_failures.py` — verification failure paths.
- `test_space_data_provider_outages.py` — provider outage handling.
- `test_durable_workflow_failures.py` — workflow failure and DLQ.
- `test_recovery_failures.py` — recovery failure injection.
- `test_oya_safety_qualification.py` — safety policy rejection paths.
- `test_runtime_edge_security.py` — edge security boundary tests.

---

## 7. Observability Architecture (OBSERVED in source)

`src/ailora/observability/` (OBSERVED in tree):

| Module | Content |
|--------|---------|
| `logging.py` | Structured logging using `structlog` + JSON output |
| `tracing.py` | OpenTelemetry tracing, OTLP/HTTP export |
| `workflow_metrics.py` | Workflow-level metrics |
| `workflow_slo.py` | SLO/error budget contracts |

ADR-012: `ACCEPTED_PARTIAL` — OpenTelemetry signals, SLO/error budget, and scientific disagreement metrics. Full operational telemetry backend not yet specified (deferred per ADR notes: "vendor, region, queue product, monitoring backend remain provider-neutral").

---

## 8. Residual Risk and Release Decision (OBSERVED in source)

Source: `docs/governance/residual-risk-and-release-decision.md` — OBSERVED at HEAD.

Final engineering decision: `CONDITIONAL_LOCAL_Production-Grade_PASS_PRODUCTION_BLOCKED`.

| Risk ID | Description | Severity | Status |
|---------|-------------|----------|--------|
| SCI-001 | Astrodynamics validity not independently approved | CRITICAL | OPEN_EXTERNAL_GATE |
| DATA-001 | Live NASA/CelesTrak source rights and operations not qualified | HIGH | OPEN_EXTERNAL_GATE |
| LEGAL-001 | License, privacy, residency, regulatory unreviewed | CRITICAL | OPEN_EXTERNAL_GATE |
| OYA-001 | Vendor, security, residency, commercial terms unknown | HIGH | OPEN_EXTERNAL_GATE |
| OPS-001 | Production infra, soak, pentest, signing, alerting, DR absent | CRITICAL | OPEN_EXTERNAL_GATE |

`PRODUCTION_RELEASE=BLOCKED_PENDING_P0_GATES`. `HUMAN_RELEASE_AUTHORITY=MANDATORY`. The portfolio may describe this as a Production Grade Candidate baseline only.

---

## 9. ADR-01-001 — Advisory-Only Platform with No Spacecraft Command Path

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-01-001 |
| **Source SHA** | `46ca4a939bee0396e90abef91905b6d1b95d470c` |
| **Source path** | [`docs/architecture/enterprise-adr-register.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/architecture/enterprise-adr-register.md) |
| **Problem** | Space systems producing incorrect or unauthorized outputs carry safety-critical risk. An astrodynamics platform accessible via API needs a clear boundary excluding any command, uplink, telecommand, or maneuver path. |
| **Constraints** | System handles orbital data that informs safety-relevant decisions; humans must remain the authority for any consequential action. |
| **Selected approach** | `ADR-001: Advisory, human-authority-first platform with no spacecraft command path` (`ACCEPTED_IMPLEMENTED_BOUNDARY`). Safety policy in `oya/safety.py` enforces `max_requested_tools = 0`. Residual risk document and status contract explicitly confirm no command surface. Threat model entries THR-013 (prompt injection) and prohibited transitions enforce this at multiple layers. |
| **Alternatives** | NOT DOCUMENTED in source. |
| **Trade-offs** | Constrains product scope; does not address AI autonomous operation risk (which is the point). Any future tool expansion requires explicit `OYA_RELEASE_AUTHORITY` gate. |
| **Evidence** | OBSERVED in source at HEAD SHA; consistent across status-contract, threat-model, residual-risk, and safety.py |
| **Status** | Active — preserved across all qualification commands per ADR notes |

---

## 10. ADR-01-002 — HS256 Candidate Runtime / RS256 Production Target

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-01-002 |
| **Source SHA** | `46ca4a939bee0396e90abef91905b6d1b95d470c` |
| **Source path** | [`src/ailora/security/asymmetric_tokens.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/security/asymmetric_tokens.py), [`docs/qualification/status-contract.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/qualification/status-contract.md) |
| **Problem** | Production-grade identity federation requires RS256 (asymmetric) tokens and OIDC. Implementing this before other qualification gates creates infrastructure risk; building only HS256 and not preparing the RS256 path creates a late migration burden. |
| **Constraints** | Render free-tier deployment; OIDC provider not yet selected; qualification milestone schedule. |
| **Selected approach** | Build RS256 foundation (code present, tested, with 2048-bit minimum enforcement and HTTPS-only issuer validation) but use HS256 as the Render candidate runtime path. Status contract explicitly documents this split: `CANDIDATE_RUNTIME_AUTH = HS256`. RS256 and OIDC are labeled `FOUNDATION / NOT YET OPERATIONAL`. |
| **Alternatives** | NOT DOCUMENTED. Plausible: full RS256 from day one (higher infra complexity); defer RS256 entirely (creates late migration gap). |
| **Trade-offs** | Honest dual-path design separates engineering progress from claimed operational state. Risk: HS256 symmetric secret management on Render is weaker than production RS256 + OIDC. ADR-004 (`ACCEPTED_TARGET`) documents the full OIDC production target. |
| **Evidence** | OBSERVED — asymmetric_tokens.py present and reviewed; status-contract.md explicitly records the split |
| **Status** | ADR-004 (`ACCEPTED_TARGET`) — OIDC path remains a qualification command target |

---

## 11. ADR-01-003 — Append-Only Audit Chain with SHA-256 Hash Integrity

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-01-003 |
| **Source SHA** | `46ca4a939bee0396e90abef91905b6d1b95d470c` |
| **Source path** | [`src/ailora/domain/ssa/audit_integrity.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/domain/ssa/audit_integrity.py) |
| **Problem** | An advisory SSA platform making human-authority decisions must maintain a tamper-evident audit trail. Relying solely on a mutable database column is insufficient for a qualify-to-release evidence chain. |
| **Selected approach** | SHA-256 hash chain: each event carries a `previous_hash` field; canonical audit payload uses length-prefixed serialization shared with PostgreSQL (enabling server-side verification); `AuditIntegrityError` raised on discontinuity or hash mismatch. Migration `0014_audit_integrity` adds append-only database constraints. |
| **Alternatives** | NOT DOCUMENTED. |
| **Trade-offs** | Chain construction is deterministic and verifiable without external tooling. Risk: chain verification currently a software-layer control; database-level immutable constraints and WORM storage remain on the qualification roadmap (THR-006). |
| **Evidence** | OBSERVED in source at HEAD SHA |
| **Status** | Active — `test_audit_integrity.py` OBSERVED in test tree |

---

## 12. Critical Control Flows

### Flow 1: SSA Screening Submission (RECONSTRUCTED FROM SOURCE)

```
Client POST /api/v1/tenants/{tenant_id}/screenings
    │
    ▼  TLS (Render)
    │
    ▼  FastAPI router: extract tenant_id from path
    │
    ▼  Security dependency: decode token
         - HS256 (candidate) or RS256 (target)
         - Validate audience, expiry, kid, jti
         - Derive actor identity
    │
    ▼  context_from_request() → TenantDatabaseContext
         - tenant_id from path (not body)
         - correlation_id from middleware
    │
    ▼  ContextualAuthorization: SCREENING_SUBMIT permission check
         - ActorKind (HUMAN or WORKLOAD) checked
         - DataClassification ceiling applied
         - _HUMAN_MAX_AUTH_AGE / _WORKLOAD_MAX_AUTH_AGE enforced
    │
    ▼  @event.listens_for "after_begin": set_config() injects tenant context
         into PostgreSQL transaction
    │
    ▼  ScreeningService: validate + persist screening
    │
    ▼  AuditService: append audit event with hash chain extension
    │
    ▼  HTTP 201 with screening resource
```

**Trust boundary enforcement:** `tenant_id` injected from path, cross-validated against token actor's membership. PostgreSQL RLS applies `app.current_tenant_id` as a second enforcement layer. OBSERVED in source. NOT independently tested.

### Flow 2: Oya Advisory Query (OBSERVED in source)

| Condition | Enforcement | Source |
|-----------|-------------|--------|
| Tool execution requested | `REJECTED (TOOL_EXECUTION_FORBIDDEN)` — `max_requested_tools = 0` | OBSERVED — `safety.py` |
| Prompt injection marker detected | `REJECTED (PROMPT_INJECTION)` + `human_review_required=True` | OBSERVED — `safety.py` |
| Authority override marker | `REJECTED (FALSE_AUTHORITY)` | OBSERVED — `safety.py` |
| Input exceeds 4096 chars | `REJECTED (INPUT_BUDGET_EXCEEDED)` | OBSERVED — `safety.py` |
| Normal input | `FALLBACK (OYA_DISABLED)` | OBSERVED — `safety.py` |

No advisory tool execution can reach the in-repo Oya path until `max_requested_tools` is raised and `OYA_RELEASE_AUTHORITY = APPROVED` in the status contract. Both conditions are explicit gates.

### Flow 3: Supply-Chain Admission (OBSERVED in source)

- `supply_chain.py` verifies Ed25519-signed JSON evidence manifests.
- `DECISION = "EVIDENCE_VERIFIED_NOT_DEPLOYMENT_AUTHORIZATION"` — explicitly prevents treating verified evidence as a deployment gate pass.
- `VexStatus` tracks vulnerability findings (NOT_AFFECTED, FIXED, KNOWN_AFFECTED, UNDER_INVESTIGATION).
- `ScanKind`: SAST, SCA, CONTAINER, SECRET_HISTORY.

---

## 13. Threat Model Assessment (OBSERVED in source)

Source: [`docs/assurance/threat-model.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/assurance/threat-model.md) — OBSERVED at HEAD.

Status: `ACTIVE_BASELINE_NOT_INDEPENDENTLY_ASSESSED`.

| Threat | Control | Source treatment |
|--------|---------|-----------------|
| THR-001: Forged/replayed token | Asymmetric validation, issuer/audience/kid/jti, short TTL, revocation | ENFORCED in foundation; HS256 runtime path has no jti revocation (limitation) |
| THR-002: BOLA/IDOR cross-tenant | Tenant binding, contextual authorization, RLS | ENFORCED at application layer (OBSERVED); database RLS `ACCEPTED_PARTIAL` |
| THR-004: SSRF/provider URL manipulation | Allowlist, TLS, no redirects, resolved-IP policy, egress proxy | DOCUMENTED target; runtime enforcement depends on unspecified provider infrastructure |
| THR-006: Audit record tampering | Append-only roles, content-addressed chain, signed batches | ENFORCED at application layer; WORM/database-level constraints remain on roadmap |
| THR-013: Prompt injection / authority override | Allowlisted advisory tools, deterministic authorization, no command path | ENFORCED in `oya/safety.py` (OBSERVED) |
| THR-014: Release authority ignores missing evidence | Immutable manifest, deny-overrides gate, separation of duties | ENFORCED in `supply_chain.py` and residual-risk-and-release-decision.md |

**Status convention (ADR document, OBSERVED):** A design document, dependency import, or test name alone cannot advance an ADR to implemented or qualified. This is the same rigor applied throughout this case.

---

## 14. Space Data and Astrodynamics Scope (OBSERVED in source)

The scientific scope involves TLE parsing, SGP4 propagation, Time of Closest Approach (TCA) computation, covariance handling, frame transformation (TEME/ITRF/GCRF), and EOP application. The system explicitly tracks scientific correctness as an unresolved qualification gate:

- ADR-007 (`ACCEPTED_PARTIAL`): Primary scientific engine plus independent reference path and conflict state — verification contracts framework exists; independence not yet qualified.
- ADR-009 (`ACCEPTED_TARGET`): Covariance-capable data precedes qualified Pc assessment — not yet implemented.
- `risk_semantics = DISTANCE_BASED_PROXIMITY_SEVERITY_NOT_COLLISION_PROBABILITY` (OBSERVED in `final-release-manifest.json`) — risk is reported as proximity severity, not collision probability (Pc). This boundary is explicitly documented.

`docs/qualification/independent-astrodynamics-verification.md` and `enhanced-astrodynamics-analysis.md` OBSERVED in tree — contents not inspected in full; existence indicates qualification work is ongoing.

---

## 15. Failure Modes and Limitations

| Failure mode | Source treatment | Evidence class |
|--------------|-----------------|----------------|
| CI failing at HEAD `46ca4a939bee` | Run `33390090146` FAILURE (2026-08-31) | OBSERVED |
| RS256/OIDC not operational on Render | Status contract explicitly states HS256 is the live path | OBSERVED |
| Oya tool execution disabled | `max_requested_tools = 0`; `OYA_RELEASE_AUTHORITY = NOT_APPROVED` | OBSERVED |
| Scientific independence gate open | SCI-001: CRITICAL, OPEN_EXTERNAL_GATE | OBSERVED in residual-risk doc |
| Legal/privacy gate open | LEGAL-001: CRITICAL, OPEN_EXTERNAL_GATE | OBSERVED in residual-risk doc |
| Live NASA data not activated | Explicitly documented: `LIVE_NASA_DATA=NOT_ACTIVATED` | OBSERVED |
| jti revocation absent on HS256 path | No revocation store observed for candidate path | INFERRED from security inspection |
| ADR-003 partial (RLS) | Tenant-scoped models present; full PostgreSQL RLS qualification ongoing | OBSERVED (ADR status `ACCEPTED_PARTIAL`) |
| ADR-010 partial (durable workflows) | Stateless/durable separation foundation present; C26 not yet complete | OBSERVED (ADR status `ACCEPTED_PARTIAL`) |
| Production infrastructure | OPS-001: CRITICAL, OPEN_EXTERNAL_GATE — no pentest, no signing, no DR drill | OBSERVED |
| Independent scientific review | SCI-001 cannot be self-issued (ADR-014 `EXTERNAL_GATE`) | OBSERVED |
| License not SPDX-recognized | NOASSERTION — copying requires rights confirmation | OBSERVED |

---

## 16. Evidence Matrix

| Evidence ID | Claim | Observation class | Result | Method | Strength |
|-------------|-------|-------------------|--------|--------|----------|
| EVD-01-001 | Qualification status is PRODUCTION_GRADE_CANDIDATE / ACTIVE_QUALIFICATION (not production-ready) | OBSERVED | PASS | `docs/qualification/status-contract.md` + `AILORA_FREEZE_RECORD.txt` at HEAD | E2 |
| EVD-01-002 | Release tag `candidate-2026-08-29` exists (non-prerelease) | OBSERVED | PASS | GitHub API releases endpoint | E2 |
| EVD-01-003 | CI run `33025990734` at SHA `9571b168` succeeded (2026-08-27) | OBSERVED | PASS | GitHub Actions API | E2 (R2 — linked CI run artifact) |
| EVD-01-004 | HEAD CI run `33390090146` FAILED (2026-08-31) | OBSERVED | FAIL | GitHub Actions API | E2 |
| EVD-01-005 | Advisory-only boundary enforced in `oya/safety.py` (`max_requested_tools = 0`) | OBSERVED | PASS | Source file inspection | E2 |
| EVD-01-006 | Tenant context injected per-transaction via SQLAlchemy event listener | OBSERVED | PASS | `src/ailora/db/tenant_context.py` inspection | E2 |
| EVD-01-007 | RS256 foundation present and tested; NOT the live Render login path | OBSERVED | PASS | `asymmetric_tokens.py` + status-contract.md | E2 |
| EVD-01-008 | SHA-256 audit integrity chain implemented | OBSERVED | PASS | `src/ailora/domain/ssa/audit_integrity.py` | E2 |
| EVD-01-009 | Production release BLOCKED pending 5 OPEN_EXTERNAL_GATE items | OBSERVED | PASS | `docs/governance/residual-risk-and-release-decision.md` | E2 |
| EVD-01-010 | 14 Alembic migrations including RLS (0013) and audit integrity (0014) | OBSERVED | PASS | `alembic/versions/` tree | E2 |
| EVD-01-011 | Threat model covers 15 STRIDE threats with explicit prohibited transitions | OBSERVED | PASS | `docs/assurance/threat-model.md` | E2 |
| EVD-01-012 | Ed25519 supply-chain admission explicitly not a deployment authorization | OBSERVED | PASS | `src/ailora/security/supply_chain.py` (DECISION constant) | E2 |
| EVD-01-013 | 91 test files in `tests/` including negative isolation, failure injection, and safety tests | OBSERVED | PASS | Git tree at HEAD | E2 (file count); independent function count NOT TESTED |
| EVD-01-014 | 1058 passing tests / 87.91% coverage claimed at qualifying SHA | REPORTED | CONDITIONAL | `docs/verification.md` PHASE_0 note; CI run `33025990734` linked | E2 (run observed) / E1 (count REPORTED) |
| EVD-01-015 | Render deployment live at `candidate-2026-08-29` qualification window | REPORTED | NOT_TESTED | `docs/qualification/real-system-qualification-record.md` + `AILORA_FREEZE_RECORD.txt` | E1 |
| EVD-01-016 | Risk semantics explicitly DISTANCE_BASED_PROXIMITY_SEVERITY not collision probability | OBSERVED | PASS | `docs/qualification/final-release-manifest.json` | E2 |
| EVD-01-017 | ADR-014: scientific, security, legal, SRE, release decisions cannot be self-issued | OBSERVED | PASS | `docs/architecture/enterprise-adr-register.md` | E2 |

---

## 17. Open Gaps After Phase 3

| Gap ID | Description | Risk level |
|--------|-------------|------------|
| GAP-01-001 | CI FAILURE at HEAD `46ca4a939bee` (run `33390090146`, 2026-08-31). Root cause of specific test failure NOT inspected. Qualifying run at earlier SHA `9571b168` is SUCCESS. | HIGH |
| GAP-01-002 | RS256/OIDC path (ADR-004) remains `ACCEPTED_TARGET` — not operational on Render. Production identity federation is a future qualification gate. | MEDIUM |
| GAP-01-003 | SCI-001 astrodynamics independence: CRITICAL OPEN_EXTERNAL_GATE — cannot be closed by repository self-attestation | CRITICAL (external) |
| GAP-01-004 | OPS-001 production infrastructure: CRITICAL OPEN_EXTERNAL_GATE — no pentest, no production signing, no DR drill | CRITICAL (external) |
| GAP-01-005 | LEGAL-001 legal/privacy review: CRITICAL OPEN_EXTERNAL_GATE | CRITICAL (external) |
| GAP-01-006 | jti-based revocation absent on HS256 candidate path; stolen tokens valid until expiry | MEDIUM |
| GAP-01-007 | ADR-003 (RLS) ACCEPTED_PARTIAL — PostgreSQL RLS foundation present; full multi-tenant defense ongoing | MEDIUM |
| GAP-01-008 | 1058 test count and 87.91% coverage independently NOT TESTED (REPORTED only at qualifying SHA) | MEDIUM (evidence maturity) |
| GAP-01-009 | License NOASSERTION — rights for portfolio source excerpts unconfirmed; links preferred | LOW |
| GAP-01-010 | Render endpoint accessibility at time of Phase 3 inspection NOT TESTED; last confirmed by author at qualification window (2026-08-29) | LOW |

---

## 18. Source Permalinks

All permalinks resolve to the immutable HEAD SHA `46ca4a939bee0396e90abef91905b6d1b95d470c`.

| File | Purpose |
|------|---------|
| [`docs/qualification/status-contract.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/qualification/status-contract.md) | Maturity vocabulary, auth truth, Oya status |
| [`docs/qualification/final-release-manifest.json`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/qualification/final-release-manifest.json) | Release decision, evidence paths, risk semantics |
| [`docs/qualification/real-system-qualification-record.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/qualification/real-system-qualification-record.md) | Live Render test record (sanitized) |
| [`docs/governance/residual-risk-and-release-decision.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/governance/residual-risk-and-release-decision.md) | Final release decision, 5 open gates |
| [`docs/architecture/enterprise-adr-register.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/architecture/enterprise-adr-register.md) | 14 ADRs with status and command tracking |
| [`docs/assurance/threat-model.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/assurance/threat-model.md) | STRIDE threat register, prohibited transitions |
| [`docs/verification.md`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/docs/verification.md) | Phase-by-phase gate records, test results |
| [`AILORA_FREEZE_RECORD.txt`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/AILORA_FREEZE_RECORD.txt) | Freeze timestamp and endpoint URLs |
| [`src/ailora/security/asymmetric_tokens.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/security/asymmetric_tokens.py) | RS256 foundation implementation |
| [`src/ailora/security/contextual_authorization.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/security/contextual_authorization.py) | Fine-grained permission model |
| [`src/ailora/security/supply_chain.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/security/supply_chain.py) | Ed25519 evidence admission |
| [`src/ailora/db/tenant_context.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/db/tenant_context.py) | PostgreSQL tenant context per transaction |
| [`src/ailora/domain/ssa/audit_integrity.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/domain/ssa/audit_integrity.py) | SHA-256 audit chain construction |
| [`src/ailora/services/oya/safety.py`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/src/ailora/services/oya/safety.py) | Oya safety policy, injection rejection |
| [`.github/workflows/ci.yml`](https://github.com/azimilab2025-ai/ailora/blob/46ca4a939bee0396e90abef91905b6d1b95d470c/.github/workflows/ci.yml) | CI pipeline: format, lint, type, test, SBOM, build |

---

## 19. Reproducibility Recipe (R1 → R3 path)

This is a documented path to reach R3 independently. It has NOT been executed (not authorized).

```bash
git clone https://github.com/azimilab2025-ai/ailora.git
cd ailora
git checkout 9571b168080605b72a464d39d1fed5239242330d  # qualifying CI SHA (SUCCESS run 33025990734)
# Requires: Python 3.11, uv
pip install uv
uv sync --locked --all-groups --all-extras
uv run ruff format --check .
uv run ruff check .
uv run mypy src/
uv run pytest tests/ -v --cov=src/ailora --cov-fail-under=85
# Expected at qualifying SHA: 1058 passed (REPORTED), ≥85% coverage (CI floor)
```

Preconditions: Python 3.11; `uv` installed; no database required for most tests (async SQLite in-memory via `aiosqlite`). PostgreSQL-specific tenant RLS tests may require a live database or skip under aiosqlite.

**HEAD reproducibility note:** Reproduction at the freeze HEAD `46ca4a939bee` will produce a **failing CI run** matching OBSERVED run `33390090146`. To reproduce a passing baseline, use the qualifying SHA above.
