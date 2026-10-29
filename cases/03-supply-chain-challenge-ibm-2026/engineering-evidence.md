# PRJ-03 — IBM Supply Chain Challenge: Engineering Evidence

**Project ID:** PRJ-03\
**Canonical name:** IBM Supply Chain Challenge — AI-Powered Supply Chain Optimization Platform\
**Source:** https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026\
**Source HEAD (Phase 3 inspection):** `d00c5ebb27948ce9e61cd63add127868f65e36dd` (2026-07-15)\
**Verified application commit:** `e39d3b1df0b9` (OBSERVED in commit log — see §3)\
**Release tag:** `v1.0.0-submission-final` (published 2026-07-15T23:21:15Z, non-prerelease)\
**License:** MIT (SPDX confirmed)\
**Phase 3 inspection date:** 2026-10-17\
**Independence:** Source/documentation observations only; no independent third-party verification claimed.\
**Independence level:** Source/documentation observations only; no independent reproduction claimed.\
**Evidence maturity:** R1 — source and configuration inspected at immutable revision\
**Reproducibility ladder:** R1 (source inspected). R2 not reached — no CI run artifact linked; tests REPORTED as passing by author but NOT independently reproduced.

---

## 1. Context and Scope

The IBM Supply Chain Challenge project is a competition submission for the **IBM SkillsBuild AI Builders with IBM Bob 2026** challenge. It implements an AI-assisted decision-support platform for supply chain operations: demand forecasting, inventory optimization, warehouse allocation, and route optimization.

**Non-goals (explicitly stated by author; OBSERVED in `docs/verification.md`):**

- Not a production-scale deployment
- Not a customer or revenue claim
- Not an IBM product or IBM-endorsed software
- Deterministic synthetic demo data only — not real-world supply chain data
- Measured results are baseline behavior demonstrations, not production KPIs

The portfolio case correctly qualifies these boundaries. No portfolio section implies IBM endorsement. Challenge submission equals participation, not certification.

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading `docs/architecture.md` and source tree at HEAD SHA `d00c5ebb`. Not a verified runtime topology.

### 2a. Layer Decomposition

```
HTTP Request
     │
     ▼
┌────────────────────────────────────────────┐
│  Routers  (backend/app/api/v1/routers/)    │
│  - Pydantic input validation               │
│  - JWT authentication (dependency)         │
│  - Calls one Service method                │
│  - Pydantic response serialization         │
│  - No business logic                       │
└───────────────────┬────────────────────────┘
                    │
                    ▼
┌───────────────────────────────────────────────┐
│  Services  (backend/app/services/)            │
│  - All business logic                         │
│  - Domain rule validation                     │
│  - Orchestrates Repositories                  │
│  - Calls AI modules when needed               │
│  - Controls transactions (commit/rollback)    │
└───────┬──────────────────────┬────────────────┘
        │                      │
        ▼                      ▼
┌──────────────────┐  ┌──────────────────────┐
│  Repositories    │  │  AI Modules (ai/)    │
│  SQL queries     │  │  Pluggable impls      │
│  No logic        │  │  Isolated, testable   │
└────────┬─────────┘  └──────────────────────┘
         │
         ▼
┌─────────────────────┐
│  ORM Models         │
│  (app/models/)      │
│  Table definitions  │
│  No business logic  │
└─────────────────────┘
```

Source: [`docs/architecture.md`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/docs/architecture.md) — OBSERVED at HEAD.

**Design principle (OBSERVED in source):** "No Business Logic in Routers — routers only validate input, call a service, and serialize output."

### 2b. Multi-Tenancy Model (OBSERVED in source)

Every domain entity (Warehouse, Product, InventoryItem, Order) carries an `organization_id` foreign key. The `CurrentUserContext` dependency resolves the calling user's organization from the JWT token and injects it into every service call. The `resolve_org_id()` method enforces that a non-admin user can only access their own organization's data.

Source: [`backend/app/core/dependencies.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/core/dependencies.py)

```python
def resolve_org_id(self, requested_org_id=None):
    """Return effective org_id: system_admin may supply any, others use their own."""
    if self.is_system_admin and requested_org_id is not None:
        return requested_org_id
    if not self.org_uuid:
        raise AuthorizationError("User is not associated with any organization")
    return self.org_uuid
```

**Tenant isolation assessment:** ENFORCED at the dependency injection layer — non-admin users receive their own `org_uuid` regardless of any query parameter. Cross-tenant read requires a `system_admin` role. **NOT TESTED** by independent runtime execution; test suite covers this path (see §6).

### 2c. Database Schema (OBSERVED in migration)

12 tables across 4 domains, defined in [`backend/migrations/versions/0001_initial_schema.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/migrations/versions/0001_initial_schema.py):

```
organizations
    ├── users
    ├── warehouses
    │       └── inventory_items ──── products
    └── orders ──── order_items ─────┘
            └── warehouse_allocation_results

demand_forecasts
inventory_recommendations
route_optimization_results
activity_logs
```

Single migration (`0001_initial_schema.py`). PostgreSQL native ENUMs defined for `user_role`, `order_status`, `order_priority`. SQLAlchemy Alembic manages schema evolution.

**Relevant engineering decision:** PostgreSQL ENUM/SQLAlchemy migration compatibility. The commit `e39d3b1df0b9` (message: "Align SQLAlchemy enums with PostgreSQL values") addresses this explicitly — OBSERVED in commit log.

---

## 3. SHA Register and Commit Evidence

| Item | Value | Observation class |
|------|-------|-------------------|
| Source HEAD (Phase 3) | `d00c5ebb27948ce9e61cd63add127868f65e36dd` | OBSERVED — GitHub API 2026-10-17 |
| Verified application commit | `e39d3b1df0b9` | OBSERVED — present in `git log` output (2026-07-15); previously BLOCKED by rate limit in Phase 1 |
| Alembic enum-fix commit | `e39d3b1df0b9` | OBSERVED — matches "Align SQLAlchemy enums" message |
| Final documentation commit | `73bcb7ff5a31` | OBSERVED — "Add final judge evidence and reproducibility verification" |
| Release tag annotated object SHA | `ce0d787761fe0b7e0d850f13c668abc9f43b5114` | OBSERVED — GitHub API tags endpoint |
| Release tag target | `v1.0.0-submission-final` → `main` branch | OBSERVED |
| Release published | 2026-07-15T23:21:15Z | OBSERVED |

**GAP-03-001 resolved:** The IBM commit SHA `e39d3b1df0b9` that was BLOCKED in Phase 1 due to API rate limits is now OBSERVED in the repository commit log on the `main` branch. The commit exists and its message is "Align SQLAlchemy enums with PostgreSQL values", confirming it is the final hardening commit before the release tag. The "69 tests passing" claim in `verification.md` is therefore supported by source evidence at that SHA (REPORTED by author; E1 → partially elevated by SHA confirmation, but NOT independently reproduced — see §6).

---

## 4. AI Module Architecture (OBSERVED in source)

Four AI modules, each implementing a defined abstract interface, located under `backend/ai/`:

| Module | Interface | Implementation | Algorithm |
|--------|-----------|----------------|-----------|
| Demand Forecasting | `DemandForecasterInterface` | `MovingAverageForecaster` | Moving average over configurable window |
| Inventory Optimization | `InventoryOptimizerInterface` | `EOQOptimizer` | Economic Order Quantity (EOQ) + safety stock |
| Warehouse Allocation | `WarehouseAllocatorInterface` | `WarehouseAllocationEngine` | Weighted scoring (coverage 70%, proximity 20%, capacity 10%) |
| Route Optimization | `RouteOptimizerInterface` | `NearestNeighborRouter` | Greedy nearest-neighbor heuristic |

Source: [`docs/architecture.md` AI Module section](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/docs/architecture.md) and file tree — OBSERVED.

**Pluggability (OBSERVED):** The interface layer enables algorithm substitution with a single-line service change. No other application code changes required. This is a documented design property, not a verified runtime swap.

### Verified Measured Results (REPORTED by author in `verification.md`)

The following values are from `docs/verification.md` at HEAD, produced by `scripts/evaluate_ai.py` on the deterministic synthetic scenario. Observation class: REPORTED (author's documented execution). Independent reproduction: NOT TESTED.

| Decision Area | Verified Result | Limitation |
|---------------|-----------------|-----------|
| 7-day demand forecast | 142.00 units | Synthetic data; moving average; not production benchmark |
| Daily average | 20.29 units/day | Same constraints |
| One-step holdout MAE | 4.00 units | Single sample; not multi-period evaluation |
| Safety stock | 23.91 units | EOQ model; 95% service level; synthetic input |
| Reorder point | 83.91 units | Same |
| Recommended order quantity | 96.66 units | Same |
| Expected shortage risk | 5.0% | Theoretical model output |
| Selected warehouse | Berlin West (score 0.9756) | Synthetic 3-warehouse scenario |
| Second-best warehouse score | 0.9102 | Score gap 0.0654 |
| Initial route distance | 167.59 km | 4-stop Berlin-area scenario |
| Optimized route distance | 133.77 km | Nearest-neighbor; no return leg |
| Route improvement | 20.18% | Haversine distance; straight-line, not road |

**These values demonstrate current baseline behavior of deterministic synthetic evaluation. They are not production KPIs, customer performance, or real-world optimization results. Stated explicitly in `verification.md` §8 Measurement Limitations.**

---

## 5. Authentication and Security Architecture (OBSERVED in source)

Source: [`backend/app/core/security.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/core/security.py)

### JWT Implementation (OBSERVED)

- **Algorithm:** HS256 (symmetric; secret key from `settings.SECRET_KEY`)
- **Access token:** includes `sub` (user ID), `iat`, `exp`, `jti` (unique ID per token), `type: access`, plus optional `role` and `org_id` claims
- **Refresh token:** separate, longer TTL, `type: refresh`
- **Token type enforcement:** `get_current_user_context` dependency rejects refresh tokens on protected routes (`payload.get("type") != "access"` → `AuthenticationError`)
- **Password hashing:** bcrypt via `passlib` (`CryptContext`)
- **Decoding:** `python-jose[cryptography]`; `JWTError` caught and converted to `AuthenticationError`

### Security Limitations (OBSERVED and INFERRED)

- **No RS256/OIDC:** HS256 with a shared secret. Production identity federation would require RS256 or OAuth2. Not claimed in portfolio.
- **No token revocation/blocklist:** `jti` claim is present but not checked against a revocation store. A stolen token is valid until expiry. Source does not implement a revocation mechanism.
- **No token revocation on password change:** INFERRED from source inspection — no mechanism to invalidate existing tokens when credentials change.
- **Demo credentials publicly documented:** `admin@supplychain-demo.com` / `SupplyChainDemo2026!` are intentionally public (challenge judge access). Not a secret leak; correctly scoped to challenge demo environment.
- **Activity log model OBSERVED:** `backend/app/models/activity_log.py` exists in tree. Audit log scope and coverage not inspected.

### Trust Boundary (OBSERVED)

```
Client → HTTPS (Render) → FastAPI → HTTPBearer dependency
                                          │
                                          ▼
                               decode_token() → payload extraction
                                          │
                                          ▼
                               CurrentUserContext
                                          │
                                          ▼
                               resolve_org_id() → org scoping
                                          │
                                          ▼
                               require_role() → authorization check
```

Authorization decision happens in routers before services are called. The JWT is the only identity proof; no additional server-side session.

---

## 6. Test Architecture (OBSERVED in source)

Source: [`backend/tests/`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/tests/) and [`backend/pytest.ini`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/pytest.ini)

### Test Modules (OBSERVED in tree)

| Path | Coverage area |
|------|---------------|
| `tests/api/test_auth_api.py` | Login, token refresh, /me endpoint — positive and negative cases |
| `tests/api/test_health_api.py` | Health check endpoint |
| `tests/api/test_inventory_api.py` | Inventory API endpoints |
| `tests/api/test_orders_api.py` | Orders API endpoints |
| `tests/api/test_products_api.py` | Products API endpoints |
| `tests/api/test_warehouses_api.py` | Warehouses API endpoints |
| `tests/unit/test_ai_modules.py` | All 4 AI modules: forecasting, EOQ, allocation, routing |
| `tests/unit/test_security.py` | JWT creation, decoding, tamper detection, password hashing |
| `tests/conftest.py` | Shared fixtures: SQLite in-memory test DB, FastAPI TestClient, seeded entities |

### Test Infrastructure (OBSERVED)

- **Test database:** SQLite in-memory with `StaticPool` — all connections share one DB instance, solving the FastAPI thread-pool isolation problem explicitly commented in `conftest.py`
- **Transaction isolation:** each test gets a connection, begins a transaction, and rolls back after — clean state per test
- **Dependency override:** `app.dependency_overrides[get_db]` substitutes test session in FastAPI's DI container
- **Cross-dialect UUID:** `UUIDType` TypeDecorator ensures `uuid.UUID` objects on both PostgreSQL (native UUID) and SQLite (CHAR(36)) — OBSERVED in `backend/app/db/types.py`

### Negative Case Coverage (OBSERVED)

Selected examples from inspected test files:

- `test_login_wrong_password` → expects HTTP 401
- `test_login_unknown_email` → expects HTTP 401
- `test_login_missing_fields` → expects HTTP 422 (Pydantic validation)
- `test_refresh_invalid_token` → expects HTTP 401
- `test_get_me_unauthenticated` → expects HTTP 401
- `test_tampered_token_raises` → expects `AuthenticationError`
- `test_invalid_token_raises` → expects `AuthenticationError`
- `test_no_warehouses_raises` → expects `ValueError` from allocation engine
- `test_predict_no_data` → zero demand forecast returns zero
- `test_zero_demand` → EOQ minimum floor applied

### 69-Test Claim Assessment

Author states in `verification.md`: "69 passed, 1 warning in 9.00s" at commit `e39d3b1df0b9`.

- Source SHA `e39d3b1df0b9` OBSERVED in commit log (Phase 3 confirmation)
- 8 test files OBSERVED (6 API + 2 unit); conftest.py provides fixtures
- No CI workflow — no automated run artifact to link
- 1 warning: Python `crypt` module deprecation from `passlib` bcrypt integration — OBSERVED as documented in `verification.md` §7 "Verified Warning Boundary"; does not affect test results per author
- **Result status:** REPORTED (E1 → E1+, SHA confirmed). Independent execution: NOT TESTED. Evidence strength remains below E3 without a linked CI run or independent reproduction.

---

## 7. Deployment Model (OBSERVED in source, NOT TESTED live)

| Property | Value | Observation class |
|----------|-------|-------------------|
| Platform | Render (free tier) | REPORTED in `docs/verification.md` |
| Live API URL | https://ibm-supply-chain-api.onrender.com | REPORTED — endpoint accessibility NOT TESTED |
| Containerization | Docker + Docker Compose | OBSERVED (`docker-compose.yml`, `backend/Dockerfile` in tree) |
| Database | PostgreSQL 15+ (production) | REPORTED in `docs/verification.md` §4 |
| Test database | SQLite in-memory | OBSERVED in `conftest.py` |
| Python runtime | 3.11.15 (verified by author) | REPORTED in `verification.md` |
| Cold start | 30–60 seconds (Render free tier) | REPORTED by author in `verification.md` §9 |

Docker Compose verification is documented in `verification.md` §11 but marked NOT COMPLETED in the final checklist (Docker not available during author's final verification). Live health check confirmed by author on 2026-07-16; not independently re-verified.

**Deployment/source drift:** HEAD `d00c5ebb` is the final documentation commit. Deployed commit is `e39d3b1df0b9`. These are different commits — `d00c5ebb` only updates documentation files (README, verification.md) after the functional freeze. Application code deployed on Render is `e39d3b1df0b9`. This is expected and correctly documented by the author.

---

## 8. ADR-03-001 — Cross-Dialect UUID TypeDecorator

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-03-001 |
| **Source SHA** | `d00c5ebb27948ce9e61cd63add127868f65e36dd` |
| **Source path** | [`backend/app/db/types.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/db/types.py) |
| **Problem** | PostgreSQL uses native UUID columns (efficient, indexed). SQLite (used in tests) has no UUID type and requires string storage. Tests would fail or require separate ORM models without an abstraction layer. |
| **Constraints** | Single SQLAlchemy model set must work on both dialects. Tests must not require PostgreSQL infrastructure. |
| **Selected approach** | `UUIDType(TypeDecorator)`: dispatches to `PG_UUID(as_uuid=True)` on PostgreSQL; to `CHAR(36)` on other dialects. Both sides coerce to/from `uuid.UUID` objects transparently. |
| **Alternatives** | NOT DOCUMENTED in source. Plausible alternatives: (a) separate test models; (b) mock PostgreSQL with testcontainers; (c) always store UUID as string. |
| **Trade-offs** | The TypeDecorator approach avoids test infrastructure complexity. Risk: any dialect-specific UUID behavior divergence between SQLite and PostgreSQL is not caught by the test suite (e.g., index efficiency, sort order). |
| **Failure implications** | If the coercion logic has a bug, UUID comparisons could fail silently on one dialect. Tests cover positive UUID operations; no explicit test for UUID boundary values. |
| **Evidence** | OBSERVED in source at HEAD SHA |
| **Status** | Active — in use at HEAD |

---

## 9. ADR-03-002 — SQLAlchemy Enum / PostgreSQL Migration Alignment

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-03-002 |
| **Source SHA** | `e39d3b1df0b9` |
| **Source path** | Commit message + `backend/migrations/versions/0001_initial_schema.py` |
| **Problem** | PostgreSQL native ENUM types created by Alembic must match SQLAlchemy model `Enum` declarations exactly. A mismatch between Python enum member names and PostgreSQL ENUM values causes runtime errors on insert/update. |
| **Context** | The directive specifically calls out commit `e39d3b1df0b9` as the SQLAlchemy enum migration fix. The commit message "Align SQLAlchemy enums with PostgreSQL values" OBSERVED in Phase 3. |
| **Selected approach** | Alembic migration creates PostgreSQL ENUMs using `op.execute("CREATE TYPE ...")` with explicit string values. `create_type=False` on SQLAlchemy ENUM columns prevents Alembic from attempting duplicate type creation. |
| **Alternatives** | NOT DOCUMENTED. Could use string columns instead of native ENUMs; loses database-level constraint enforcement. |
| **Trade-offs** | Native ENUM: enforces constraint at DB level; requires careful migration sequencing; harder to alter. String column: flexible; no DB constraint. |
| **Evidence** | Commit `e39d3b1df0b9` OBSERVED in commit log; migration file inspected at HEAD |
| **Status** | Resolved in final submission commit |

---

## 10. Critical Control Flows

### Flow 1: Authenticated Supply Chain Request

```
Client POST /api/v1/orders/{id}/allocate
    │
    ▼ HTTPBearer → credentials extracted
    │
    ▼ get_current_user_context()
        - decode_token() → validate HS256 signature + expiry
        - type == "access" check
        - extract sub, role, org_id
    │
    ▼ Router: ctx.resolve_org_id() → enforced org scoping
    │
    ▼ AIService.allocate_order(order_id, org_id)
        - Load Order (OrderRepository) — organization_id filter ENFORCED
        - Load Warehouses (WarehouseRepository) — org_id filter ENFORCED
        - Load Inventory (InventoryRepository) — org_id filter ENFORCED
        - AllocationEngine.allocate(order, warehouses, inventory) → score-ranked result
        - Persist WarehouseAllocationResult
    │
    ▼ HTTP 200 with allocation details + explanation
```

**Trust boundary enforcement:** org_id injected from JWT, not from request body. A non-admin client cannot specify a different org_id. ENFORCED at dependency layer. OBSERVED in source. NOT independently tested.

### Flow 2: Token Rejection Paths (OBSERVED in test coverage)

| Condition | Enforcement | Test coverage |
|-----------|-------------|---------------|
| Missing Authorization header | `AuthenticationError` → HTTP 401 | OBSERVED in test_auth_api.py |
| Wrong password | `AuthenticationError` → HTTP 401 | OBSERVED in test_auth_api.py |
| Tampered token signature | `JWTError` → `AuthenticationError` → HTTP 401 | OBSERVED in test_security.py |
| Expired token | `JWTError` → `AuthenticationError` | NOT TESTED explicitly (no expiry simulation test observed) |
| Refresh token used on protected route | `AuthenticationError` (type check) → HTTP 401 | OBSERVED in test_auth_api.py (refresh flow) |
| Insufficient role | `AuthorizationError` → HTTP 403 | INFERRED — `require_role()` OBSERVED in dependencies.py; router-level coverage NOT inspected |

---

## 11. Failure Modes and Limitations

| Failure mode | Source treatment | Evidence class |
|--------------|-----------------|----------------|
| Database unavailable | `psycopg2.OperationalError` propagates to `AppException` (HTTP 500) handler | INFERRED from exception handler map in `main.py` |
| Render cold start (30–60s) | Documented in `verification.md` §9 with acceptance criteria | REPORTED |
| No CI workflow | Tests require manual execution; no automated run artifact | OBSERVED (no `.github/workflows/`) |
| Single migration | Schema evolution history is thin; one initial schema file | OBSERVED |
| No token revocation | HS256 tokens valid until expiry; no blocklist | INFERRED from security.py inspection |
| Docker Compose verification incomplete | Marked NOT COMPLETE in `verification.md` §15 freeze checklist | OBSERVED (checklist item unchecked) |
| Route optimization ignores return leg | `docs/verification.md` explicitly states "excludes a return leg to the warehouse" | REPORTED — measurement limitation documented by author |
| Moving average is a simple baseline | No ARIMA, ML, or seasonal adjustment; single-step MAE only | OBSERVED in algorithm implementation |
| EOQ model assumptions | Constant demand rate, no backorders, no quantity discounts assumed | INFERRED from `eoq_optimizer.py` presence; detailed assumptions NOT inspected |
| Warehouse scoring weights hardcoded | 70% coverage / 20% proximity / 10% capacity — not configurable per org | REPORTED in `verification.md`; weights confirmed by evaluate_ai.py source |

---

## 12. Evidence Matrix

| Evidence ID | Claim ID | Claim | Observation class | Result | Method | Evidence strength |
|-------------|----------|-------|-------------------|--------|--------|------------------|
| EVD-03-001 | CLM-03-001 | 69 tests passing at commit `e39d3b1df0b9` | REPORTED | CONDITIONAL | Author statement in `verification.md`; SHA now OBSERVED in commit log; no CI artifact | E1 (elevated from BLOCKED to CONDITIONAL: SHA confirmed) |
| EVD-03-002 | CLM-03-002 | MIT license | OBSERVED | PASS | GitHub API SPDX confirmation | E2 |
| EVD-03-003 | CLM-03-003 | Release tag `v1.0.0-submission-final` exists | OBSERVED | PASS | GitHub API release endpoint + refs endpoint | E2 |
| EVD-03-004 | CLM-03-004 | No IBM endorsement implied | OBSERVED | PASS | Portfolio README inspected; correct qualification present | E2 |
| EVD-03-005 | — | 4-layer clean architecture (Router→Service→Repository→Model) | OBSERVED | PASS | `docs/architecture.md` + source tree inspection | E2 |
| EVD-03-006 | — | Multi-tenant org scoping via JWT dependency injection | OBSERVED | PASS | `backend/app/core/dependencies.py` inspected | E2 |
| EVD-03-007 | — | Cross-dialect UUID TypeDecorator | OBSERVED | PASS | `backend/app/db/types.py` inspected | E2 |
| EVD-03-008 | — | SQLAlchemy enum migration fix at `e39d3b1` | OBSERVED | PASS | Commit log confirms SHA and message | E2 |
| EVD-03-009 | — | Test suite: SQLite in-memory with StaticPool | OBSERVED | PASS | `backend/tests/conftest.py` inspected | E2 |
| EVD-03-010 | — | Negative test cases for auth boundaries | OBSERVED | PASS | `test_auth_api.py` and `test_security.py` inspected | E2 |
| EVD-03-011 | — | AI modules implement abstract interfaces | OBSERVED | PASS | `backend/ai/interfaces/` + implementation tree | E2 |
| EVD-03-012 | — | Exception-to-HTTP mapping in `main.py` | OBSERVED | PASS | `backend/app/main.py` inspected | E2 |
| EVD-03-013 | — | Live endpoint accessibility on 2026-07-16 | REPORTED | NOT_TESTED | Author's freeze checklist; no independent HTTP check | E1 |
| EVD-03-014 | — | Docker Compose verification | NOT_TESTED | NOT_TESTED | Unchecked in author's freeze checklist | E0 |
| EVD-03-015 | — | Test count = 69 (static source count, not verified by file inspection) | NOT_TESTED | NOT_TESTED | 8 test files OBSERVED; individual test count NOT verified by file inspection | E2 (files) / E0 (count) |

---

## 13. Open Gaps After Phase 3

| Gap ID | Description | Risk level |
|--------|-------------|------------|
| GAP-03-002 | No CI workflow — test execution cannot be automated or independently reproduced without manual setup | MEDIUM |
| GAP-03-003 | Token expiry simulation not observed in test suite; expired token handling untested | LOW |
| GAP-03-004 | Router-level `require_role()` coverage not fully inspected; authorization test completeness unknown | MEDIUM |
| GAP-03-005 | Docker Compose verification marked incomplete by author | LOW |
| GAP-03-006 | No independent R3/R4 reproduction of test results | MEDIUM (evidence maturity ceiling at R1/R2) |
| GAP-03-007 | EOQ model assumptions (`eoq_optimizer.py`) not inspected in detail | LOW |
| GAP-03-008 | Activity log model present in tree but scope/coverage not verified | LOW |

---

## 14. Source Permalinks

All permalinks resolve to the immutable HEAD SHA `d00c5ebb27948ce9e61cd63add127868f65e36dd`.

| File | Purpose |
|------|---------|
| [`docs/architecture.md`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/docs/architecture.md) | System architecture, layer decomposition, flow examples |
| [`docs/verification.md`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/docs/verification.md) | Verification pack, measured results, freeze checklist |
| [`backend/app/core/security.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/core/security.py) | JWT implementation, password hashing |
| [`backend/app/core/dependencies.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/core/dependencies.py) | Dependency providers, CurrentUserContext, org scoping |
| [`backend/app/db/types.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/db/types.py) | Cross-dialect UUIDType TypeDecorator |
| [`backend/app/main.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/app/main.py) | App factory, exception handlers, router registration |
| [`backend/migrations/versions/0001_initial_schema.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/migrations/versions/0001_initial_schema.py) | Database schema, ENUM definitions |
| [`backend/tests/conftest.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/tests/conftest.py) | Test fixtures, SQLite StaticPool setup |
| [`backend/tests/unit/test_security.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/tests/unit/test_security.py) | JWT and password security unit tests |
| [`backend/tests/unit/test_ai_modules.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/tests/unit/test_ai_modules.py) | AI module unit tests: forecasting, EOQ, allocation, routing |
| [`backend/tests/api/test_auth_api.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/tests/api/test_auth_api.py) | Auth API integration tests |
| [`scripts/evaluate_ai.py`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/scripts/evaluate_ai.py) | Deterministic AI evaluation runner |
| [`backend/requirements.txt`](https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026/blob/d00c5ebb27948ce9e61cd63add127868f65e36dd/backend/requirements.txt) | Pinned production dependencies |

---

## 15. Reproducibility Recipe (R1 → R3 path)

This is a documented path to reach R3 independently. It has NOT been executed (not authorized).

```bash
git clone https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026.git
cd IBM-Bob-Challenge-2026
git checkout e39d3b1df0b9  # verified application commit
cd backend
python3.11 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt
python -m pytest -q
# Expected: 69 passed, 1 warning in ~9s
cd ..
python scripts/evaluate_ai.py
# Expected: "Evaluation completed successfully."
```

Preconditions: Python 3.11; no database required (tests use SQLite in-memory; evaluate_ai.py is database-free). Verified Python runtime: 3.11.15 per author.
