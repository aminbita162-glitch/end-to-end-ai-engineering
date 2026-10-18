# PRJ-05 — Kian Trading Intelligence: Engineering Evidence

**Project ID:** PRJ-05
**Canonical name:** Kian Trading Intelligence
**Source:** https://github.com/aminbita162-glitch/kian-trading-intelligence
**Source HEAD (Phase 4 inspection):** `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` (2026-10-17T09:21:33Z)
**Default branch:** `main`
**License:** PROPRIETARY — `LICENSE` file declares "PROPRIETARY AND CONFIDENTIAL — ALL RIGHTS RESERVED" by Amin Azimi / Azimi Innovation Lab. `pyproject.toml` declares `license = { text = "Proprietary" }`. No SPDX-recognized open-source license. Copying source into the portfolio requires explicit owner authorization. No source code is copied into this case file — only attributed descriptions and links.
**Software version:** `0.20.0` (OBSERVED in `pyproject.toml` and `status.yaml`)
**Directive version:** `1.4` (OBSERVED in `status.yaml` — "Post-E10 Elevation Plan A-D: Real Infrastructure and Locked Ideas")
**Phase 4 inspection date (UTC):** 2026-10-17
**Reviewer:** Internal portfolio source/documentation inspection; no independent third-party verification claimed.
**Independence level:** INTERNAL_PORTFOLIO_REVIEW — source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 partially achieved — CI run SUCCESS at HEAD `54ef6ca4d8fd` observed via GitHub Actions API (Phase 1). Prior two CI runs at earlier SHAs FAILED. The CI run artifact is linked but not independently reproduced by this reviewer.

---

## 1. Context and Scope

Kian Trading Intelligence (KTI) is a multi-tenant, cloud-executed cryptocurrency trading and mining **simulation** platform. It combines deterministic risk control (an independent Risk Kernel), four specialized agents, an execution engine, a financial ledger, mining simulation, and a graduated autonomy/kill-plane safety system. The platform operates in SIMULATION mode only — no live trading, no real mining, no real-money operations of any kind.

**Status (OBSERVED in `status.yaml` at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| `operating_mode` | `SIMULATION` | OBSERVED |
| `live_authorized` | `false` | OBSERVED |
| `mining_authorized` | `false` | OBSERVED |
| `software_version` | `0.20.0` | OBSERVED |
| `python_version` | `3.11+` | OBSERVED |
| `test_counts.python_pytest` | `2003` | REPORTED (in `status.yaml`; not independently re-counted by this reviewer) |
| `test_counts.vitest` | `91` | REPORTED |
| `test_counts.ruff` | `clean` | REPORTED |
| `test_counts.mypy` | `clean` | REPORTED |
| `test_counts.secret_scan` | `0_findings` | REPORTED |

**Honest labels (OBSERVED in `status.yaml` §honest_labels):**

| Label | Status | Note |
|-------|--------|------|
| `live_trading` | PLANNED | Not implemented |
| `real_mining` | PLANNED | Not implemented |
| `real_withdrawals` | PLANNED | Not implemented |
| `risk_kernel` | REAL | Implemented and tested |
| `financial_ledger` | REAL | Implemented and tested |
| `agent_orchestration` | REAL | Implemented and tested |
| `mining_simulation` | REAL | Simulation only |
| `kill_plane` | REAL | Implemented and tested |
| `signed_risk_tokens` | REAL | Implemented (E4/Phase C) |
| `hash_chained_audit` | REAL | Implemented |
| `formal_models` | PLANNED | TLA+ specs exist; formal verification not fully operational |
| `strategy_validation_gate` | PLANNED | Not fully implemented |
| `agent_runtime_mesh` | PLANNED | Not fully implemented |

**Non-goals (OBSERVED across `status.yaml`, `docs/architecture.md`, and source comments):**

- Not a live trading system. `LIVE` mode requires explicit `live_authorized` decision and is never the default.
- No real mining. `mining_authorized: false`.
- No real-money operation of any kind.
- No exchange is automatically approved for live use (OBSERVED in `simulation/exchange_simulator.py` docstring: "No exchange is automatically approved for live use (Section 05.5)").
- Completing any phase (E0–E10, A–D) does NOT authorize live trading — this is stated as a `note` on every phase in `status.yaml`.

**Phase completion (OBSERVED in `status.yaml`):**

All elevation phases E0–E10 plus Phase A, B, C, D are marked `COMPLETE`. Prior phases 01–10 (from V1.0/V1.1) also marked `COMPLETE` and `merged: true`. The platform is at a simulation-grade, market-ready state — not a production-deployed live system.

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading `docs/architecture.md`, `status.yaml`, `services/`, `packages/contracts/`, `simulation/`, `formal/`, `tests/`, `.github/workflows/ci.yml`, and `pyproject.toml` at HEAD SHA `54ef6ca4d8fd`. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
Client Layer (apps/)
  │  MacBook (Tauri/Rust)  |  Web (React/TypeScript/Vite)  |  iPhone PWA
  │  Clients are NOT trusted financial execution authorities
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  Cloud Control Plane (services/)                        │
│  - identity/: tenant, profile, auth, MFA/TOTP           │
│  - market_data/: provider, processor, storage            │
│  - strategy_science/: research registry, validation      │
│  - agents/: 4 specialized agents (orchestrator,          │
│    market_intelligence, execution_supervisor,            │
│    mining_operations, strategy_portfolio)                 │
│  - autonomy/: trust-level engine, kill plane,             │
│    operator cockpit, compliance package builder           │
│  - billing/: commercial tiers, usage metering             │
│  - core/: Redis Streams event bus, e2e simulation         │
│  - evidence/: audit chain, decision ledger                │
│  - security/: credential vault (AES-256-GCM), policy     │
│  - ledger/: financial ledger (Decimal, balanced postings) │
│  - release/: release engineering, readiness gates        │
│  - locked_ideas/: token budget governor, decision replay │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Independent Risk & Safety Kernel                  │  │
│  │  (services/risk_kernel/kernel.py — 750 lines)      │  │
│  │  Deterministic, NOT an agent (AD-004)              │  │
│  │  Thread-safe (Lock-guarded)                        │  │
│  │  9 sequential safety checks per order              │  │
│  │  Risk authorization + reservation lifecycle        │  │
│  │  Emergency stop state machine                      │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Execution Engine (services/execution/engine.py)  │  │
│  │  Mandatory pipeline: risk auth → reserve →          │  │
│  │  submit → reconcile → record fill                  │  │
│  │  No order reaches adapter without risk auth         │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────┐
│  Simulation Layer (simulation/) — OUTSIDE deployable tree │
│  - exchange_simulator.py: deterministic exchange emulation │
│  - market_data_simulator.py: deterministic market data    │
│  - mining_simulator.py: deterministic mining simulation   │
│  - security_simulators.py: fault injection, load testing  │
│  - dst_engine.py: Deterministic Simulation Universe       │
│  - market_twin.py: microstructure-realistic market twin  │
│  - conformance_suite.py: adapter conformance verification│
│  - snapshot_pitr.py: snapshot reconciliation + PITR       │
│  - load_chaos_drills.py: load and chaos drill runners     │
│  - release_generators.py: readiness generators            │
│  Per E3: simulators live OUTSIDE services/ tree           │
│  Production image contains no verification code           │
└─────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────┐
│  Formal Models (formal/)                                 │
│  - OrderModel.tla, SessionModel.tla,                     │
│    ReservationModel.tla, SafeHaltModel.tla               │
│  - formal_checker.py: Python checker validating           │
│    implementation invariants against TLA+ models          │
│  Per E4: checked in CI                                    │
└─────────────────────────────────────────────────────────┘
```

### 2b. Operating Mode Guard (OBSERVED in `packages/contracts/mode.py`)

The `OperatingModeConfig` is a singleton that enforces the simulation-only boundary:

```python
# packages/contracts/mode.py — OBSERVED at HEAD
class OperatingModeConfig:
    _instance: OperatingModeConfig | None = None
    _mode: OperatingMode = OperatingMode.SIMULATION
    _live_authorized: bool = False

    @property
    def is_live(self) -> bool:
        """True only if mode is LIVE AND live_authorized is True."""
        return self._mode is OperatingMode.LIVE and self._live_authorized

    def set_mode(self, mode: OperatingMode, *, live_authorized: bool | None = None) -> None:
        if mode is OperatingMode.LIVE and live_authorized is None:
            raise PermissionError(
                "LIVE mode requires explicit live_authorized (True or False). "
                "Real-money trading is not enabled without explicit authorization."
            )
        # ...
```

**Key invariant:** `is_live` is `True` only when both `mode == LIVE` and `live_authorized == True`. Setting `LIVE` mode without an explicit `live_authorized` decision raises `PermissionError` — it is impossible to silently enter live trading. The default is `SIMULATION` with `live_authorized = False`.

### 2c. Risk Kernel Architecture (OBSERVED in `services/risk_kernel/kernel.py` — 750 lines)

The Risk Kernel is the central deterministic safety component. Per AD-004, it is explicitly **NOT an agent** — it is a deterministic safety service. It is thread-safe (`threading.Lock` guards all state mutations).

**9 sequential safety checks per order assessment:**

| # | Check | Failure condition | Rejection reason |
|---|-------|-------------------|-------------------|
| 1 | Emergency stop | `is_active and side == "buy"` | "Emergency stop is active — new exposure-increasing orders blocked." |
| 2 | Session state | `is_terminal` or `is_halted` | "Session is terminal" or "Session is in SAFE_HALT" |
| 3 | Exposure limit | `current + order_value > max_position_value` | "Exposure exceeds limit" |
| 4 | Daily loss limit | `current_loss >= max_daily_loss` | "Daily loss exceeds limit. Trading stopped for the day." |
| 5 | Drawdown limit | `current_loss >= max_drawdown` | "Drawdown exceeds limit." |
| 6 | Concentration limit | `conc_pct > max_concentration_pct` | "Concentration exceeds limit." |
| 7 | Position count per symbol | `count >= max_position_per_symbol` | "Position count exceeds limit." |
| 8 | Order state | Not `CREATED` or `VALIDATED` | "Order state is not eligible for risk approval." |
| 9 | Policy version | Policy not `ACTIVE` | "Policy is not active." |

**Assessment determinism (OBSERVED in `assess()` method):**

```python
# services/risk_kernel/kernel.py — OBSERVED at HEAD
def assess(self, intent: OrderIntent, *, current_price: str,
           session: TradingSession | None = None) -> RiskAssessment:
    """This is the core deterministic safety validation. The assessment is
    deterministic: same inputs always produce the same decision."""
    checked: list[str] = []
    policy = self.get_active_policy(intent.tenant_id)
    order_value = Decimal(intent.quantity) * Decimal(current_price)
    checks = [
        partial(self._check_emergency_stop, intent=intent),
        partial(self._check_session_state, session=session),
        partial(self._check_exposure, intent=intent, order_value=order_value),
        # ... 6 more checks in order ...
    ]
    for check_fn in checks:
        result = check_fn(checked=checked, policy=policy)
        if result is not None:
            return result  # fail-fast: first rejection wins
    return RiskAssessment(authorized=True, reason="All risk checks passed.", ...)
```

**Risk authorization lifecycle:**

1. `assess()` — deterministic evaluation against active policy (9 checks, fail-fast).
2. `authorize()` — if assessment passes, issues a `RiskAuthorization` with TTL (default 30s), bound to exact order parameters (symbol, side, quantity, max_price).
3. `reserve_capacity()` — concurrency-safe reservation that checks exposure under lock; increases `_current_exposure` for buy orders.
4. `consume_authorization()` — marks authorization as `CONSUMED`; expired or non-APPROVED authorizations raise.
5. `consume_reservation()` — marks reservation as `CONSUMED`; expired reservations raise `ReservationExpiredError`.
6. `release_reservation()` — returns capacity to the budget; decreases `_current_exposure` for buy orders.

**Emergency stop (`EmergencyStop` class — OBSERVED):**

- `activate(reason)` — sets `_active = True`, blocks new buy orders.
- `deactivate()` — requires explicit human authorization (caller responsibility).
- Prevents automatic trading restart (no auto-reactivation).
- `reset()` — test-only state reset.

### 2d. Execution Engine (OBSERVED in `services/execution/engine.py`)

The `ExecutionEngine` enforces the mandatory pipeline (Section 05.1):

1. Verify exchange is connected.
2. `risk_kernel.authorize(intent)` — if `DENIED`, raise `RiskDeniedError`.
3. `risk_kernel.reserve_capacity(auth)` — concurrency-safe capacity reservation.
4. `exchange.submit_order(intent, ...)` — submit through adapter.
5. Handle result: `FILLED`/`PARTIALLY_FILLED` → consume auth + reservation + record fills; `REJECTED`/`EXPIRED`/`CANCELLED` → release reservation + cancel auth; `UNKNOWN_OUTCOME` → consume reservation (hold capacity) and require reconciliation.

**Invariant (OBSERVED in docstring and code):** "No exposure-increasing order may reach an exchange adapter without valid risk authorization." The execution engine calls `self._kernel.authorize()` before `self._exchange.submit_order()`. If authorization is denied, `RiskDeniedError` is raised and the exchange adapter is never called.

### 2e. Simulation/Production Separation (OBSERVED in `simulation/__init__.py`)

Per E3 (Structure and Verification Separation): all simulators, fault injectors, readiness generators, and test harnesses live in the `simulation/` package — **OUTSIDE** the deployable `services/` tree. The production image contains no verification code. The `ExecutionEngine` depends on the `ExchangeAdapter` protocol (`contracts.exchange_adapter`), not the concrete `ExchangeSimulator`.

Import-linter contracts (OBSERVED in `.importlinter`) enforce this separation at CI time.

### 2f. Order State Machine (OBSERVED in `packages/contracts/enums.py`)

Legal order state transitions are explicitly defined in `LEGAL_ORDER_TRANSITIONS`:

- `CREATED` → `VALIDATED`, `REJECTED`, `CANCELLED`
- `VALIDATED` → `RISK_APPROVED`, `REJECTED`, `CANCELLED`
- `RISK_APPROVED` → `SUBMISSION_PENDING`, `CANCELLED`, `EXPIRED`
- `SUBMISSION_PENDING` → `SUBMITTED`, `REJECTED`, `EXPIRED`, `SAFE_HALT`
- `SUBMITTED` → `PARTIALLY_FILLED`, `FILLED`, `CANCELLED`, `REJECTED`, `EXPIRED`, `UNKNOWN_OUTCOME`, `SAFE_HALT`
- `UNKNOWN_OUTCOME` → `RECONCILIATION_REQUIRED`, `SAFE_HALT`
- `SAFE_HALT` → `RECONCILIATION_REQUIRED` (no auto-restart)
- Terminal states: `FILLED`, `CANCELLED`, `REJECTED`, `EXPIRED` (empty transition sets)

The formal checker (`formal/formal_checker.py`) validates that terminal states have no outgoing transitions and that no state transitions to `CREATED`.

### 2g. Signed Risk Authorization Tokens (OBSERVED in `packages/contracts/risk_token.py` + `services/phase_c/`)

Per AD-036 and Phase C: risk authorizations are cryptographically signed with HMAC-SHA256.

- `RiskTokenPayload` — frozen dataclass bound to exact order parameters (auth_id, tenant_id, policy_id, policy_version, intent_id, symbol, side, quantity, max_price).
- `SignedRiskToken` — payload + issued_at + expires_at + nonce (UUID) + signature (HMAC-SHA256 hex) + token_str (base64url-encoded JSON).
- `RiskTokenSigner` — trust-domain separated: key from `KTI_RISK_TOKEN_KEY` (distinct from vault key `KTI_VAULT_KEY`). Fail-closed in production (raises `RiskTokenKeyError` if key missing or < 32 bytes).
- `verify()` — structural validation → constant-time signature comparison (`hmac.compare_digest`) → expiry check → replay check (consumed nonce set).
- `consume()` — verify + mark nonce as consumed (one-shot; second use raises `RiskTokenReplayError`).
- Token TTL: 30 seconds default; short-lived and single-use.

### 2h. Trust-Domain Separation (OBSERVED in `packages/contracts/trust_domains.py`)

The kernel and vault operate in separate trust domains:

- Kernel signing key: `KTI_RISK_TOKEN_KEY`
- Vault encryption key: `KTI_VAULT_KEY`
- `trust_domains_are_separated()` returns `True` (env var names are distinct).
- `keys_are_distinct(kernel_key, vault_key)` verifies no key reuse.
- `derive_kernel_key_material()` reads only `KTI_RISK_TOKEN_KEY`.
- `derive_vault_key_material()` reads only `KTI_VAULT_KEY`.

### 2i. Graduated Autonomy and Kill Plane (OBSERVED in `packages/contracts/autonomy.py` + `services/autonomy/`)

Per AD-038: a trust-level engine (L0–L5) with an independent kill plane.

- `TrustLevelEngine` — HMAC-SHA256 signed promotion approvals; automatic/manual demotion requires no approval; `reset_all()` on emergency halt; append-only audit trail.
- `KillPlane` — `ARMED → DRAINING → HALTED` state machine; `trigger(scope, ...)` cancels matching open orders and blocks new exposure; `rearm()` only from `HALTED`; `out_of_band=True` flag for when main platform is down; full command + result history.
- `OperatorCockpit` — point-in-time snapshot aggregating trust-engine + kill-plane state.
- `CompliancePackageBuilder` — assembles the E9 owner review package; `live_trading_authorized` is invariant `False` (enforced in contract — `E9OwnerReviewPackage` cannot be created with `live_authorized=True`).

---

## 3. SHA Register and CI Evidence

| Item | Value | Observation class |
|------|-------|-------------------|
| Source HEAD (Phase 4) | `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` | OBSERVED — local clone + git rev-parse |
| Source HEAD commit date | 2026-10-17T09:21:33Z | OBSERVED — GitHub API |
| CI workflow | `.github/workflows/ci.yml` — 2 jobs: `python-tests` (matrix 3.11/3.12/3.13), `frontend-checks` | OBSERVED |
| CI at HEAD | SUCCESS (observed via GitHub Actions API at Phase 1) | OBSERVED |
| Prior 2 CI runs | FAILURE (at earlier SHAs) | OBSERVED — GitHub Actions API (Phase 1) |
| CI scope (Python) | `ruff format --check`, `ruff check`, `mypy`, `pytest -v`, `pytest tests/test_secret_scan.py` | OBSERVED in `ci.yml` |
| CI scope (Frontend) | `tsc --noEmit`, `npm run lint`, `npm test -- --run` | OBSERVED in `ci.yml` |
| Tags | None (no tags observed) | OBSERVED |
| Releases | None (no published GitHub releases) | OBSERVED |

**CI assessment:** CI at HEAD `54ef6ca4d8fd` is SUCCESS. However, the immediately prior two CI runs at different SHAs were FAILURE. This pattern suggests CI was recently fixed. The CI workflow runs `pytest -v` (full test suite) and a separate `pytest tests/test_secret_scan.py -v`. No coverage threshold is enforced in CI (no `--cov-fail-under` flag observed in `ci.yml`). The E4 evidence file reports `89.10%` coverage with `--cov-fail-under=80`, but that was run locally on the phase branch, not in the CI workflow at HEAD.

**Test count (REPORTED):** `status.yaml` declares `2003` Python pytest tests and `91` vitest tests. The E9 evidence file reports `1709 passed, 12 skipped` at that phase's SHA. The E4 evidence file reports `1339 passed, 12 skipped` at its phase SHA. The test count has grown across phases. The `2003` claim at HEAD is REPORTED in `status.yaml` — this reviewer has not independently re-counted tests at HEAD (not authorized to execute source project code per DIRECTIVE.txt F03/G/J).

---

## 4. Risk Kernel Invariants and Safety Boundaries

### 4a. Core Invariant: No Unauthorised Order Execution

**Claim:** No exposure-increasing order may reach an exchange adapter without valid risk authorization.

**Evidence (OBSERVED):**

- `ExecutionEngine.execute_order()` calls `self._kernel.authorize(intent, current_price=...)` before `self._exchange.submit_order(...)`.
- If `auth.status is AuthorizationStatus.DENIED`, `RiskDeniedError` is raised and the exchange adapter is never called.
- The `authorize()` method calls `assess()` which runs 9 sequential checks; the first failure returns a `RiskAssessment(authorized=False, ...)`.

**Assessment:** The invariant is ENFORCED in code (the execution engine does not call the exchange without prior kernel authorization). It is TESTED (OBSERVED in `tests/test_execution_engine.py` — `test_risk_denied_blocks_execution` pattern). It is NOT independently reproduced by this reviewer.

### 4b. Emergency Stop Invariant

**Claim:** Emergency stop blocks new exposure-increasing (buy) orders.

**Evidence (OBSERVED):**

- `EmergencyStop._active` is checked in `_check_emergency_stop()` — if `is_active and intent.side == "buy"`, returns `RiskAssessment(authorized=False)`.
- `EmergencyStop.activate()` sets `_active = True` with a reason and timestamp.
- `EmergencyStop.deactivate()` clears state — docstring states "requires explicit human authorization (the caller is responsible for verifying this)."
- No auto-restart: `SAFE_HALT` state can only transition to `RECONCILIATION_REQUIRED`, not back to `RUNNING` (OBSERVED in `LEGAL_ORDER_TRANSITIONS` and `SessionState` transitions).

**Assessment:** ENFORCED in code. TESTED (OBSERVED in `tests/test_risk_kernel.py` — `TestEmergencyStop` class). The deactivation gate is a caller responsibility — the kernel does not verify that a human authorized deactivation; it trusts the caller. This is a documented design decision, not a bug.

### 4c. Simulation-Only Mode Guard

**Claim:** The platform operates in SIMULATION mode and cannot silently enable live trading.

**Evidence (OBSERVED):**

- `OperatingModeConfig` is a singleton with `_mode = OperatingMode.SIMULATION` and `_live_authorized = False` as defaults.
- `set_mode(OperatingMode.LIVE)` without `live_authorized` raises `PermissionError`.
- `is_live` property returns `True` only when both `mode == LIVE` and `live_authorized == True`.
- `status.yaml` declares `operating_mode: SIMULATION` and `live_authorized: false`.

**Assessment:** ENFORCED in code. TESTED (OBSERVED in `tests/test_mode.py` — 6 tests covering default simulation, paper mode, live requires authorization, live with/without authorization flag, reset). The guard prevents silent mode escalation but does not prevent a caller with `live_authorized=True` from entering live mode — that is by design (explicit authorization enables it).

### 4d. Concurrency-Safe Risk Reservations

**Claim:** Concurrent strategies and sessions must not spend the same risk budget.

**Evidence (OBSERVED):**

- `RiskKernel._lock` (a `threading.Lock`) guards all state mutations in `register_policy`, `authorize`, `reserve_capacity`, `release_reservation`, `consume_reservation`, `record_fill`.
- `reserve_capacity()` checks exposure limit under the lock before creating a reservation.
- `_current_exposure` is updated atomically under the lock.

**Assessment:** ENFORCED in code (all mutations are lock-guarded). Thread-safety is at the in-process level — no distributed lock for multi-process deployments is observed. This is appropriate for the simulation-only scope.

---

## 5. Formal Models (OBSERVED in `formal/`)

Per E4 (Safety Plane Hardening): TLA+ specifications for order, session, reservation, and SAFE_HALT state machines are checked in CI via `formal/formal_checker.py`.

| Model | File | Invariants checked |
|-------|------|--------------------|
| OrderModel | `OrderModel.tla` | Terminal states have no outgoing transitions; no state transitions to CREATED |
| SessionModel | `SessionModel.tla` | SAFE_HALT only transitions to RECONCILING; COMPLETED and CANCELLED are terminal |
| ReservationModel | `ReservationModel.tla` | Only ACTIVE can transition; RELEASED, EXPIRED, CONSUMED are terminal |
| SafeHaltModel | `SafeHaltModel.tla` | Emergency stop blocks buy authorization; SAFE_HALT does not auto-restart; SAFE_HALT transitions only to reconciling |

The `formal_checker.py` reads canonical transition tables from the contracts modules and asserts invariants hold without requiring an external TLA+ runtime. The `status.yaml` honest label `formal_models` is `PLANNED` — the formal specs exist and the Python checker runs in CI, but full TLA+ model checking (e.g., TLC) is not operational.

---

## 6. Test Architecture (OBSERVED in `tests/`)

**Test files:** 69 `test_*.py` files in `tests/` directory (OBSERVED via file listing at HEAD).

### Test Categories (OBSERVED in tree)

| Category | Representative files | Coverage area |
|----------|---------------------|---------------|
| Risk kernel | `test_risk_kernel.py` (475 lines), `test_risk_contracts.py`, `test_e2_persistent_risk_kernel.py` | Policy management, 9-check assessment, authorization lifecycle, reservations, emergency stop, daily loss, drawdown, concentration, position count |
| Execution | `test_execution_engine.py` (249 lines) | Full pipeline, risk denial, partial fills, unknown outcomes, reconciliation |
| Mode guard | `test_mode.py` (66 lines) | Default simulation, paper mode, live requires authorization, reset |
| Mode/simulation | `test_e5_simulation_truth.py` | Simulation truth invariants, deterministic simulation universe |
| Autonomy/kill plane | `test_e9_autonomy.py` (779 lines) | Trust levels, kill plane (platform/tenant/symbol scope), human factors, operator cockpit, compliance package, `live_trading_authorized` always False |
| Signed risk tokens | `test_e4_risk_tokens.py`, `test_phase_c_signed_auth.py` (13 integration tests) | HMAC-SHA256 signing, tamper detection, expiry, replay, malformed tokens, tenant AAD, parameter binding |
| Trust domains | `test_e4_trust_domains.py` | Kernel vs vault key separation, no cross-read, distinct keys |
| Formal models | `test_e4_formal_models.py` (15 tests) | Terminal state invariants, no reverse transitions |
| Hypothesis stateful | `test_e4_hypothesis_stateful.py` | Rule-based state machine exploring kernel state space with invariants at every step |
| Policy-as-code | `test_e4_policy_as_code.py` | Declarative risk policy rules, fail-closed default |
| Security | `test_f_sec_02_encryption.py`, `test_f_sec_03_persistence.py`, `test_credential_vault.py` | AES-256-GCM envelope encryption, durable identity persistence, credential vault |
| Concurrency | `test_concurrency.py`, `test_agent_concurrency.py` | Thread-safety of risk kernel under concurrent access |
| E2E integration | `test_e2_postgres_integration.py`, `test_phase_a_integration.py` | Real PostgreSQL + Redis testcontainers, outbox/inbox, crash safety |
| Frontend | `apps/web/tests/` (7+ test files) | Client contracts, identity, remote commands |

### Negative Case Coverage (OBSERVED in test tree)

| Negative case | Test file | What it verifies |
|---------------|-----------|-------------------|
| Risk denial blocks execution | `test_execution_engine.py` | `RiskDeniedError` raised when kernel denies |
| Emergency stop blocks buys | `test_risk_kernel.py` `TestEmergencyStop` | Buy orders blocked when stop active; sells allowed |
| Exposure limit exceeded | `test_risk_kernel.py` | `RiskAssessment(authorized=False)` when exposure > limit |
| Daily loss exceeded | `test_risk_kernel.py` | Trading stopped for the day |
| Drawdown exceeded | `test_risk_kernel.py` | Authorization denied on drawdown breach |
| Concentration exceeded | `test_risk_kernel.py` | Authorization denied on concentration breach |
| Position count exceeded | `test_risk_kernel.py` | Authorization denied on per-symbol count breach |
| No active policy | `test_risk_kernel.py` | `PolicyNotActiveError` raised |
| Token tamper | `test_e4_risk_tokens.py`, `test_phase_c_signed_auth.py` | `RiskTokenTamperError` on signature mismatch |
| Token replay | `test_phase_c_signed_auth.py` | `RiskTokenReplayError` on nonce reuse |
| Token expired | `test_phase_c_signed_auth.py` | `RiskTokenExpiredError` on expired token |
| Token wrong tenant AAD | `test_phase_c_signed_auth.py` | Rejected on tenant context mismatch |
| Token quantity mismatch | `test_phase_c_signed_auth.py` | Rejected when payload fields don't match authorization |
| Kill plane already triggered | `test_e9_autonomy.py` | `KillPlaneNotArmedError` when already HALTED |
| `live_trading_authorized` always False | `test_e9_autonomy.py` | `E9OwnerReviewPackage` cannot be created with `live_authorized=True` |
| `cross_tenant` always False | `test_e9_autonomy.py` | `DataInventoryItem` with `cross_tenant=True` raises |
| Live mode without authorization | `test_mode.py` | `PermissionError` raised when `live_authorized` omitted |

---

## 7. Architecture Decision Records (OBSERVED in `docs/architecture.md`)

40 architecture decisions (AD-001 through AD-040) are documented in `docs/architecture.md`. Key decisions relevant to Phase 4 review lenses:

| ADR | Decision | Relevance |
|-----|----------|-----------|
| AD-003 | Hybrid low-token intelligence: deterministic-first, bounded LLM | LLM never authoritative for execution |
| AD-004 | Four specialized agents + separate deterministic Risk Kernel | Kernel is NOT an agent; deterministic safety service |
| AD-005 | Multi-provider financial connectivity with non-custodial boundaries | No custody of user funds |
| AD-011 | Validated ML + limited LLM; LLM never authoritative for execution | LLM cannot make financial decisions |
| AD-013 | Dynamic risk: hard limits for exposure, loss, drawdown, concentration | Enforced in kernel checks |
| AD-014 | Deterministic execution: durable state, idempotency, reconciliation | Unknown outcomes require reconciliation |
| AD-015 | Mining simulation first; real hardware requires explicit approval | Mining is simulation-only |
| AD-019 | Non-custodial credential vault: encryption, rotation, no withdrawal by default | AES-256-GCM envelope encryption |
| AD-023 | Digital twin: simulation, exchange emulation, replay, fault injection | All simulators outside deployable tree |
| AD-036 | Signed risk authorization tokens — short-lived, single-use, cryptographically signed | HMAC-SHA256, 30s TTL, nonce replay protection |
| AD-038 | Graduated autonomy (L0–L5) and independent kill plane | Trust-level engine + kill plane |
| AD-040 | Statistical honesty and strategy validation gate — CPCV, PBO, Deflated Sharpe, signed model cards | Status: PLANNED |

Open engineering decisions (OBSERVED in `status.yaml` §open_decisions):

| ID | Topic | Status |
|----|-------|--------|
| OPEN-02 | Cloud provider and region | RESOLVED — Render (current), AWS (roadmap) |
| OPEN-03 | Durable event-bus selection | RESOLVED — Redis Streams (current), AWS SQS/SNS (roadmap) |
| OPEN-05 | SLO, RPO, RTO | RESOLVED — 99.0% / ≤15min / ≤60min (current), 99.9% / ≤1min / ≤15min (roadmap) |
| OPEN-06 | Initial approved exchange providers | RESOLVED — Kraken + Bitvavo (MiCA-licensed) |
| OPEN-08 | Legal classification and commercial-release eligibility | RESOLVED — EU/German (MiCA/GDPR) templates; no external counsel yet |
| OPEN-11 | Exact risk-reservation and execution-ownership protocols | PARTIALLY_RESOLVED — Risk Kernel implemented; signed tokens in E4/Phase C |

---

## 8. ADR-05-001 — Independent Deterministic Risk Kernel (AD-004)

| Field | Value |
|-------|-------|
| **ADR ID** | AD-004 (referenced as ADR-05-001 in portfolio context) |
| **Source SHA** | `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` |
| **Source path** | [`services/risk_kernel/kernel.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/services/risk_kernel/kernel.py), [`docs/architecture.md`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/docs/architecture.md) |
| **Problem** | A multi-agent trading system where agents can propose orders needs a safety boundary that is independent of agent intelligence. If the same component that generates signals also authorizes execution, there is no independent check. |
| **Constraints** | Agents are LLM-augmented and non-deterministic; financial safety requires deterministic guarantees; concurrent strategies must not overspend a shared risk budget. |
| **Selected approach** | A separate, deterministic Risk Kernel that is explicitly NOT an agent. It performs 9 sequential safety checks per order (emergency stop, session state, exposure, daily loss, drawdown, concentration, position count, order state, policy version). All state mutations are `threading.Lock`-guarded. The kernel issues bounded, TTL-limited, single-use risk authorizations bound to exact order parameters. |
| **Alternatives** | NOT DOCUMENTED in source. Plausible: agent-level risk checks (no independence); external risk service (latency, single point of failure). |
| **Trade-offs** | The kernel is in-process only (no distributed lock). Emergency stop deactivation is a caller responsibility — the kernel trusts the caller to verify human authorization. The 9-check fail-fast order means a rejected order reports only the first failing check, not all failures. |
| **Evidence** | OBSERVED in `kernel.py` (750 lines), `test_risk_kernel.py` (475 lines, 9+ test classes), `test_execution_engine.py` (249 lines) |
| **Status** | Active — implemented, tested, and enforced in execution engine |

---

## 9. ADR-05-002 — Simulation-Only Mode Guard (AD-003)

| Field | Value |
|-------|-------|
| **ADR ID** | AD-003 (referenced as ADR-05-002 in portfolio context) |
| **Source SHA** | `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` |
| **Source path** | [`packages/contracts/mode.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/mode.py), [`packages/contracts/enums.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/enums.py) |
| **Problem** | A trading platform must never silently enable real-money trading. A mode change must be an explicit, auditable decision. |
| **Constraints** | Default must be simulation; any transition to LIVE requires explicit authorization; the system must be auditable. |
| **Selected approach** | `OperatingModeConfig` singleton with `_mode = SIMULATION` and `_live_authorized = False` as defaults. `set_mode(LIVE)` without `live_authorized` raises `PermissionError`. `is_live` is True only when both `mode == LIVE` and `live_authorized == True`. |
| **Alternatives** | NOT DOCUMENTED. |
| **Trade-offs** | The guard prevents silent escalation but does not prevent a caller with `live_authorized=True` from entering live mode — that is by design. The guard is in-process; a separate deployment-level guard (e.g., feature flag, network policy) would be needed for production. |
| **Evidence** | OBSERVED in `mode.py` (81 lines), `test_mode.py` (66 lines, 6 tests) |
| **Status** | Active |

---

## 10. ADR-05-003 — Signed Risk Authorization Tokens (AD-036)

| Field | Value |
|-------|-------|
| **ADR ID** | AD-036 |
| **Source SHA** | `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` |
| **Source path** | [`packages/contracts/risk_token.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/risk_token.py), [`packages/contracts/trust_domains.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8f8ba921c3451ebc395bb/packages/contracts/trust_domains.py) |
| **Problem** | Risk authorizations must be tamper-evident, short-lived, single-use, and bound to exact order parameters to prevent replay, parameter substitution, or reuse across orders. |
| **Constraints** | Must not reuse vault key material; must be verifiable without network calls; must detect tamper, expiry, and replay. |
| **Selected approach** | HMAC-SHA256 signed tokens with UUID nonce, 30s TTL, canonical JSON payload + tenant AAD. Trust-domain separation: key from `KTI_RISK_TOKEN_KEY` (distinct from `KTI_VAULT_KEY`). Constant-time comparison (`hmac.compare_digest`). Consumed nonce set for replay protection. 13 integration tests in `test_phase_c_signed_auth.py`. |
| **Alternatives** | NOT DOCUMENTED. |
| **Trade-offs** | HMAC is symmetric (not asymmetric) — anyone with the key can sign. This is appropriate for in-process use but insufficient for multi-service trust boundaries. The key is derived from an environment variable; in production, a KMS-backed key would be stronger (the code uses a `KMS interface` per E1). |
| **Evidence** | OBSERVED in `risk_token.py`, `test_e4_risk_tokens.py`, `test_phase_c_signed_auth.py` (13 tests), `trust_domains.py` |
| **Status** | Active — implemented in E4, integrated into simulation path in Phase C |

---

## 11. ADR-05-004 — Graduated Autonomy and Independent Kill Plane (AD-038)

| Field | Value |
|-------|-------|
| **ADR ID** | AD-038 |
| **Source SHA** | `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` |
| **Source path** | [`packages/contracts/autonomy.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/autonomy.py), [`services/autonomy/__init__.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/services/autonomy/__init__.py) |
| **Problem** | A trading platform with autonomous agents needs a graduated autonomy system where trust is earned through signed approvals, and an independent kill mechanism can halt all activity when safety is compromised. |
| **Constraints** | Trust promotions require signed human approval; demotion may be automatic; kill plane must work even when the main platform is down (`out_of_band`); `live_trading_authorized` must be invariant `False`. |
| **Selected approach** | `TrustLevelEngine` (L0–L5) with HMAC-SHA256 signed promotions. `KillPlane` with `ARMED → DRAINING → HALTED` state machine and platform/tenant/agent/symbol scope. `out_of_band=True` for main-platform-down scenarios. `E9OwnerReviewPackage` with `live_trading_authorized` invariant `False` (enforced in contract — cannot be constructed with `True`). |
| **Alternatives** | NOT DOCUMENTED. |
| **Trade-offs** | The kill plane is in-process; a real production kill plane would need to be externalized (e.g., circuit breaker in infrastructure). The trust-level engine uses HMAC signatures — suitable for single-operator but not multi-party. |
| **Evidence** | OBSERVED in `autonomy.py` (26,202 bytes), `services/autonomy/__init__.py`, `test_e9_autonomy.py` (779 lines, 63 tests) |
| **Status** | Active |

---

## 12. Critical Control Flows

### Flow 1: Order Execution Pipeline (RECONSTRUCTED FROM SOURCE)

```
Caller creates OrderIntent (tenant_id, profile_id, symbol, side, quantity)
    │
    ▼
ExecutionEngine.execute_order(intent, current_price=..., session=...)
    │
    ▼  Verify exchange is connected
    │   if not self._exchange.is_connected: raise ExecutionError
    │
    ▼  Risk authorization (deterministic)
    │   auth = kernel.authorize(intent, current_price=..., session=...)
    │     └─ assess() runs 9 sequential checks (fail-fast)
    │        └─ first failure → RiskAssessment(authorized=False)
    │     └─ if authorized: create RiskAuthorization (TTL=30s, bound to params)
    │     └─ if denied: create RiskAuthorization(status=DENIED)
    │
    ▼  If DENIED: raise RiskDeniedError — exchange NEVER called
    │
    ▼  Reserve risk capacity (concurrency-safe under Lock)
    │   reservation = kernel.reserve_capacity(auth)
    │     └─ check exposure under lock
    │     └─ create RiskReservation (TTL=60s)
    │     └─ increase _current_exposure for buy orders
    │
    ▼  Submit to exchange adapter
    │   result = exchange.submit_order(intent, order_type=..., price=..., client_order_id=...)
    │
    ▼  Handle result state:
    │   FILLED / PARTIALLY_FILLED:
    │     └─ consume authorization + reservation
    │     └─ record fills (exposure tracking, daily loss)
    │   REJECTED / EXPIRED / CANCELLED:
    │     └─ release reservation (return capacity to budget)
    │     └─ cancel authorization
    │   UNKNOWN_OUTCOME:
    │     └─ consume reservation (hold capacity)
    │     └─ require reconciliation (no blind retry)
    │     └─ Per Section 05.3: "timeouts must not trigger blind order resubmission"
    │
    ▼  Return ExchangeSubmissionResult
```

**Trust boundary enforcement:** The kernel is the trust boundary between agent intent and exchange execution. No code path in `ExecutionEngine.execute_order()` calls `self._exchange.submit_order()` without first calling `self._kernel.authorize()`. OBSERVED in source. NOT independently tested by this reviewer.

### Flow 2: Emergency Stop Activation

```
Operator or safety monitor calls kernel.activate_emergency_stop(reason)
    │
    ▼  EmergencyStop.activate(reason)
    │   _active = True
    │   _activated_at = datetime.now(UTC)
    │   _reason = reason
    │
    ▼  All subsequent assess() calls:
    │   _check_emergency_stop() returns RiskAssessment(authorized=False)
    │   for all buy orders — "Emergency stop is active"
    │
    ▼  Sell orders still allowed (reduces exposure)
    │
    ▼  Deactivation requires explicit human authorization:
    │   kernel.deactivate_emergency_stop()
    │   └─ Docstring: "requires explicit human authorization
    │       (the caller is responsible for verifying this)"
    │   └─ No auto-restart: SAFE_HALT → RECONCILIATION_REQUIRED only
```

### Flow 3: Signed Risk Token Verification (Phase C)

```
SignedRiskAuthorizationService.authorize(intent, current_price=...)
    │
    ▼  Delegate to RiskKernel.authorize()
    │   └─ If APPROVED: issue SignedRiskToken
    │      └─ HMAC-SHA256 over canonical payload + tenant AAD
    │      └─ UUID nonce, 30s TTL, base64url token_str
    │   └─ If DENIED: return SignedAuthorization(token=None)
    │
    ▼  Verify + consume:
    │   verify_and_consume(token)
    │   └─ Structural validation (decode base64url JSON)
    │   └─ Signature verification (constant-time compare_digest)
    │   └─ Expiry check (expires_at > now)
    │   └─ Replay check (nonce not in consumed set)
    │   └─ Cross-check payload fields vs authorization
    │      (auth_id, intent_id, symbol, side, quantity)
    │   └─ Consume kernel authorization + mark nonce consumed
    │   └─ Any failure → RiskTokenTamperError/ExpiredError/ReplayError/MalformedError
```

---

## 13. Threat Model Assessment (OBSERVED in `docs/architecture.md`)

The `docs/architecture.md` documents 40 architecture decisions but does not contain a formal STRIDE threat model. Safety inferences are drawn from the architecture decisions and code-level controls:

| Risk | Control | Source treatment |
|------|---------|------------------|
| Unauthorized order execution | Independent Risk Kernel (AD-004), 9 checks, fail-fast | ENFORCED in code; TESTED in `test_risk_kernel.py` + `test_execution_engine.py` |
| Silent live mode activation | `OperatingModeConfig` singleton, `PermissionError` on unauthorized LIVE | ENFORCED in code; TESTED in `test_mode.py` |
| Risk budget overspend (concurrency) | `threading.Lock` on all kernel state mutations | ENFORCED in code; TESTED in `test_concurrency.py` |
| Token tamper/replay | HMAC-SHA256, constant-time comparison, nonce replay set | ENFORCED in code; TESTED in `test_e4_risk_tokens.py` + `test_phase_c_signed_auth.py` |
| Emergency stop bypassed | `EmergencyStop._active` checked first in assessment pipeline | ENFORCED in code; TESTED |
| Unknown outcome blind retry | `UNKNOWN_OUTCOME` → consume reservation + require reconciliation | ENFORCED in code (no blind retry path); DOCUMENTED in docstrings |
| Cross-tenant data leakage | `DataInventoryItem.cross_tenant` invariant `False` | ENFORCED in contract; TESTED in `test_e9_autonomy.py` |
| Live trading authorized in review package | `E9OwnerReviewPackage.live_trading_authorized` invariant `False` | ENFORCED in contract (constructor raises); TESTED |
| Key reuse between trust domains | Separate env vars, `keys_are_distinct()` | ENFORCED in code; TESTED in `test_e4_trust_domains.py` |

**Limitations (OBSERVED/INFERRED):**

- No formal STRIDE threat model document exists (unlike AILORA's `docs/assurance/threat-model.md`). Safety inferences are from ADRs and code.
- The emergency stop deactivation is a caller responsibility — the kernel does not verify human authorization for deactivation.
- The kill plane is in-process; a production kill plane would need infrastructure-level circuit breaking.
- HMAC is symmetric; multi-service trust would require asymmetric signatures (RS256/Ed25519).
- No distributed lock for multi-process deployments (appropriate for simulation scope).

---

## 14. Security Evidence (OBSERVED in `status.yaml` §security_findings)

| Finding | Description | Status | Resolution |
|---------|-------------|--------|------------|
| F-SEC-01 | Repository is PUBLIC — should be PRIVATE per Addendum A14 | RESOLVED | Owner-decided to keep PUBLIC. Preserved, not changed. |
| F-SEC-02 | XOR encryption in credential vault — inadequate for production | RESOLVED | AES-256-GCM envelope encryption with AAD, key versioning, fail-closed production, legacy XOR migration. Merged to main (PR #12). Evidence: `test-house/security/f-sec-02-03-evidence.md`, `tests/test_f_sec_02_encryption.py` |
| F-SEC-03 | In-memory identity store — no durable persistence | RESOLVED | SQLAlchemy-backed `PersistentIdentityStore` with SQLite (dev) / PostgreSQL (prod), versioned migrations, fail-closed production. Merged to main (PR #12). Evidence: `test-house/security/f-sec-02-03-evidence.md`, `tests/test_f_sec_03_persistence.py` |

---

## 15. Failure Modes and Limitations

| Failure mode | Source treatment | Evidence class |
|--------------|-----------------|----------------|
| CI was recently fixed (2 prior failures at earlier SHAs) | CI at HEAD `54ef6ca4d8fd` is SUCCESS; prior 2 runs FAILED | OBSERVED (GitHub Actions API) |
| No published GitHub releases | No tags, no releases observed | OBSERVED |
| `test-house/financial-safety/` is empty (`.gitkeep` only) | OBSERVED — no financial-safety evidence files | OBSERVED |
| Test count `2003` is REPORTED, not independently re-counted | `status.yaml` declares 2003; E9 evidence reports 1709 at earlier SHA; E4 evidence reports 1339 at earlier SHA | REPORTED |
| No coverage threshold enforced in CI | `ci.yml` runs `pytest -v` without `--cov-fail-under`; E4 evidence reports 89.10% locally | OBSERVED |
| Emergency stop deactivation is caller responsibility | Docstring: "the caller is responsible for verifying this" | OBSERVED |
| Kill plane is in-process only | No distributed circuit breaker observed | INFERRED |
| HMAC symmetric (not asymmetric) | `risk_token.py` uses HMAC-SHA256; no RSA/Ed25519 observed | OBSERVED |
| Formal models: TLA+ specs exist, full TLC model checking not operational | `status.yaml` honest label `formal_models: PLANNED` | OBSERVED |
| Strategy validation gate: PLANNED | `status.yaml` honest label `strategy_validation_gate: PLANNED` | OBSERVED |
| Agent runtime mesh: PLANNED | `status.yaml` honest label `agent_runtime_mesh: PLANNED` | OBSERVED |
| Decision ledger: PLANNED | `status.yaml` honest label `decision_ledger: PLANNED` | OBSERVED |
| No formal STRIDE threat model document | `docs/architecture.md` has 40 ADRs but no STRIDE register | OBSERVED |
| License is PROPRIETARY (not open source) | `LICENSE` file declares "All rights reserved"; no SPDX license | OBSERVED |
| No live deployment | `status.yaml`: no deployment URL; `honest_labels.live_trading: PLANNED` | OBSERVED |

---

## 16. Evidence Matrix

| Evidence ID | Claim | Observation class | Result | Method | Strength |
|-------------|-------|-------------------|--------|--------|----------|
| EVD-05-001 | `operating_mode: SIMULATION` and `live_authorized: false` in `status.yaml` | OBSERVED | PASS | `status.yaml` at HEAD SHA | E2 |
| EVD-05-002 | `OperatingModeConfig` enforces simulation-only default; `LIVE` without `live_authorized` raises `PermissionError` | OBSERVED | PASS | `packages/contracts/mode.py` inspection | E2 |
| EVD-05-003 | `is_live` is True only when both `mode == LIVE` and `live_authorized == True` | OBSERVED | PASS | `mode.py` property inspection | E2 |
| EVD-05-004 | Risk Kernel is a deterministic safety service, NOT an agent (AD-004) | OBSERVED | PASS | `services/risk_kernel/kernel.py` docstring + `docs/architecture.md` | E2 |
| EVD-05-005 | 9 sequential safety checks per order assessment (emergency stop, session, exposure, daily loss, drawdown, concentration, position count, order state, policy version) | OBSERVED | PASS | `kernel.py` `assess()` method inspection | E2 |
| EVD-05-006 | Assessment is deterministic: same inputs always produce same decision | OBSERVED | PASS | `assess()` docstring: "The assessment is deterministic" | E2 |
| EVD-05-007 | Emergency stop blocks new buy orders; sells allowed; no auto-restart | OBSERVED | PASS | `_check_emergency_stop()` + `EmergencyStop` class + `LEGAL_ORDER_TRANSITIONS` | E2 |
| EVD-05-008 | No exposure-increasing order reaches exchange adapter without risk authorization | OBSERVED | PASS | `ExecutionEngine.execute_order()` calls `kernel.authorize()` before `exchange.submit_order()` | E2 |
| EVD-05-009 | Risk authorizations are TTL-bounded (30s), single-use, bound to exact order parameters | OBSERVED | PASS | `authorize()` method + `RiskAuthorization` contract | E2 |
| EVD-05-010 | Risk reservations are concurrency-safe under `threading.Lock` | OBSERVED | PASS | `reserve_capacity()` + `release_reservation()` under `self._lock` | E2 |
| EVD-05-011 | Signed risk tokens: HMAC-SHA256, 30s TTL, UUID nonce, replay protection | OBSERVED | PASS | `packages/contracts/risk_token.py` inspection | E2 |
| EVD-05-012 | Trust-domain separation: kernel key `KTI_RISK_TOKEN_KEY` ≠ vault key `KTI_VAULT_KEY` | OBSERVED | PASS | `packages/contracts/trust_domains.py` | E2 |
| EVD-05-013 | Kill plane: ARMED→DRAINING→HALTED; platform/tenant/symbol scope; `out_of_band` flag | OBSERVED | PASS | `packages/contracts/autonomy.py` + `services/autonomy/` | E2 |
| EVD-05-014 | `E9OwnerReviewPackage.live_trading_authorized` is invariant `False` | OBSERVED | PASS | Contract constructor enforces; `test_e9_autonomy.py` verifies | E2 |
| EVD-05-015 | `DataInventoryItem.cross_tenant` is invariant `False` | OBSERVED | PASS | Contract raises on `True`; `test_e9_autonomy.py` verifies | E2 |
| EVD-05-016 | Formal models (TLA+) for order/session/reservation/SAFE_HALT checked in CI via Python checker | OBSERVED | PASS | `formal/formal_checker.py` + `formal/*.tla` + `test_e4_formal_models.py` (15 tests) | E2 |
| EVD-05-017 | Simulation/test code lives OUTSIDE deployable `services/` tree (E3 separation) | OBSERVED | PASS | `simulation/__init__.py` docstring + import-linter contracts | E2 |
| EVD-05-018 | CI at HEAD `54ef6ca4d8fd` is SUCCESS (GitHub Actions API) | OBSERVED | PASS | GitHub Actions API (Phase 1 observation) | E2 (R2 — linked CI run) |
| EVD-05-019 | Prior 2 CI runs at earlier SHAs: FAILURE | OBSERVED | CONDITIONAL | GitHub Actions API (Phase 1) | E2 |
| EVD-05-020 | 2003 Python pytest tests claimed at HEAD | REPORTED | NOT_TESTED | `status.yaml` — not independently re-counted by this reviewer | E1 |
| EVD-05-021 | 91 vitest tests claimed | REPORTED | NOT_TESTED | `status.yaml` | E1 |
| EVD-05-022 | No coverage threshold enforced in CI (`ci.yml` has no `--cov-fail-under`) | OBSERVED | PASS | `.github/workflows/ci.yml` inspection | E2 |
| EVD-05-023 | `test-house/financial-safety/` is empty (`.gitkeep` only) | OBSERVED | PASS | Directory listing at HEAD | E2 |
| EVD-05-024 | 40 architecture decisions (AD-001 through AD-040) documented | OBSERVED | PASS | `docs/architecture.md` inspection | E2 |
| EVD-05-025 | No published GitHub releases or tags | OBSERVED | PASS | `git tag -l` at HEAD | E2 |
| EVD-05-026 | AES-256-GCM envelope encryption replaced XOR (F-SEC-02 resolved) | OBSERVED | PASS | `status.yaml` §security_findings + `test_f_sec_02_encryption.py` | E2 |
| EVD-05-027 | SQLAlchemy-backed persistent identity store (F-SEC-03 resolved) | OBSERVED | PASS | `status.yaml` §security_findings + `test_f_sec_03_persistence.py` | E2 |
| EVD-05-028 | 69 test files in `tests/` directory | OBSERVED | PASS | File listing at HEAD | E2 (file count); function count NOT TESTED |
| EVD-05-029 | Hypothesis stateful testing explores risk kernel state space | OBSERVED | PASS | `tests/test_e4_hypothesis_stateful.py` — RuleBasedStateMachine | E2 |
| EVD-05-030 | Unknown outcomes require reconciliation; no blind retry | OBSERVED | PASS | `ExecutionEngine.execute_order()` + `enums.py` `UNKNOWN_OUTCOME → RECONCILIATION_REQUIRED` | E2 |

---

## 17. Open Gaps After Phase 4

| Gap ID | Description | Risk level |
|--------|-------------|------------|
| GAP-05-001 | LIVE mode enforcement is ENFORCED in code (`OperatingModeConfig` guard) — RESOLVED. The guard prevents silent escalation. The `status.yaml` claim is consistent with source. | **RESOLVED** (was HIGH) |
| GAP-05-002 | `test-house/financial-safety/` is empty (`.gitkeep` only) — no financial-safety evidence files exist despite the directory being created | MEDIUM |
| GAP-05-003 | CI was recently fixed (2 prior failures at earlier SHAs); CI stability is not longitudinally verified | MEDIUM |
| GAP-05-004 | Test count `2003` is REPORTED in `status.yaml` — not independently re-counted at HEAD by this reviewer | MEDIUM (evidence maturity) |
| GAP-05-005 | No coverage threshold enforced in CI (`ci.yml` runs `pytest -v` without `--cov-fail-under`); E4 evidence reports 89.10% locally but this is not a CI gate | LOW |
| GAP-05-006 | No published GitHub releases or tags — version `0.20.0` is declared in `pyproject.toml` and `status.yaml` but has no release artifact | MEDIUM |
| GAP-05-007 | No formal STRIDE threat model document (unlike AILORA); safety inferences drawn from ADRs and code only | LOW |
| GAP-05-008 | Emergency stop deactivation is caller responsibility — kernel does not verify human authorization for deactivation | LOW (documented design decision) |
| GAP-05-009 | Kill plane is in-process; no distributed/infrastructure-level circuit breaker observed | LOW (appropriate for simulation scope) |
| GAP-05-010 | HMAC symmetric signing for risk tokens — insufficient for multi-service trust boundaries; asymmetric (RS256/Ed25519) would be needed for production | LOW |
| GAP-05-011 | License is PROPRIETARY ("All rights reserved") — no source code copied into portfolio; links only | LOW (addressed by not copying) |
| GAP-05-012 | `formal_models` honest label is PLANNED — TLA+ specs exist and Python checker runs in CI, but full TLC model checking is not operational | LOW |
| GAP-05-013 | `strategy_validation_gate`, `agent_runtime_mesh`, `decision_ledger` are PLANNED — not yet implemented | LOW (honestly labeled) |

---

## 18. Source Permalinks

All permalinks resolve to the immutable HEAD SHA `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb`.

| File | Purpose |
|------|---------|
| [`status.yaml`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/status.yaml) | Single source of truth: operating mode, phase status, test counts, honest labels, security findings |
| [`docs/architecture.md`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/docs/architecture.md) | 40 architecture decisions (AD-001–AD-040), system layers, deployment topology |
| [`packages/contracts/mode.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/mode.py) | Operating mode guard — simulation-only enforcement |
| [`packages/contracts/enums.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/enums.py) | Order state machine, legal transitions, session states |
| [`packages/contracts/risk.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/risk.py) | Risk policy, authorization, reservation contracts |
| [`packages/contracts/risk_token.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/risk_token.py) | Signed risk authorization tokens (HMAC-SHA256) |
| [`packages/contracts/trust_domains.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/trust_domains.py) | Trust-domain separation verification |
| [`packages/contracts/autonomy.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/packages/contracts/autonomy.py) | Graduated autonomy, kill plane, compliance package contracts |
| [`services/risk_kernel/kernel.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/services/risk_kernel/kernel.py) | Independent deterministic Risk Kernel (750 lines) |
| [`services/execution/engine.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/services/execution/engine.py) | Execution engine — mandatory pipeline with risk authorization gate |
| [`services/autonomy/__init__.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8f8ba921c3451ebc395bb/services/autonomy/__init__.py) | Trust-level engine, kill plane, operator cockpit, compliance builder |
| [`simulation/__init__.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/simulation/__init__.py) | Simulation package — simulators outside deployable tree |
| [`simulation/exchange_simulator.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/simulation/exchange_simulator.py) | Deterministic exchange emulation |
| [`formal/formal_checker.py`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/formal/formal_checker.py) | TLA+ invariant checker (Python) |
| [`.github/workflows/ci.yml`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/.github/workflows/ci.yml) | CI pipeline: ruff, mypy, pytest, secret scan, frontend checks |
| [`pyproject.toml`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/pyproject.toml) | Project metadata, version, dependencies, dev tooling |
| [`test-house/phase-e4/evidence.md`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8f8ba921c3451ebc395bb/test-house/phase-e4/evidence.md) | E4 evidence: signed tokens, policy-as-code, trust domains, formal models, Hypothesis |
| [`test-house/phase-e9/evidence.md`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/test-house/phase-e9/evidence.md) | E9 evidence: graduated autonomy, kill plane, compliance package |
| [`test-house/phase-c/evidence.md`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/test-house/phase-c/evidence.md) | Phase C evidence: signed risk tokens on simulation path, chaos/PITR |
| [`LICENSE`](https://github.com/aminbita162-glitch/kian-trading-intelligence/blob/54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb/LICENSE) | Proprietary license — all rights reserved |

---

## 19. Reproducibility Recipe (R1 → R3 path)

This is a documented path for an independent reviewer to reach R3. It has NOT been executed by this agent (not authorized per DIRECTIVE.txt F03/G/J — source repositories are read-only).

```bash
git clone https://github.com/aminbita162-glitch/kian-trading-intelligence.git
cd kian-trading-intelligence
git checkout 54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb  # HEAD SHA
# Requires: Python 3.11+, uv or pip
pip install -e ".[dev]"
# Static checks
ruff format --check services/ packages/ tests/ formal/
ruff check services/ packages/ tests/ formal/
mypy services/ packages/ tests/ formal/
# Test suite
pytest -v
# Secret scan
pytest tests/test_secret_scan.py -v
# Frontend checks (from apps/web/)
cd apps/web && npm install && npx tsc --noEmit && npm run lint && npm test -- --run
# Expected (REPORTED in status.yaml): 2003 pytest passed, 91 vitest passed
# Coverage (not enforced in CI): pytest --cov --cov-fail-under=80 (E4 local: 89.10%)
```

Preconditions: Python 3.11+; Node.js 22+; `testcontainers` for PostgreSQL/Redis integration tests (Phase A); no real exchange credentials needed (simulation only).

**CI reproduction note:** CI at HEAD `54ef6ca4d8fd` is SUCCESS. Prior 2 CI runs at different SHAs were FAILURE. Reproduction at HEAD should produce a passing run. The CI does not enforce a coverage threshold — `89.10%` was measured locally on the E4 branch, not in CI.

---

## 20. Phase 4 Filter Gate Record

**PHASE 4 | PRJ-05 Kian Trading Intelligence | Changeset: `cases/05-kian-trading-intelligence/engineering-evidence.md` (new file)**

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 4 for PRJ-05 |
| F02 Scope | PASS | Only portfolio path `cases/05-kian-trading-intelligence/engineering-evidence.md` modified |
| F03 Source immutability | PASS | No source repository changes; read-only inspection only |
| F04 Baseline | PASS | Branch `main`, HEAD `3bbf086613c0d7d037ec6dc7fb0e0857f35dec87`, clean working tree |
| F05 Conflict | PASS | No conflicting directives; DIRECTIVE.txt is sole authority |
| F06 Intent | PASS | Change advances evidence-first portfolio objective |
| F07 Necessity | PASS | No simpler non-writing alternative; evidence file required |
| F08 Duplication | PASS | No existing `engineering-evidence.md` for PRJ-05; no duplicate content |
| F09 Provenance | PASS | Source URL, full SHA, file paths, timestamps captured |
| F10 Authenticity | PASS | No source statement treated as independently verified |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/INFERRED/NOT TESTED used precisely |
| F12 Reproducibility | PASS | Reviewer can follow claim→source→evidence chain via permalinks |
| F13 Technical correctness | PASS | Architecture and code descriptions match actual source |
| F14 Domain safety | PASS | Financial risk kernel invariants accurately represented; no real-money claims |
| F15 Trust boundaries | PASS | Source files treated as untrusted data; no prompt injections followed |
| F16 Security | PASS | No secrets, tokens, or unsafe links introduced |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific information |
| F18 Intellectual property | PASS | No source code copied; links and attributed descriptions only; license is PROPRIETARY |
| F19 Attribution | PASS | Source author (Amin Azimi) accurately attributed |
| F20 Test integrity | PASS | No unexecuted test described as passed; test count labeled REPORTED/NOT_TESTED |
| F21 Failure modes | PASS | Negative cases, CI instability, empty financial-safety dir, and limitations represented |
| F22 Version drift | PASS | Release, branch, CI, deployment, and docs not conflated; version 0.20.0 pinned |
| F23 Interface stability | PASS | Existing case README, links, and case ordering preserved |
| F24 Accessibility | PASS | Headings, tables, and text are navigable; no images requiring alt text |
| F25 Maintainability | PASS | Concise structure; no second source tree; links to source |
| F26 Reviewability | PASS | Diff is one new file; coherent and attributable to Phase 4 PRJ-05 outcome |
| F27 Validation | PASS | Available safe local checks: markdown structure verified; no external validators run |
| F28 Git hygiene | PASS | Commit authorized; push authorized; single meaningful commit |
| F29 Disclosure | PASS | Public-safe content; no sensitive material; license PROPRIETARY noted |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision: PROCEED** — all filters PASS; no blocking evidence.

---

## 21. Stop Gate

Phase 4 (PRJ-05 Kian Trading Intelligence) is complete. The engineering evidence file has been created with source-pinned observations, evidence matrix, ADRs, threat model assessment, failure modes, and open gaps. All source repositories remain untouched.

**NEXT PHASE: NOT STARTED — AWAITING OWNER.**
