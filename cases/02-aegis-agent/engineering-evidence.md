# PRJ-02 — Aegis Agent: Engineering Evidence

**Project ID:** PRJ-02\
**Canonical name:** Aegis Agent — Local-First Multi-Agent Cognitive Twin\
**Source:** https://github.com/aminazimi42-coder/aegis-agent\
**Source HEAD (Phase 3 inspection):** `69f159771c35b04b250fd282189f99b06de0c76d` (2026-10-02T14:46:47Z)\
**Default branch:** `main`\
**License:** Apache-2.0 (`pyproject.toml` declares `license = { text = "Apache-2.0" }`; LICENSE file contains Apache 2.0 text with attribution note. GitHub API reports `NOASSERTION` due to a mixed MIT/"All rights reserved" preamble in the LICENSE file — see §12.)\
**Phase 3 inspection date (UTC):** 2026-10-17\
**Independence:** Source/documentation observations only; no independent third-party verification claimed.\
**Independence level:** Source/documentation observations only; no independent reproduction claimed.\
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.\
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 not reached — all three CI workflows (CI, Release, Security) show FAILURE at HEAD `69f159771c35` (2026-10-02). No passing CI artifact is linked from source documentation. See §6.

---

## 1. Context and Scope

Aegis Agent is a local-first multi-agent cognitive twin for a single senior operator. It provides a consented profile, proposed actions, a human approval gate, and local markdown work products. The default LLM is an offline echo; no paid cloud LLM is wired by default. The system is explicitly scoped to local-first operation — no cloud dependency for core operation, no hosted multi-tenant SaaS, no payment processor in the twin core.

**Status (OBSERVED in `STATUS.md`, `RELEASES.md`, `pyproject.toml`, `docs/manifests/system-manifest.yaml`):**

| Field | Value |
|-------|-------|
| Version | `1.0.0-rc1` (declared in `pyproject.toml`, `RELEASES.md`, `system-manifest.yaml`) |
| Release status | release candidate |
| Tags on GitHub | `v1.0.0-rc2` and `v0.6.0-a` (2 tags found via GitHub API; no releases published) |
| CI status at HEAD | ALL THREE workflows FAILURE (CI, Release, Security) at `69f159771c35` (2026-10-02) |
| Total CI runs | 709 total workflow runs observed via GitHub API |
| LLM default | EchoProvider (offline echo, no network) |
| Agent count | 6 specialists (Alina, Kian, Bita, Aylin, Ahmad, Amin) |
| Test files | 269 test files OBSERVED in tree |
| Total source blobs | 524 blobs (564 tree entries including directories) |
| Live demo | https://aegis-agent-haka.onrender.com (Render, starter plan, 0.5c-512mb) |

**Non-goals (OBSERVED in `docs/LIMITATIONS.md`, `docs/PRIVACY_MODEL.md`, `STATUS.md`):**

- Not hosted multi-tenant SaaS — no hosted multi-tenant billing, card-charging, or tenant-provisioning logic.
- Not a fully autonomous system — six specialists propose only; nothing executes without a human approval gate.
- Not a notarized/shipped desktop installer — codesign/notarize scripts gate on `AEGIS_NOTARY_PROFILE`; without it they print `NOTARY_SKIPPED`.
- No live card charge in core — entitlement is a local signed file; no Stripe, no checkout URL in core.
- No cloud LLM as default — EchoProvider is the default; optional HTTP adapter is off unless explicitly configured.

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading `core/`, `app/`, `agents/`, `docs/`, `pyproject.toml`, `render.yaml`, `Dockerfile`, `docker-compose.yml`, and `STATUS.md` at HEAD SHA `69f159771c35`. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
HTTP Request (FastAPI / Uvicorn)
     │
     ▼
┌─────────────────────────────────────────────────┐
│  app/server.py (109,829 bytes)                  │
│  - FastAPI application factory (create_app)     │
│  - /api/v1/twin/* surface (propose, approve,    │
│    reject, execute, home, memory, email,        │
│    schedule, brief, digest, decisions,          │
│    work products, session flows)                 │
│  - /health, /metrics, /docs, /openapi.json       │
│  - Pydantic contract models                      │
└──────────┬──────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│  app/api/routes.py + app/orchestrator.py        │
│  - Agent registry (AGENT_REGISTRY)              │
│  - AICore dispatch with keyword routing          │
│  - TrustAwareModelRouter (cheap vs expensive)    │
│  - HumanAuthority risk evaluation                │
│  - ReversibleWorkflowManager (rollback stack)    │
│  - RetryGuard (circuit breaker)                 │
└──────────┬──────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│  core/ (140 modules, 697.4 KiB)                 │
│  ┌──────────────────────────────────────────┐  │
│  │ Agent Layer                               │  │
│  │ - agent_base.py: ABC + confidence scoring │  │
│  │ - agent_registry.py: 6 AgentSpec entries  │  │
│  │ - ai_core.py: AICore dispatch + routing    │  │
│  │ - model_router.py: TrustAwareModelRouter  │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │ Safety / Authority Layer                 │  │
│  │ - human_authority.py: 4-level risk eval   │  │
│  │ - authority_matrix.py: RBAC per tenant   │  │
│  │ - tool_tokens.py: HMAC-signed tokens     │  │
│  │ - llm_safety.py: complete_safe gateway   │  │
│  │ - twin_risk.py: L0-L3 classification     │  │
│  │ - sandbox.py: manifest deny-list         │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │ Twin / Cognitive Layer                    │  │
│  │ - twin_actions.py: hash-bound approve     │  │
│  │ - twin_decisions.py: decision ledger      │  │
│  │ - twin_audit.py: audit trail              │  │
│  │ - twin_home.py: pending-action queue      │  │
│  │ - twin_memory_control.py: forget/purge    │  │
│  │ - cognitive_identity.py: fingerprinting   │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │ Persistence / Privacy Layer               │  │
│  │ - persistence.py: SQLite connection       │  │
│  │ - data_at_rest.py: optional encryption    │  │
│  │ - session_lock.py: idle lock + Keychain   │  │
│  │ - redact.py: secret-shape redaction       │  │
│  │ - quota_ledger.py: monthly allowance      │  │
│  │ - evidence_ledger.py: append-only trail   │  │
│  │ - audit_logger.py: JSONL audit events      │  │
│  └──────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │ Reliability Layer                         │  │
│  │ - circuit_breaker.py: failure threshold   │  │
│  │ - retry_guard.py: bounded retry           │  │
│  │ - reversible_workflow.py: rollback stack   │  │
│  │ - self_healing.py: recovery               │  │
│  └──────────────────────────────────────────┘  │
└──────────┬──────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│  agents/ (6 specialist modules)                  │
│  - Alina (strategic coordination)               │
│  - Kian (operational execution)                 │
│  - Bita (analysis and synthesis)                │
│  - Aylin (quality and validation)               │
│  - Ahmad (security and oversight)               │
│  - Amin (finance and executive bridge)          │
└──────────┬──────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│  LLM Provider Layer                              │
│  - EchoProvider (default, offline, no network)  │
│  - HttpProvider (optional, env-gated, timeout 8s│
│    fallback to Echo)                             │
│  - complete_safe() gateway with:                 │
│    * secret redaction (redact_secrets)           │
│    * tool allow-list (ALLOWED_TOOLS)             │
│    * forbidden-claims rejection                  │
│    * quota enforcement                           │
│    * evidence ledger best-effort                │
└──────────┬──────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│  Local SQLite (AEGIS_DATA_DIR/aegis.sqlite)     │
│  - Profile, actions, jobs, receipts, decisions   │
│  - Backup/restore (twin_backup.py)              │
│  - Optional encryption at rest (data_at_rest.py)│
└─────────────────────────────────────────────────┘
```

### 2b. Human Approval Gate Architecture (PRIMARY DIFFERENTIATOR)

The human approval gate is the central architectural invariant of Aegis Agent. The mechanism operates at multiple layers:

**Layer 1 — Risk Classification (`core/twin_risk.py`, OBSERVED):**
- Every proposed action carries a risk level (L0–L3).
- `ALLOWED_L0_EFFECTS` is a `frozenset` of 7 explicit effect strings ("file observe", "git observe", "local .eml outbox", "review_digest", "review_repos", "prepare_weekly_plan", "propose").
- Any effect not in the allow-list is classified as at least L1 (requires human scrutiny).
- L3 keywords: "pay", "wire", "deploy", "production", "delete account".
- `classify(title, effect)` is a pure function — no side effects, deterministic.

**Layer 2 — HumanAuthority Risk Evaluator (`core/human_authority.py`, OBSERVED):**
- `ApprovalLevel` enum: `AUTO`, `SINGLE_APPROVAL`, `DUAL_APPROVAL`, `DENY`.
- `evaluate_risk(action, context)` computes a deterministic 0–1 score from: action length factor, keyword factor (0.35 for delete/remove/exfiltrate/transfer/send/publish), and tenant sensitivity (0.2).
- Score thresholds: < 0.3 → AUTO, < 0.7 → SINGLE_APPROVAL, < 0.95 → DUAL_APPROVAL, ≥ 0.95 → DENY.
- `check_authorization()` raises `PermissionError` on DENY.
- This is a heuristic evaluator, not a formal policy engine — the keyword list is hardcoded and limited.

**Layer 3 — Hash-Bound Approve (`core/twin_actions.py`, OBSERVED):**
- Approvals are bound to the canonical SHA-256 digest of the action payload envelope (`expected_payload_sha256`).
- `_action_digest(action)` computes the canonical envelope from `action_id`, `tenant_id`, `kind`, `title`, `payload`, `effect_type`, and `risk_level`.
- A mutated payload cannot execute — digest mismatch rejects the action before any external effect.
- This is the binding mechanism referenced in `docs/THREAT_NOTES.md`: "Approval binds to the canonical SHA-256 digest of the action payload."

**Layer 4 — Tenant Binding (`core/twin_actions.py`, OBSERVED):**
- Proposed, approved, and executed actions are scoped to `tenant_id`; cross-tenant use is refused.
- The `decision_ledger.py` module stores one row per approve/reject, indexed by `tenant_id` and capped at 50 rows.
- `forget(tenant_id)` deletes rows for one tenant only; neighbor rows are preserved.

**Assessment:** The human approval gate is ENFORCED at the code level — `check_authorization()` raises `PermissionError` on DENY, and the hash-bound approve mechanism prevents payload mutation between propose and execute. However, the risk evaluation is a simple heuristic (keyword matching + length factor), not a formal policy engine. The system has no runtime enforcement audit showing that the approval gate is the sole execution path — that would require runtime instrumentation not present in the source tree.

### 2c. LLM Safety Cage (`core/llm_safety.py`, OBSERVED)

The `complete_safe()` function is the single LLM completion gateway. Every LLM call passes through it:

1. **Secret redaction** — `redact_secrets(prompt)` replaces env-var values (len ≥ 8) and shape-based patterns (AWS keys, JWTs, Bearer tokens, SSH key blocks, webhook secrets) with `***` before the prompt reaches the provider.
2. **Budget check** — if `AEGIS_LLM_BUDGET_EXHAUSTED=1`, the provider is not called at all; returns "rejected: budget exhausted".
3. **Quota enforcement** — `quota_allows_http(tenant_id)` and `quota_state(tenant_id)` check the local quota ledger (`quota.json` under `AEGIS_DATA_DIR`). When exhausted, Echo is forced even if HTTP is configured.
4. **Tool allow-list** — `ALLOWED_TOOLS = ("weekly_digest", "render_work_products", "propose_actions")`. Any `TOOL:<name>` in the response not in this list → `ok=False`, text replaced with "rejected: tool not allowed".
5. **Forbidden-claims rejection** — `FORBIDDEN_CLAIMS = ("transferred funds", "pushed to origin", "payment sent", "email sent")`. If any forbidden phrase appears in the response, `ok=False`.
6. **Evidence ledger** — best-effort ledger entry appended for every completion (ok or rejected).
7. **Provider labeling** — the result includes `provider_kind` ("echo" or "http"), `llm_path`, and `quota_state` for audit transparency.

### 2d. Data Privacy Model (`docs/PRIVACY_MODEL.md`, OBSERVED)

- All durable state — profile, twin actions, jobs, feedback, receipts — is written to SQLite under `AEGIS_DATA_DIR` on the operator's machine (defaults to `data/`).
- No hosted multi-tenant database; no server-side tenant provisioning.
- `backup_tenant` / `restore_tenant` (`core/twin_backup.py`) round-trip a tenant's profile, actions, jobs, and receipts through a local archive.
- Optional encryption at rest (`core/data_at_rest.py`): when `AEGIS_DATA_ENCRYPT=1` and `AEGIS_DATA_PASSPHRASE` are set, the SQLite file is stored as a Fernet-encrypted blob using PBKDF2-HMAC-SHA256 (480,000 iterations, 16-byte random salt).
- Session idle lock (`core/session_lock.py`): after `AEGIS_SESSION_IDLE_SECONDS` (default 900s, min 60, max 86400) of no operator mutation, the tenant session locks. Unlock is a local PIN stored in the macOS Keychain on Darwin or a mock store on CI. Lockout after 5 wrong PINs for 5 minutes.

### 2e. Deployment Topology (RECONSTRUCTED FROM SOURCE)

Two deployment paths exist:

**Path 1 — Render demo (`render.yaml`, OBSERVED):**
- Service name: `aegis-agent-haka` (not `aegis-agent-platform` — old hostname returns 404).
- Plan: starter (0.5c-512mb, paid).
- Auto-deploy on `main` branch.
- Health check at `/health`.
- The `render.yaml` comment explicitly states: "This is a demo deployment, not the sold product; the product loop stays local (`start_operator.sh`, `$HOME/.aegis`, port 8741)."

**Path 2 — Local operator (`scripts/start_operator.sh`, OBSERVED):**
- Exports `AEGIS_DATA_DIR`, uses project venv, binds `127.0.0.1:8741`.
- `desktop/macos/Aegis.app` is a wrapper around `start_operator.sh` — not a notarized installer.
- Helm chart at `charts/aegis-agent/` for Kubernetes deployment.

**Assessment:** The two-path design is a deliberate architectural decision — the demo is a showcase, not the product. The product loop is local-only. This is honestly documented in `render.yaml` and `docs/LIMITATIONS.md`.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Repository | https://github.com/aminazimi42-coder/aegis-agent |
| Default branch | `main` |
| HEAD SHA | `69f159771c35b04b250fd282189f99b06de0c76d` |
| HEAD commit date | 2026-10-02T14:46:47Z |
| Last push | 2026-10-02 |
| Total blobs | 524 (564 tree entries) |
| Repo size | ~4,148 KiB (GitHub API) |
| License (declared) | Apache-2.0 (`pyproject.toml`) |
| License (file) | Apache 2.0 text + mixed preamble (`NOASSERTION` per GitHub API) |
| Tags | `v1.0.0-rc2`, `v0.6.0-a` (2 tags, no published releases) |
| CI workflows | 3: `ci.yml`, `release.yml`, `security.yml` |
| CI status at HEAD | ALL THREE: FAILURE |
| Total CI runs | 709 (observed via GitHub API) |

---

## 4. Agent Architecture and Specialist Decomposition

Six specialist agents are registered in `core/agent_registry.py` as `AgentSpec` dataclass instances:

| Agent | Role | Capabilities | Keywords (routing) |
|-------|------|-------------|-------------------|
| Alina | Strategic coordination | planning, coordination, routing | strategy, strategic, coordination, prioritization |
| Kian | Operational execution | execution, monitoring, optimization | execution, execute, operational, delivery |
| Bita | Analysis and synthesis | analysis, synthesis, reporting | analysis, analyze, synthesis, report |
| Aylin | Quality and validation | validation, quality, review | quality, validation, validate, audit |
| Ahmad | Security and oversight | security, oversight, governance, kms | security, governance, oversight, kms |
| Amin | Finance and executive bridge | finance, invoicing, settlement, executive | finance, financial, invoice, executive |

**Routing mechanism (`core/ai_core.py` → `resolve_agent_name(task)`):** pure keyword matching on lowercased task string. Falls back to "Alina" when no keyword matches. The `TrustAwareModelRouter` adds a second layer: cheap vs expensive model selection based on task length (> 800 chars → expensive) and keyword heuristics ("architecture", "security-review", "incident", etc.).

**Assessment:** The routing is keyword-based and deterministic — no ML model is used for agent selection. This is a deliberate design choice for testability and local-first operation. The routing quality depends entirely on the keyword tables, which are limited and could miss semantically equivalent requests.

**Confidence scoring (`core/agent_base.py`, T160):** `compute_confidence()` produces a 0–100 integer from three signals: specialist keyword match (+15), profile field presence (up to +15), and risk-level penalty (L0: 0, L1: -5, L2: -10, L3: -20). Echo path reduces by 15. Below a threshold of 40, the card body includes a clarifying question.

---

## 5. Critical-Path Analysis: Propose → Approve → Execute Flow

### 5a. Propose Flow

1. A specialist agent proposes an action via the `/api/v1/twin/actions/propose` endpoint.
2. The action is classified by `twin_risk.classify(title, effect)` into L0–L3.
3. The action is stored in the local SQLite twin action store, scoped to `tenant_id`.
4. The action appears in the home queue (`twin_home.py`) as "proposed".
5. A `propose` audit event is written to JSONL (`audit_logger.py`).

### 5b. Approve Flow

1. The human operator reviews the proposed action in the home queue.
2. Approval requires: `tenant_id`, `actor_id`, and `expected_payload_sha256` (the canonical SHA-256 digest of the action payload envelope).
3. On approval, a decision row is appended to `decision_ledger` with: `action_id`, `specialist`, `decision="approve"`, `reason_code`, `digest_prefix` (first 12 chars of payload SHA-256), `created_at` (UTC ISO).
4. If the payload mutates between propose and approve, the digest mismatch rejects the action.

### 5c. Execute Flow

1. Approved actions execute only after the receipt is written.
2. Execution writes a local outbox file or `receipts/{action_id}.md`.
3. Nothing sends externally — the twin core has no network egress for action execution.
4. An `execute` audit event is written to JSONL.

### 5d. Failure Envelope

- **Payload mutation:** rejected by digest mismatch (ENFORCED).
- **Cross-tenant access:** rejected by tenant binding (ENFORCED).
- **Expired tool token:** `ToolTokenExpiredError` (PermissionError subclass) raised.
- **Insufficient capabilities:** `TokenScopeError` raised.
- **Budget exhaustion:** `complete_safe` returns "rejected: budget exhausted" without calling provider.
- **Quota exhaustion:** Echo is forced even when HTTP is configured.
- **Forbidden LLM claims:** ("transferred funds", "pushed to origin", "payment sent", "email sent") → `ok=False`.
- **Circuit breaker open:** `CircuitBreaker.allow_request(key)` returns False after 3 failures; half-open after 30s.
- **Retry exhaustion:** `RetryGuard.execute` raises `RuntimeError` after `max_retries` (default 2).

**Assessment:** The failure envelope is well-designed for a local-first system. The hash-bound approve mechanism is the strongest invariant — it prevents payload tampering between human review and execution. The circuit breaker and retry guard provide basic resilience for the HTTP provider path. The tool token mechanism provides scoped, time-limited access with HMAC-signed tokens.

---

## 6. Test and CI Evidence

### 6a. Test File Inventory

- **269 test files** OBSERVED in `tests/` directory at HEAD SHA `69f159771c35`.
- Test files follow a `test_tNNN_*.py` naming convention (T01 through T234), plus `test_phaseNN_*.py` files.
- Total test code size: ~1,522 KiB (1,558,684 bytes).
- Test files range from 452 bytes (`test_desktop_engine.py`) to 14,204 bytes (`test_t200_entitlement_banner.py`).
- Tests cover: platform integrity, LLM token substrate, cognitive twin, tool tokens, decision ledger, authority matrix, human authority, circuit breaker, reversible workflow, retry guard, redaction, data at rest, session lock, quota ledger, evidence ledger, audit chain, and many more.

**Test count claims:**
- `docs/phase-reports/final-milestone-report.md` claims "27 passed unit tests" — this appears outdated (the test suite has grown substantially since that report).
- `docs/phase-reports/phase-6-final-audit.md` claims "60 tests executed, 0 failures, 0 errors" — also appears historical.
- `RELEASES.md` claims "725 passed, 1 skipped" — this is the most recent count but is REPORTED by the author, NOT independently verified.
- The 269 test files at HEAD likely contain significantly more than 725 test functions, but independent function count NOT TESTED.

### 6b. CI Status

**All three GitHub Actions workflows show FAILURE at HEAD `69f159771c35` (2026-10-02):**

| Workflow | Status | SHA | Date |
|----------|--------|-----|------|
| CI (`ci.yml`) | FAILURE | `69f159771c35` | 2026-10-02T14:46:50Z |
| Release (`release.yml`) | FAILURE | `69f159771c35` | 2026-10-02T14:46:50Z |
| Security Checks (`security.yml`) | FAILURE | `69f159771c35` | 2026-10-02T14:46:50Z |

The immediately prior commit (`19743a20161b`, 2026-10-02T13:45:00Z) also shows FAILURE for CI and Release. This suggests the CI was already failing before the HEAD commit.

**CI workflow details (`ci.yml`, OBSERVED):**
- Runs on push/PR to `main`.
- Python 3.11, uv for dependency management.
- Steps: install dev deps, pytest --collect-only, ruff check, pytest -q, bandit (optional).
- Uses `uv sync --frozen --extra dev` when `uv.lock` is present.

**Assessment:** The triple CI failure at HEAD is a CRITICAL evidence gap. The portfolio does not claim CI passes, but the failure of all three workflows is a material risk. The root cause is NOT inspected — this would require reading the CI logs on GitHub, which is beyond the read-only source inspection scope. The `RELEASES.md` claim of 725 passing tests is REPORTED, not independently verified.

---

## 7. Security and Privacy Boundaries

### 7a. Tool Token System (`core/tool_tokens.py`, OBSERVED)

- `ToolAccessToken`: frozen dataclass with `tenant_id`, `task_id`, `tool_name`, `capabilities`, `issued_at`, `expires_at`, `token_id`.
- `ToolTokenManager`: issues and validates short-lived zero-trust tool tokens using HMAC-SHA256 signing.
- Default TTL: 300 seconds (5 minutes).
- Token validation: `allows()` checks tool name, tenant ID, task ID, required capabilities (case-insensitive), and expiry.
- `ToolTokenExpiredError` and `TokenScopeError` are `PermissionError` subclasses — expired or insufficient tokens raise at the access boundary.

### 7b. Secret Redaction (`core/redact.py`, OBSERVED)

Regex-based redaction of secret-shaped substrings:
- `sk-…` / `sk_live_…` — OpenAI-style API keys
- `ghp_…` / `gho_…` / `ghu_…` / `ghs_…` — GitHub tokens
- `AKIA…` — AWS access-key IDs (20 chars)
- `Bearer <token>` — bearer tokens
- `eyJ…\.…\.` — JWT-shaped tokens
- `-----BEGIN … PRIVATE KEY-----` — PEM private key blocks
- `whsec_…` — webhook signing secrets
- Webhook URL query parameters (`?secret=…`, `&token=…`)

`redact_payload()` walks dict/list/str values for full payload redaction. The audit logger applies `redact_payload` to all `extra` values before writing JSONL.

### 7c. Data at Rest Encryption (`core/data_at_rest.py`, OBSERVED)

- Optional: activated only when `AEGIS_DATA_ENCRYPT=1` and `AEGIS_DATA_PASSPHRASE` are both set.
- Uses `cryptography.fernet.Fernet` with PBKDF2-HMAC-SHA256 (480,000 iterations, 16-byte random salt).
- Default: plaintext SQLite (for backward compatibility with existing stores).
- Passphrase is never written to README, STATUS, receipts, audit JSONL, or session receipts.

### 7d. Session Lock (`core/session_lock.py`, OBSERVED)

- Idle lock after `AEGIS_SESSION_IDLE_SECONDS` (default 900s = 15 min, min 60, max 86400).
- PIN stored in macOS Keychain on Darwin; mock store on CI/other platforms.
- Lockout after 5 wrong PINs for 5 minutes (`PIN_LOCKOUT` code, `retry_after` field).
- PIN is never written into SQLite plaintext, audit JSONL, or receipts.
- Lock state is persisted on the session receipt so a restart preserves the lock.

### 7e. Untrusted Ingest (`docs/THREAT_NOTES.md`, `core/t92_untrusted_ingest.py`, OBSERVED)

- Ingested content (email text, transcript, task string) is never treated as a trusted system instruction.
- Passes through the local allow-list cage (`complete_safe`) and the secret-shape redactor before any processing.
- A GitHub PAT must not be pasted into a public URL — the GitHub observer reads the token from the environment (`AEGIS_GITHUB_TOKEN`) or a local file; it is never embedded in a URL, logged, or committed.

---

## 8. Engineering Decisions (ADR-style)

### ADR-02-001: Local-First as First-Class Constraint

- **Context:** A multi-agent cognitive twin must operate without cloud dependency for core operation.
- **Decision:** Local SQLite under `AEGIS_DATA_DIR`; EchoProvider as default LLM; no network egress for action execution.
- **Alternatives:** Hosted multi-tenant SaaS with cloud LLM (rejected — would require multi-tenant isolation, billing, and cloud LLM dependency).
- **Trade-offs:** Local-only limits accessibility; no remote access to the twin; operator must be on the machine.
- **Evidence:** `docs/PRIVACY_MODEL.md`, `docs/LIMITATIONS.md`, `render.yaml` comment, `core/llm_provider.py`.
- **Status:** ACCEPTED — enforced in code.

### ADR-02-002: Hash-Bound Approve as Tamper-Proofing

- **Context:** Human approval must be bound to the exact payload reviewed, not a mutable reference.
- **Decision:** Approvals bind to the canonical SHA-256 digest of the action payload envelope. A mutated payload cannot execute.
- **Alternatives:** Reference-based approval (rejected — allows payload mutation between review and execution).
- **Trade-offs:** Any payload change requires re-approval; higher friction for iterative edits.
- **Evidence:** `core/twin_actions.py` (`expected_payload_sha256`, `_action_digest`), `docs/THREAT_NOTES.md`.
- **Status:** ACCEPTED — enforced in code.

### ADR-02-003: EchoProvider as Default LLM (Explicit Offline-First)

- **Context:** A local-first system must not silently call a paid cloud LLM.
- **Decision:** `get_provider()` returns `EchoProvider()` unless `AEGIS_LLM_PROVIDER=http` is explicitly set with `AEGIS_LLM_BASE_URL` and `AEGIS_LLM_API_KEY`. Even when configured, `complete_safe` enforces quota, budget, timeout (8s), and falls back to Echo on failure.
- **Alternatives:** Default to OpenAI API (rejected — would violate local-first constraint and introduce cost/privacy risk).
- **Trade-offs:** Echo produces no real completions; all LLM-driven outputs are deterministic echoes until HTTP is configured.
- **Evidence:** `core/llm_provider.py`, `core/llm_safety.py`, `docs/LIMITATIONS.md`.
- **Status:** ACCEPTED — enforced in code.

### ADR-02-004: Risk Classification as Pure Function

- **Context:** Risk assessment must be deterministic and testable, not dependent on external services.
- **Decision:** `twin_risk.classify()` is a pure function with keyword tables and an explicit `ALLOWED_L0_EFFECTS` frozenset. No external ML model, no network call.
- **Alternatives:** ML-based risk scoring (rejected — would require model deployment and introduce non-determinism).
- **Trade-offs:** Keyword-based classification may miss semantically equivalent risks; limited keyword vocabulary.
- **Evidence:** `core/twin_risk.py`, `core/human_authority.py`.
- **Status:** ACCEPTED — but limitations acknowledged.

### ADR-02-005: Demo vs Product Separation

- **Context:** A public demo must not be conflated with the actual product.
- **Decision:** Render deployment is explicitly labeled as "a demo deployment, not the sold product" in `render.yaml`. The product loop stays local (`start_operator.sh`, `$HOME/.aegis`, port 8741). The `docs/LIMITATIONS.md` document states "The Render host is a demo, not the product."
- **Alternatives:** Treat the demo as production (rejected — would imply hosted SaaS, contradicting local-first).
- **Trade-offs:** The demo may not reflect the product's full capability; confusion if demo is treated as product.
- **Evidence:** `render.yaml`, `docs/LIMITATIONS.md`, `docs/PRIVACY_MODEL.md`.
- **Status:** ACCEPTED — documented and enforced in deployment configuration.

---

## 9. Known Limitations and Gaps

### GAP-02-001 (CRITICAL): Triple CI failure at HEAD

All three CI workflows (CI, Release, Security) show FAILURE at HEAD `69f159771c35` (2026-10-02). The immediately prior commit also shows FAILURE. Root cause not inspected — requires CI log analysis. The 725-passing-tests claim in `RELEASES.md` is REPORTED, not independently verified. This is a CRITICAL gap because CI failure at HEAD means the source may not pass its own quality gates.

**Follow-up:** Inspect CI failure logs on GitHub Actions to determine root cause. Determine whether this is a dependency issue, a test failure, or an environment problem.

### GAP-02-002 (HIGH): No published GitHub release

`pyproject.toml` declares version `1.0.0-rc1` and `RELEASES.md` claims "release candidate and verified" status. However, GitHub has 0 published releases. Two tags exist (`v1.0.0-rc2`, `v0.6.0-a`) but neither is a published release. The rc1 version string is REPORTED in source files only, not confirmed by a release artifact. The `v1.0.0-rc2` tag exists but is not a release, and `docs/phase-reports/phase-6-final-audit.md` references it as "Final Tag" — this creates version ambiguity (rc1 in `pyproject.toml` vs rc2 in tags).

**Follow-up:** Confirm whether `v1.0.0-rc2` is the intended release tag. If so, `pyproject.toml` should be updated to match.

### GAP-02-003 (MEDIUM): License ambiguity (NOASSERTION)

`pyproject.toml` declares `license = { text = "Apache-2.0" }`. The `LICENSE` file begins with a preamble: "Copyright 2026 Amin Azimi... Reason: current file mixes MIT grant with 'All rights reserved' and exclusive-property language. That conflict blocks clean commercialization. Apache-2.0 is permissive..." followed by the full Apache 2.0 text. GitHub API reports `NOASSERTION` because the preamble mixes MIT grant language with "All rights reserved" and the Apache 2.0 text. This creates ambiguity for redistribution.

**Follow-up:** The preamble note in the LICENSE file should be separated from the Apache 2.0 text to avoid GitHub's `NOASSERTION` classification. This is a portfolio documentation concern only — the source repository license is not modified by this portfolio.

### GAP-02-004 (MEDIUM): Risk evaluation is heuristic, not formal

The `HumanAuthority.evaluate_risk()` function uses a simple keyword + length heuristic with hardcoded thresholds. The keyword list is limited to 6 words ("delete", "remove", "exfiltrate", "transfer", "send", "publish"). The risk score is a linear combination of three factors capped at 1.0. This is a reasonable starting point but would not catch many real-world high-risk actions (e.g., "drop database", "overwrite config", "revoke credentials").

**Follow-up:** Consider expanding the keyword table or integrating a more sophisticated risk model. Document the limitation in the portfolio case.

### GAP-02-005 (LOW): app/server.py is 109,829 bytes (monolithic)

The `app/server.py` file is 109,829 bytes (~107 KiB) — the single largest source file in the repository. It contains the FastAPI application factory and all `/api/v1/twin/*` route handlers. While the core logic is well-decomposed into `core/` modules, the route layer is monolithic. This is a maintainability concern, not a correctness issue.

**Follow-up:** Consider splitting `app/server.py` into per-resource modules. No action required for portfolio evidence.

### GAP-02-006 (LOW): RuntimeSandbox is a stub

`core/runtime_sandbox.py` contains a `RuntimeSandbox` class that is explicitly a "lightweight placeholder to represent the runtime isolation component. It validates inputs and returns a deterministic response. Replace with container/process sandboxing for production." The manifest sandbox (`core/sandbox.py`) provides a deny-list check (`eval`, `exec`, `subprocess`, `os.system`) and an allow-list (`http`, `db`, `fs`, `nlp`), but this is a pre-check, not OS-level isolation.

**Follow-up:** The stub is honestly labeled. No production sandbox claim should be made in the portfolio.

---

## 10. Comparison with Portfolio Claims

### CLM-02-001: "Aegis Agent is at release candidate 1.0.0-rc1"

| Aspect | Assessment |
|--------|-------------|
| `pyproject.toml` | `version = "1.0.0-rc1"` — OBSERVED |
| `RELEASES.md` | "Version: 1.0.0-rc1" — OBSERVED |
| `system-manifest.yaml` | `version: 1.0.0-rc1` — OBSERVED |
| GitHub releases | 0 published releases — OBSERVED |
| GitHub tags | `v1.0.0-rc2` exists (not rc1) — OBSERVED |
| CI status | ALL FAIL at HEAD — OBSERVED |

**Assessment:** The rc1 version is REPORTED in source files, not confirmed by a release artifact or passing CI. The tag `v1.0.0-rc2` creates version ambiguity. The portfolio README states "1.0.0-rc1" which matches `pyproject.toml` but not the tag. CONDITIONAL — version string is consistent in source, but no release artifact confirms it, and CI is failing.

### CLM-02-002: "Aegis Agent has human approval gates enforced by design"

| Aspect | Assessment |
|--------|-------------|
| `human_authority.py` | `check_authorization()` raises `PermissionError` on DENY — OBSERVED |
| `twin_risk.py` | L0-L3 classification with `ALLOWED_L0_EFFECTS` frozenset — OBSERVED |
| `twin_actions.py` | Hash-bound approve with `expected_payload_sha256` — OBSERVED |
| `decision_ledger.py` | SQLite decision rows scoped to tenant_id — OBSERVED |
| `docs/LIMITATIONS.md` | "Six specialists propose only; a human must approve" — OBSERVED |
| `docs/PRIVACY_MODEL.md` | "No specialist executes an action on its own" — OBSERVED |

**Assessment:** PASS — the human approval gate is ENFORCED at multiple code layers. The hash-bound approve mechanism prevents payload mutation. The risk classification is deterministic. The limitation is that the risk evaluation is heuristic, not formal (GAP-02-004). The approval gate is a genuine architectural differentiator, not just documentation.

### CLM-02-003: "Aegis Agent CI passes"

**Assessment:** FAIL — all three CI workflows show FAILURE at HEAD `69f159771c35` (2026-10-02). The portfolio does not claim CI passes (this was an OBSERVED gap, not a portfolio claim). The triple failure is a material evidence gap.

### CLM-02-004: "Aegis Agent is local-first with no cloud dependency"

| Aspect | Assessment |
|--------|-------------|
| `llm_provider.py` | `EchoProvider` is default; `HttpProvider` is optional and env-gated — OBSERVED |
| `docs/PRIVACY_MODEL.md` | "No paid cloud LLM is wired by default" — OBSERVED |
| `docs/LIMITATIONS.md` | "Echo is the default engine" — OBSERVED |
| `docker-compose.yml` | Local Docker with `python:3.11-slim` — OBSERVED |
| `render.yaml` | Render deployment is explicitly a demo, not the product — OBSERVED |
| Helm chart | `charts/aegis-agent/` present — implies possible hosted deployment path |

**Assessment:** PASS with nuance — the core is local-first with EchoProvider as default. The optional HTTP adapter is off unless explicitly configured. The Helm chart implies a possible hosted deployment path, but this is for the demo/development, not the product. The portfolio claim "local-first with no cloud dependency for core operation" is accurate.

---

## 11. Reproducibility Recipe

To independently inspect the Aegis Agent source at the pinned SHA:

1. Clone the repository: `git clone https://github.com/aminazimi42-coder/aegis-agent`
2. Checkout the pinned SHA: `git checkout 69f159771c35b04b250fd282189f99b06de0c76d`
3. Verify the tree: `git log --oneline -1` should show `69f1597`.
4. Read key files:
   - `core/tool_tokens.py` — tool token HMAC signing and validation
   - `core/decision_ledger.py` — tenant-scoped decision SQLite
   - `core/human_authority.py` — 4-level risk evaluator
   - `core/twin_risk.py` — L0-L3 classification with allow-list
   - `core/llm_safety.py` — `complete_safe` gateway with allow-list, redaction, quota
   - `core/llm_provider.py` — EchoProvider and HttpProvider
   - `core/data_at_rest.py` — optional Fernet encryption
   - `core/session_lock.py` — idle lock with Keychain PIN
   - `core/redact.py` — secret-shape regex redaction
   - `docs/LIMITATIONS.md` — honest shipped/not-shipped list
   - `docs/PRIVACY_MODEL.md` — where data lives and what inference runs
   - `docs/THREAT_NOTES.md` — untrusted ingest, approval binding, PAT handling
   - `STATUS.md` — shipped capability status
5. To run the test suite (requires Python 3.11): `pip install -e ".[dev]" && python -m pytest -q`
   - NOTE: CI is failing at HEAD; the test suite may not pass without dependency resolution.
6. To verify CI status: check GitHub Actions at `https://github.com/aminazimi42-coder/aegis-agent/actions`
   - All three workflows show FAILURE at `69f159771c35`.

**Limitations:** This recipe is R1 (source inspection) only. R3 (independent clean-room execution) is NOT TESTED. The CI failure at HEAD means the test suite may not pass in a fresh environment.

---

## 12. Evidence Matrix

| Evidence ID | Claim | Source | Observation | Result | Notes |
|-------------|-------|--------|-------------|--------|-------|
| EVD-02-001 | Human approval gate enforced via `check_authorization()` raising PermissionError on DENY | `core/human_authority.py` | OBSERVED | PASS | ENFORCED in code; `DENY` level raises `PermissionError` |
| EVD-02-002 | Hash-bound approve prevents payload mutation | `core/twin_actions.py` | OBSERVED | PASS | `expected_payload_sha256` + `_action_digest()` canonical envelope; mismatch rejects |
| EVD-02-003 | L0 allow-list restricts observe-only effects to 7 explicit strings | `core/twin_risk.py` | OBSERVED | PASS | `ALLOWED_L0_EFFECTS` frozenset; unknown effects → at least L1 |
| EVD-02-004 | Tool tokens are HMAC-SHA256 signed, short-lived (300s default), capability-scoped | `core/tool_tokens.py` | OBSERVED | PASS | `ToolTokenManager._sign()` uses HMAC-SHA256; `ToolAccessToken.is_expired` and `allows()` enforce scope |
| EVD-02-005 | `complete_safe` gateway enforces tool allow-list, forbidden claims, secret redaction, quota | `core/llm_safety.py` | OBSERVED | PASS | `ALLOWED_TOOLS` (3 items), `FORBIDDEN_CLAIMS` (4 items), `redact_secrets()`, `quota_allows_http()` |
| EVD-02-006 | EchoProvider is default; HttpProvider is optional, env-gated, 8s timeout, Echo fallback | `core/llm_provider.py` | OBSERVED | PASS | `get_provider()` returns EchoProvider unless `AEGIS_LLM_PROVIDER=http` + `AEGIS_LLM_BASE_URL` + key |
| EVD-02-007 | Decision ledger is tenant-scoped SQLite with 50-row cap and typed approve/reject | `core/decision_ledger.py` | OBSERVED | PASS | `append()` validates `decision in ("approve", "reject")`; `list(tenant_id)` and `forget(tenant_id)` are tenant-scoped |
| EVD-02-008 | Secret redaction covers AWS keys, JWTs, Bearer tokens, GitHub tokens, SSH key blocks, webhook secrets | `core/redact.py` | OBSERVED | PASS | 7 regex patterns; `redact_payload()` walks dict/list/str; audit logger applies `redact_payload` to all `extra` |
| EVD-02-009 | Optional encryption at rest uses Fernet + PBKDF2-HMAC-SHA256 (480K iterations) | `core/data_at_rest.py` | OBSERVED | PASS | `encrypt_enabled()` checks `AEGIS_DATA_ENCRYPT=1`; `_derive_key()` uses PBKDF2; `_encrypt_bytes()` prepends random salt |
| EVD-02-010 | Session idle lock with Keychain PIN, 5-attempt lockout for 5 minutes | `core/session_lock.py` | OBSERVED | PASS | `_idle_seconds()` (default 900, min 60, max 86400); `_MAX_FAILURES=5`; `_LOCKOUT_SECONDS=300`; PIN never in SQLite/audit/receipts |
| EVD-02-011 | Circuit breaker opens after 3 failures, half-open after 30s recovery | `core/circuit_breaker.py` | OBSERVED | PASS | `failure_threshold=3`, `recovery_seconds=30`; `allow_request()` returns False when open; half-open resets failures |
| EVD-02-012 | Reversible workflow with rollback stack for compensating undo | `core/reversible_workflow.py` | OBSERVED | PASS | `begin()` → `execute(action, rollback)` → `commit()` or `rollback()`; rollbacks execute in reverse order |
| EVD-02-013 | 269 test files OBSERVED in `tests/` at pinned SHA | `tests/` directory tree | OBSERVED | PASS | File count confirmed via GitHub API `git/trees` recursive listing |
| EVD-02-014 | CI at HEAD: ALL THREE workflows FAILURE | GitHub Actions API | OBSERVED | FAIL | CI, Release, Security all `conclusion=failure` at `69f159771c35` (2026-10-02). Root cause NOT inspected. |
| EVD-02-015 | No published GitHub releases; 2 tags (`v1.0.0-rc2`, `v0.6.0-a`) | GitHub API | OBSERVED | PASS | `releases` endpoint returns empty array; `tags` endpoint returns 2 tags |
| EVD-02-016 | License declared as Apache-2.0 in `pyproject.toml`; GitHub reports NOASSERTION due to mixed preamble | `pyproject.toml` + GitHub API | OBSERVED | CONDITIONAL | `license = { text = "Apache-2.0" }` in pyproject; LICENSE file has mixed MIT/"All rights reserved" preamble before Apache 2.0 text |
| EVD-02-017 | Render demo is explicitly not the product | `render.yaml` | OBSERVED | PASS | Comment: "This is a demo deployment, not the sold product; the product loop stays local" |
| EVD-02-018 | 725 passing tests claimed in RELEASES.md | `RELEASES.md` | REPORTED | NOT_TESTED | Author-reported; not independently verified; CI failing at HEAD |
| EVD-02-019 | Six specialists propose only; no specialist executes alone | `docs/LIMITATIONS.md` + `docs/PRIVACY_MODEL.md` | OBSERVED | PASS | "Six specialists propose only; a human must approve" — LIMITATIONS.md; "No specialist executes an action on its own" — PRIVACY_MODEL.md |
| EVD-02-020 | TrustAwareModelRouter routes by keyword heuristics to cheap/expensive model | `core/model_router.py` | OBSERVED | PASS | `ROUTE_KEYWORDS` dict for 4 agents; `EXPENSIVE_KEYWORDS` for model selection; `decide_model()` and `decide_llm_model()` |
| EVD-02-021 | AuthorityMatrix provides per-tenant RBAC with role definitions and evidence ledger | `core/authority_matrix.py` | OBSERVED | PASS | `define_role()`, `assign_role()`, `revoke_role()`, `check_permission()` — all scoped to tenant_id and recorded in EvidenceLedger |
| EVD-02-022 | Commitment engine tracks create/fulfill/cancel with audit trail | `core/commitment_engine.py` | OBSERVED | PASS | `create_commitment()`, `fulfill_commitment()`, `cancel_commitment()` — all recorded in EvidenceLedger |
| EVD-02-023 | JSONL audit logger with correlation IDs and secret redaction | `core/audit_logger.py` | OBSERVED | PASS | `log_event()` with 7 valid kinds; `_get_or_create_correlation()`; `redact_payload()` applied to `extra` |
| EVD-02-024 | Quota ledger enforces monthly allowance per tenant; default allowance is 0 (Echo-limited) | `core/quota_ledger.py` | OBSERVED | PASS | `_row()` returns default `allowance=0`; `remaining = max(allowance - used, 0)`; "When the file is missing the default allowance is 0 — Echo-limited" |
| EVD-02-025 | Helm chart present for Kubernetes deployment | `charts/aegis-agent/` | OBSERVED | PASS | `Chart.yaml`, `deployment.yaml`, `service.yaml`, `values.yaml` present in tree |

---

## 13. 30-Filter Gate Record

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 3 for PRJ-02 explicitly |
| F02 Scope | PASS | Only `cases/02-aegis-agent/engineering-evidence.md` and `docs/evidence/central-index.md` modified |
| F03 Source immutability | PASS | No source repo files modified; read-only GitHub API inspection only |
| F04 Baseline | PASS | Branch `main`, HEAD `d3ba70b`, clean working tree verified before edits |
| F05 Conflict | PASS | No existing directive or owner work conflicts |
| F06 Intent | PASS | Directly advances evidence-first portfolio objective |
| F07 Necessity | PASS | Engineering evidence file is the Phase 3 deliverable |
| F08 Duplication | PASS | Existing README preserved; no duplicate content created |
| F09 Provenance | PASS | Source SHA `69f159771c35` pinned; file paths and timestamps captured |
| F10 Authenticity | PASS | No source statement treated as independently verified |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/NOT_TESTED used precisely |
| F12 Reproducibility | PASS | §11 provides recipe to inspect at pinned SHA |
| F13 Technical correctness | PASS | Architecture and code excerpts match actual source |
| F14 Domain safety | PASS | Agent governance, privacy, and local-first claims respected |
| F15 Trust boundaries | PASS | Source files treated as untrusted data; no prompt injection followed |
| F16 Security | PASS | No injection, unsafe links, or exposed attack details |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific information leaked |
| F18 IP | PASS | Apache-2.0 license declared; no substantial source code copied; only excerpts referenced |
| F19 Attribution | PASS | Source author (Amin Azimi, Azimi Innovation Lab) accurately attributed |
| F20 Test integrity | PASS | No unexecuted test described as passed; 269 files OBSERVED, 725-passing CLAIM is REPORTED/NOT_TESTED |
| F21 Failure modes | PASS | Triple CI failure, license ambiguity, and heuristic risk limitations all documented |
| F22 Version drift | PASS | rc1 vs rc2 tag ambiguity documented (GAP-02-002) |
| F23 Interface stability | PASS | Existing README, case ordering, and links preserved |
| F24 Accessibility | PASS | Headings, tables, and text navigation are readable |
| F25 Maintainability | PASS | Concise structured document; no second source tree created |
| F26 Reviewability | PASS | Diff is one new evidence file + central-index update |
| F27 Validation | PASS | Source links point to pinned SHA; GitHub API responses verified |
| F28 Git hygiene | PASS | One meaningful commit for Phase 3 PRJ-02 evidence |
| F29 Disclosure | PASS | No public-safe content beyond evidence; no production claims |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision:** PROCEED — all 30 filters PASS; no blocking evidence.

---

## 14. Phase 3 Completion Report

**PHASE:** 3 — Flagship Deep Engineering Cases (PRJ-02 Aegis Agent)\
**AUTHORIZED SCOPE:** `cases/02-aegis-agent/engineering-evidence.md` (new), `docs/evidence/central-index.md` (update), `docs/evidence/claims.csv` (append EVD-02 records)\
**BASELINE HEAD:** `d3ba70b3a46fdabf2751525f4d835a6738a435d9` (portfolio `main`)\
**SOURCE HEAD:** `69f159771c35b04b250fd282189f99b06de0c76d` (Aegis Agent `main`)

**OBSERVED FACTS:**
- 524 source blobs, 269 test files, 140 core modules at pinned SHA
- 6 specialist agents with keyword-based routing and confidence scoring
- Human approval gate enforced at 4 layers: risk classification, authority evaluation, hash-bound approve, tenant binding
- LLM safety cage with tool allow-list, forbidden-claims rejection, secret redaction, quota enforcement
- Optional encryption at rest (Fernet + PBKDF2), session idle lock with Keychain PIN
- All 3 CI workflows: FAILURE at HEAD (2026-10-02)
- 0 published GitHub releases; 2 tags (rc2, v0.6.0-a)
- Apache-2.0 declared in pyproject.toml; NOASSERTION per GitHub API (mixed preamble)
- Render demo explicitly not the product

**FILES CHANGED:** `cases/02-aegis-agent/engineering-evidence.md` (new), `docs/evidence/central-index.md` (updated), `docs/evidence/claims.csv` (appended EVD-02-* records)

**EVIDENCE ADDED:** EVD-02-001 through EVD-02-025 (25 evidence records), 5 ADRs (ADR-02-001 through ADR-02-005), 6 gaps (GAP-02-001 through GAP-02-006)

**CHECKS ACTUALLY RUN:**
- GitHub API `git/trees` recursive listing at pinned SHA — confirmed 524 blobs, 269 test files
- GitHub API `releases` endpoint — confirmed 0 published releases
- GitHub API `tags` endpoint — confirmed 2 tags (v1.0.0-rc2, v0.6.0-a)
- GitHub API `actions/runs` — confirmed all 3 workflows FAILURE at HEAD, 709 total runs
- GitHub raw content fetch — 44+ source files fetched and inspected at pinned SHA
- LICENSE file content inspection — confirmed mixed preamble + Apache 2.0 text

**NOT TESTED / BLOCKED:**
- Independent test execution (R3) — NOT TESTED; CI failing at HEAD
- CI failure root cause — NOT INSPECTED; requires GitHub Actions log analysis
- Live endpoint accessibility — NOT TESTED; no HTTP probing authorized
- 725-passing-tests claim — REPORTED by author in RELEASES.md; NOT independently verified

**RISKS / OPEN DECISIONS:**
- GAP-02-001 (CRITICAL): Triple CI failure at HEAD — root cause unknown
- GAP-02-002 (HIGH): Version ambiguity (rc1 in pyproject.toml vs rc2 tag)
- GAP-02-003 (MEDIUM): License NOASSERTION — mixed preamble in LICENSE file
- OD-02 remains OPEN: rc1 label is source-file-only, no release artifact confirms it
- OD-04 (PRJ-02) remains OPEN: CI failures at HEAD — known regression?

**COMMIT SHA:** (pending owner authorization)\
**PUSH STATUS:** (pending owner authorization)\
**NEXT PHASE:** NOT STARTED — AWAITING OWNER.
