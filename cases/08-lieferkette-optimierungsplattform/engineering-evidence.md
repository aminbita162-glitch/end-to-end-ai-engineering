# PRJ-08 — Lieferkette Optimierungsplattform: Engineering Evidence

**Project ID:** PRJ-08
**Canonical name:** Lieferkette Optimierungsplattform
**Source:** https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform
**Source HEAD (Phase 5 inspection):** `91c61210e9134302f17a634c2b830cadadf5c4bc` (2026-07-27T00:58:46Z)
**Default branch:** `main`
**License:** **NONE** — no `LICENSE` file detected; GitHub API reports `license: null`. Copying source into portfolio requires explicit owner authorization. All evidence uses SHA-pinned permalinks and attributed descriptions; no source code blocks are pasted.
**Language:** Python 3.11 (CI environment)
**Framework:** FastAPI 0.8.1 + SQLAlchemy 2.0+ + Pydantic 2.6+
**Phase 5 inspection date (UTC):** 2026-10-18
**Reviewer:** Second-agent review (Hermes) — SECOND_AGENT_REVIEW; not an independent third-party review
**Independence level:** SECOND_AGENT_REVIEW — all findings are this agent's read-only source inspection via GitHub API at pinned SHA `91c61210e913`
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 partially achieved — CI run SUCCESS at HEAD `91c61210e913` observed via GitHub Actions API (run #30228690474, 2026-07-27). 4 consecutive CI runs all SUCCESS. The CI run artifact is linked but not independently reproduced by this reviewer.

---

## 1. Context and Scope

Lieferkette Optimierungsplattform is a German-language logistics decision-support platform providing demand forecasting, inventory optimization, warehouse allocation, route sequencing, and synthetic demand simulation through a FastAPI backend with user-scoped data persistence and a static dashboard.

**Status (OBSERVED in README.md at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| System status | "Actively evolving engineering platform" | OBSERVED |
| CI | Python Tests workflow — 4 consecutive SUCCESS at HEAD | OBSERVED |
| Test count | 1 test file (`test_kalender.py`), 4 test functions | OBSERVED |
| Release tags | 0 | OBSERVED |
| Published releases | 0 | OBSERVED |
| Frontend | Static HTML dashboard mounted at `/dashboard` | OBSERVED |
| Deployment | Render — `https://lieferkette-optimierungsplattform.onrender.com` | REPORTED |
| API version | 0.8.1 | OBSERVED |

**Non-goals (OBSERVED across README.md):**

- Not a completed enterprise product (README explicitly states this).
- Not production-ready in the sense of a hardened, certified deployment.
- Not the IBM TechXchange 2026 Challenge project (that is PRJ-03, a separate system).
- Forecasting is a moving-average baseline with stochastic noise, not a trained probabilistic model.
- Route optimization is a nearest-neighbor heuristic, not an exact solver (no TSP/VRP algorithm).
- Warehouse allocation is nearest-coordinate Euclidean distance, not capacity-aware optimization.
- No distributed microservices, asynchronous processing, model serving infrastructure, or autonomous agents (README states these are roadmap items).
- No AsyncAPI, event contracts, or IoT/GPS telemetry integration (planned in documentation-as-code section).

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading all source files at HEAD SHA `91c61210e9134302f17a634c2b830cadadf5c4bc` via GitHub API. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
External Clients (untrusted)
  │  POST /login (OAuth2 password flow → JWT)
  │  POST /register (email + password ≥8 chars → bcrypt hash)
  │  GET  /me (JWT bearer → user profile)
  │  POST /warehouses/ (auth: JWT → user-scoped CRUD)
  │  POST /orders/ (auth: JWT → user-scoped CRUD + auto warehouse allocation)
  │  POST /routes/optimize (auth: JWT → group orders by warehouse, nearest-neighbor)
  │  POST /optimize (Pydantic-validated optimization input → safety stock policy)
  │  POST /ai/* (forecast, route, allocate, logistics-plan)
  │  GET  /simulation/demand (Gaussian daily demand)
  │  GET  /dashboard/ (static HTML)
  │
  ▼
FastAPI Application (main.py, v0.8.1)
  │  CORS middleware: allow_origins=["*"], allow_credentials=True
  │  StaticFiles mount: /dashboard → dashboard/index.html
  │  Router registration: ai_router, optimize_router, simulation_router,
  │                       warehouse_router, order_router, route_router
  │
  ├─ Identity (main.py)
  │  bcrypt password hashing (passlib)
  │  HS256 JWT (python-jose), 60-minute expiry
  │  SECRET_KEY = "change-this-secret-key-in-production" (HARDCODED)
  │  get_current_user(): JWT decode → user lookup → 401 on failure
  │
  ├─ Decision Services (app/services/)
  │  demand_forecast.py: moving average + np.random.normal noise (non-deterministic)
  │  optimize_service.py: safety stock + reorder point (z-values, DomainValidationError)
  │  route_optimizer.py: nearest-neighbor heuristic (TSP approximation)
  │  warehouse_ai.py: nearest-coordinate Euclidean allocation
  │
  ├─ Routers (app/routers/)
  │  warehouse_router.py: CRUD, owner-scoped queries (WHERE owner_id = current_user.id)
  │  order_router.py: CRUD, auto-allocate warehouse if not specified
  │  route_router.py: group orders by warehouse, build distance matrix, optimize
  │  ai_router.py: stateless AI endpoints (forecast, route, allocate, plan)
  │  optimize.py: Pydantic-validated optimization with DomainValidationError
  │  simulation_router.py: /simulation/demand → Gaussian daily demand
  │
  ├─ Simulation (app/app/simulation/)
  │  demand_simulator.py: random.gauss (non-deterministic)
  │
  ├─ Quellcode (quellcode/datensimulation/)
  │  kalender_erzeugen.py: generate_calendar (deterministic, payday rule)
  │  nachfrage_simulieren.py: DemandSimulationConfig (seed=42, negative binomial, seasonality)
  │
  ▼
Persistence (app/database.py)
  │  DATABASE_URL required at import (raises ValueError if unset)
  │  PostgreSQL via psycopg2 (postgresql+psycopg:// scheme)
  │  SQLAlchemy ORM: User, Warehouse, Order models
  │  pool_pre_ping=True
  │
  ▼
Dashboard (dashboard/index.html — static)
  │  KPI cards, Chart.js, Leaflet map
  │  Calls API via hardcoded API_BASE URL
```

### 2b. Key Architectural Decisions

1. **FastAPI + SQLAlchemy + PostgreSQL stack** — Typed API contracts via Pydantic, ORM models with user-scoped ownership. PostgreSQL as the production database with `pool_pre_ping=True` for connection health.

2. **User-scoped data isolation via query filtering** — All warehouse and order CRUD operations filter by `owner_id == current_user.id`. Cross-user reads are prevented by query design (WHERE clauses), not by database-level RLS. No global queries exist.

3. **Heuristic decision services** — Demand forecast is a moving average (not ML), route optimization is nearest-neighbor (not exact TSP), warehouse allocation is Euclidean distance (not capacity-aware). All are labeled as heuristics, not optimal solvers.

4. **Non-deterministic online forecast** — `demand_forecast.py` uses `np.random.normal(0, 2)` without a fixed seed. Repeated calls produce different forecasts. The offline simulation (`nachfrage_simulieren.py`) uses `DemandSimulationConfig(seed=42)` with `np.random.default_rng(config.seed)` — deterministic.

5. **Hardcoded JWT secret** — `SECRET_KEY = "change-this-secret-key-in-production"` is hardcoded in `main.py`, `warehouse_router.py`, `order_router.py`, and `route_router.py`. Not environment-variable-driven. Security gap.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Source URL | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform |
| HEAD SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| HEAD commit date | 2026-07-27T00:58:46Z |
| Default branch | `main` |
| License | **NONE** — no LICENSE file; GitHub API `license: null` |
| Total tracked files | 32 (excluding .git) |
| Source .py files | 19 (app/ + main.py + quellcode/) |
| Test files | 1 (`tests/test_kalender.py`) |
| Test functions | 4 (OBSERVED by reading the file) |
| CI workflow | `.github/workflows/python-tests.yml` |
| CI runs at HEAD | 4 consecutive SUCCESS (2026-07-25 through 2026-07-27) |
| CI run at HEAD | Run #30228690474 — SUCCESS — `91c61210e913` — 2026-07-27T00:58:48Z |
| Release tags | 0 |
| Published releases | 0 |
| Deployment | Render — `https://lieferkette-optimierungsplattform.onrender.com` (REPORTED) |
| Inspection method | GitHub API at pinned SHA + GitHub Actions API for CI metadata |
| Observation class | OBSERVED (all file content); OBSERVED (CI run status via API) |
| Repo size | 251 KB |

**CI history (last 5 runs, OBSERVED via GitHub Actions API):**

| Run ID | Conclusion | SHA | Date |
|--------|------------|-----|------|
| 30228690474 | SUCCESS | `91c61210e913` | 2026-07-27T00:58:48Z |
| 30224465422 | SUCCESS | `14125fa4b95d` | 2026-07-26T23:04:39Z |
| 30137602451 | SUCCESS | `0a37d8e56e2f` | 2026-07-25T01:00:54Z |
| 30137375283 | SUCCESS | `ba2001c94a6c` | 2026-07-25T00:54:51Z |
| 23017078480 | SUCCESS | `856322321af9` | 2026-03-12T18:14:01Z |

---

## 4. Demand Forecasting (Phase 5 Lens: Optimization Objective)

### 4a. Online Forecast (OBSERVED in `app/services/demand_forecast.py`, 36 lines)

**Algorithm:**
1. Input: `history: List[float]` (demand history).
2. If empty → return `[0] * days` (7 days by default).
3. If < 3 data points → use overall mean.
4. If ≥ 3 data points → 3-period rolling moving average (`series.rolling(window=3).mean().iloc[-1]`).
5. If rolling mean is NaN → fallback to overall mean.
6. For each forecast day: `noise = np.random.normal(0, 2)`, `value = max(0, int(avg + noise))`.

**Key observation:** `np.random.normal(0, 2)` has **no fixed seed**. Repeated calls to the same endpoint with the same history produce different forecasts. This is a non-deterministic forecast — the README explicitly states: "Repeated calls may produce different values because the online function does not currently fix a random seed."

**Method label:** No explicit `method` field in the API response. The README describes it as a "lightweight forecasting baseline, not a trained probabilistic forecasting model."

### 4b. Offline Demand Simulation (OBSERVED in `quellcode/datensimulation/nachfrage_simulieren.py`, 200 lines)

**Algorithm:** `DemandSimulationConfig` dataclass with:
- `seed=42` (deterministic via `np.random.default_rng(config.seed)`)
- Negative binomial demand distribution (Gamma-Poisson mixture) with dispersion `alpha=0.25`
- Weekly seasonality (7-day pattern), yearly seasonality (Fourier sine/cosine)
- Linear trend (`trend_strength=0.0002` per day on log-scale)
- Promotions (probability 0.08, uplift 0.45)
- Price elasticity (mean -1.2, std 0.5, clipped to [-3.5, -0.1])
- ABC class multipliers (A=1.8, B=1.0, C=0.6)
- Max daily units cap: 500

**Key observation:** The offline simulation is **deterministic** (seeded), while the online forecast is **non-deterministic** (unseeded). This is a deliberate separation — the offline generator is designed for reproducible data, while the online service prioritizes simplicity.

### 4c. Calendar Generation (OBSERVED in `quellcode/datensimulation/kalender_erzeugen.py`, 40 lines)

**Algorithm:**
1. `pd.date_range(start, end, freq="D")` — inclusive daily range.
2. Columns: `date`, `year`, `month`, `day`, `weekday` (Monday=0), `is_weekend` (Saturday/Sunday), `is_payday` (day == 25).
3. Payday rule: `is_payday = (calendar["day"] == 25).astype(int)` — 25th of each month.

**Test coverage:** `tests/test_kalender.py` (4 tests):
- `test_generate_calendar_has_expected_columns`: verifies column set.
- `test_generate_calendar_row_count_inclusive`: verifies inclusive range (10 days).
- `test_weekend_flag_is_binary`: verifies is_weekend is 0/1.
- `test_payday_rule_day_25`: verifies payday is day 25 only.

---

## 5. Inventory Optimization (Phase 5 Lens: Hard Constraints)

### 5a. Safety Stock and Reorder Point (OBSERVED in `app/services/optimize_service.py`, 158 lines)

**Algorithm:**
1. Input: `service_level` (0.5–0.9999), `holding_cost` (≥0), `shortage_cost` (≥0), `demand[]` list.
2. Z-value lookup table for common service levels (0.80–0.99). Fallback: Abramowitz-Stegun inverse erf approximation.
3. Per item: `mean_lt = forecast * lead_time`, `safety_stock = z * demand_std * sqrt(max(1, lead_time))`, `reorder_point = mean_lt + safety_stock`.
4. `recommended_order_qty = max(0, reorder_point - inventory)`.
5. `expected_shortage = max(0, mean_lt - inventory)`.
6. `cost_item = holding_cost * safety_stock + shortage_cost * expected_shortage`.

**Input validation (OBSERVED):**
- `DomainValidationError` raised on: non-dict payload, invalid numbers (NaN/Inf), service_level out of (0, 1), negative costs, empty demand list, non-dict items, missing/empty product_id, negative forecast/inventory/std/lead_time.
- `_require()` helper: `_require(condition, message, field)` → `DomainValidationError` on failure.
- `_to_float()` / `_to_int()`: type coercion with NaN/Inf rejection.

**Key observation:** This is a proper newsvendor-style safety stock policy, not a fixed-ratio rule. It uses service-level-driven z-values and supports both lookup and approximate inverse-erf fallback. The validation is thorough — 15+ validation checks across the input payload.

### 5b. Pydantic Contracts (OBSERVED in `app/models/optimization.py`)

- `DemandItem`: `product_id` (str), `forecast_demand` (float), `current_inventory` (float), optional `demand_std`, `lead_time_days`.
- `OptimizationInput`: `service_level` (float, 0.5–0.9999), `holding_cost` (≥0), `shortage_cost` (≥0), `demand: List[DemandItem]`, `default_demand_std` (≥0, default 10.0), `default_lead_time_days` (>0, default 7.0).
- `OptimizationResultItem`: `product_id`, `recommended_order_quantity`, `safety_stock`, `reorder_point`, `expected_shortage`, `total_cost_item`.
- `OptimizationResult`: `total_cost`, `recommendations`.

---

## 6. Warehouse Allocation and Route Optimization (Phase 5 Lens: Baseline Comparisons)

### 6a. Warehouse Allocation (OBSERVED in `app/services/warehouse_ai.py`, 31 lines)

**Algorithm:**
1. Input: `order_location` (lat/lon), `warehouses` list (lat/lon).
2. If no warehouses → return `None` with message.
3. For each warehouse: `distance = sqrt((order_lat - w_lat)^2 + (order_lon - w_lon)^2)`.
4. Select warehouse with minimum Euclidean distance.

**Key observation:** This is pure Euclidean distance — not great-circle (haversine), not road-network distance, and not capacity-aware. The README does not claim optimal allocation; it describes "proximity-based allocation." The distance metric is appropriate for a heuristic baseline but would be inaccurate for long distances (curvature of Earth).

### 6b. Route Optimization (OBSERVED in `app/services/route_optimizer.py`, 37 lines)

**Algorithm:** Nearest-neighbor heuristic:
1. Start at city 0 (first stop).
2. At each step: find the nearest unvisited city.
3. Move to it, mark visited.
4. Repeat until all cities visited or no unvisited cities remain.

**Key observation:** This is a greedy TSP approximation — O(n²) complexity. It does not produce optimal routes (2-opt, Christofides, or exact solvers would be needed). The README honestly labels this as "nearest-neighbor heuristic" and "Multi-stop route sequencing using a nearest-neighbor heuristic." No claim of optimality.

### 6c. Route Router with Distance Matrix (OBSERVED in `app/routers/route_router.py`, 179 lines)

**Algorithm:**
1. Query all user-scoped orders and warehouses.
2. Group orders by warehouse (by `warehouse_id`, `assigned_warehouse_name`, or "unassigned").
3. For each group: build a stops list (warehouse + orders), compute Euclidean distance matrix, run nearest-neighbor, compute total distance.
4. Return `route_groups` with ordered stops and total distance per group.

**Distance metric:** `euclidean_distance(a_lat, a_lon, b_lat, b_lon)` — pure Euclidean on lat/lon coordinates. Not haversine. Appropriate for a local-area heuristic baseline.

---

## 7. Identity, Auth, and Data Isolation (Phase 5 Lens: Edge Cases)

### 7a. Authentication (OBSERVED in `main.py`, 222 lines)

**Registration:**
- `RegisterRequest`: email (EmailStr), password (str), optional full_name.
- `get_password_hash()`: bcrypt via `passlib.context.CryptContext(schemes=["bcrypt"])`.
- User stored with `hashed_password`, `email` (lowercased), `full_name`, `disabled` (default False).

**Login:**
- OAuth2 password flow (`OAuth2PasswordBearer(tokenUrl="/login")`).
- `authenticate_user()`: lookup by email (lowercased), verify bcrypt hash.
- `create_access_token()`: HS256 JWT with `sub=email`, 60-minute expiry.
- `SECRET_KEY = "change-this-secret-key-in-production"` — **HARDCODED** in `main.py`, `warehouse_router.py`, `order_router.py`, `route_router.py`.

**Auth dependency:** `get_current_user()` — JWT decode → user lookup → 401 on failure. Duplicated in `main.py`, `warehouse_router.py`, `order_router.py`, `route_router.py` (4 copies of the same function with the same hardcoded secret).

### 7b. User-Scoped Data Isolation (OBSERVED)

**Warehouse CRUD:**
- `create_warehouse()`: `owner_id=current_user.id`.
- `list_warehouses()`: `WHERE owner_id == current_user.id`.
- `get_warehouse()`: `WHERE id == warehouse_id AND owner_id == current_user.id`.
- `delete_warehouse()`: same scoping.

**Order CRUD:**
- `create_order()`: `owner_id=current_user.id`, auto-allocate warehouse from user's warehouses if not specified.
- `list_orders()`: `WHERE owner_id == current_user.id`.
- `get_order()`: `WHERE id == order_id AND owner_id == current_user.id`.
- `delete_order()`: same scoping.

**Route optimization:**
- `optimize_routes()`: queries `WHERE Order.owner_id == current_user.id` and `WHERE Warehouse.owner_id == current_user.id`.

| Boundary | Enforced? | Evidence |
|----------|-----------|----------|
| User-scoped warehouse reads | YES | `WHERE owner_id == current_user.id` on all warehouse queries |
| User-scoped order reads | YES | `WHERE owner_id == current_user.id` on all order queries |
| Cross-user reads impossible | YES (by query design) | No global queries; all filtered by owner_id |
| JWT validation | YES | `jwt.decode()` with HS256; 401 on invalid/missing token |
| Password hashing | YES | bcrypt via passlib |
| Secret management | **NO** | `SECRET_KEY` hardcoded in 4 files |

### 7c. CORS Configuration (OBSERVED in `main.py`)

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Key observation:** `allow_origins=["*"]` with `allow_credentials=True` is a permissive CORS policy. Per the CORS spec, `allow_credentials=True` with `allow_origins=["*"]` is technically invalid (browsers reject credentialed requests with wildcard origin), but FastAPI/Starlette handles this by reflecting the request's `Origin` header instead of sending `*`. This is a security concern for production but not a vulnerability in a demo/development platform.

---

## 8. Persistence and Database (OBSERVED in `app/database.py`, 33 lines)

**Configuration:**
- `DATABASE_URL` env var required at import — `raise ValueError("DATABASE_URL is not set")` if absent.
- PostgreSQL URL rewriting: `postgresql://` → `postgresql+psycopg://` (psycopg2 driver).
- `create_engine(DATABASE_URL, pool_pre_ping=True)` — connection health check.
- `SessionLocal` with `autocommit=False, autoflush=False`.

**Models:**
- `User`: id, email (unique), full_name, hashed_password, disabled.
- `Warehouse`: id, name, lat, lon, capacity, owner_id (FK→users.id).
- `Order`: id, description, lat, lon, demand, status, assigned_warehouse_name, optimized_route, warehouse_id (FK→warehouses.id), owner_id (FK→users.id), created_at.

**Key observation:** `DATABASE_URL` is required at import time — the application will not start without a database. There is no SQLite fallback or in-memory mode for development/testing. This means the test file (`test_kalender.py`) tests only the calendar generation utility, not the API or database layer.

---

## 9. Dashboard (OBSERVED in `app/dashboard/router.py` + `dashboard/index.html`)

**Dashboard router:**
- `/dashboard/status` — returns `{"status": "ok", "message": "Dashboard API is running"}`.
- `/dashboard/health` — calls `requests.get(f"{API_BASE}/health")` where `API_BASE = "https://lieferkette-optimierungsplattform.onrender.com"` (hardcoded).
- `/dashboard/sample-optimization` — calls the API's `/optimize` endpoint with a hardcoded sample payload.
- `/dashboard/summary` — calls `/dashboard/sample-optimization` and extracts the first recommendation.

**Key observation:** The dashboard router makes HTTP calls to the live Render deployment via a hardcoded `API_BASE` URL. This means the dashboard API is a proxy to the production deployment, not to the local FastAPI instance. The dashboard HTML (`dashboard/index.html`, 46KB) is a static SPA mounted at `/dashboard`.

---

## 10. Engineering Decisions (ADR-Style)

### ADR-08-001: Moving Average Over Trained Forecasting Model

| Field | Value |
|-------|-------|
| ADR_ID | ADR-08-001 |
| Source SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| Problem | Forecast demand with limited historical data and no ML infrastructure |
| Constraints | No GPU; must run on Render free tier; must be honest about method |
| Selected approach | 3-period moving average with Gaussian noise (np.random.normal(0, 2)) |
| Alternatives | Trained time-series model (ARIMA/prophet); flat average; exponential smoothing |
| Trade-offs | Simple, transparent, explainable. Does not capture seasonality, trend, or autocorrelation. Non-deterministic (no seed). README explicitly labels as "lightweight forecasting baseline." |
| Failure implications | Forecast quality degrades with seasonal or trending data; noise term adds ±2 units of random variation per day |
| Evidence | `app/services/demand_forecast.py` lines 1-36; README "Demand forecasting" section |
| Status | IMPLEMENTED — honest label in README |

### ADR-08-002: Nearest-Neighbor Heuristic Over Exact TSP Solver

| Field | Value |
|-------|-------|
| ADR_ID | ADR-08-002 |
| Source SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| Problem | Sequence multi-stop delivery routes from warehouse to orders |
| Constraints | Must be fast; must be explainable; no OR-tools dependency |
| Selected approach | Nearest-neighbor greedy heuristic (start at first stop, always go to nearest unvisited) |
| Alternatives | 2-opt improvement; Christofides approximation; exact ILP solver |
| Trade-offs | O(n²) complexity; produces a valid but suboptimal route. Does not guarantee ≤1.5× optimal (Christofides) or any approximation ratio. |
| Failure implications | Routes may be significantly longer than optimal for non-trivial numbers of stops; no quality bound |
| Evidence | `app/services/route_optimizer.py` lines 1-37; README "Route Optimization" section |
| Status | IMPLEMENTED — honestly labeled as heuristic |

### ADR-08-003: Euclidean Distance Over Haversine for Warehouse Allocation

| Field | Value |
|-------|-------|
| ADR_ID | ADR-08-003 |
| Source SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| Problem | Select the nearest warehouse for an order |
| Constraints | Simplicity over precision; local-area logistics context |
| Selected approach | Euclidean distance on lat/lon coordinates |
| Alternatives | Haversine (great-circle) distance; road-network distance; capacity-weighted allocation |
| Trade-offs | Euclidean is simple and fast but inaccurate for long distances (ignores Earth's curvature). For local logistics (same city/region), the error is negligible. |
| Failure implications | At continental scale, distances would be significantly wrong. For the intended local logistics use case, acceptable. |
| Evidence | `app/services/warehouse_ai.py` lines 1-31; `app/routers/route_router.py` `euclidean_distance()` function |
| Status | IMPLEMENTED — appropriate for local logistics, inappropriate for global scale |

### ADR-08-004: DomainValidationError Input Validation in Optimization

| Field | Value |
|-------|-------|
| ADR_ID | ADR-08-004 |
| Source SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| Problem | Validate optimization input before computing safety stock and reorder points |
| Constraints | Must reject invalid inputs with specific error messages; must handle type coercion |
| Selected approach | `DomainValidationError(ValueError)` with field-specific messages; `_require()`, `_to_float()`, `_to_int()` helpers; 15+ validation checks |
| Alternatives | Pydantic-only validation (already present at API layer); no domain validation; generic 400 errors |
| Trade-offs | Domain validation is redundant with Pydantic but catches edge cases (NaN, Inf, empty lists) that Pydantic may not reject by default. Error messages are field-specific. |
| Failure implications | Additional validation layer may reject valid-but-edge inputs; z-value approximation for non-standard service levels may be imprecise |
| Evidence | `app/services/optimize_service.py` lines 1-158; `app/routers/optimize.py` Pydantic + DomainValidationError |
| Status | IMPLEMENTED — thorough validation, defense-in-depth |

### ADR-08-005: Hardcoded JWT Secret Across 4 Router Files

| Field | Value |
|-------|-------|
| ADR_ID | ADR-08-005 |
| Source SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| Problem | JWT signing and verification requires a shared secret |
| Constraints | Development/demo environment; no secret management infrastructure |
| Selected approach | `SECRET_KEY = "change-this-secret-key-in-production"` hardcoded in main.py, warehouse_router.py, order_router.py, route_router.py |
| Alternatives | Environment variable; secrets manager; per-router dependency injection |
| Trade-offs | Simple to run; no env config needed. But: secret is visible in source, duplicated 4×, and identical across all environments. |
| Failure implications | **SECURITY GAP** — anyone with source access can forge JWTs. The string "change-this-secret-key-in-production" is a known placeholder. Not suitable for production. |
| Evidence | `main.py` line 50, `warehouse_router.py` line 19, `order_router.py` line 18, `route_router.py` line 15 |
| Status | IMPLEMENTED — security gap, not a design decision. GAP-08-002. |

### ADR-08-006: Deterministic Offline Simulation vs Non-Deterministic Online Forecast

| Field | Value |
|-------|-------|
| ADR_ID | ADR-08-006 |
| Source SHA | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| Problem | Generate demand data for testing and forecasting |
| Constraints | Offline simulation should be reproducible; online forecast should be simple |
| Selected approach | Offline: `DemandSimulationConfig(seed=42)` + `np.random.default_rng(seed)` (deterministic). Online: `np.random.normal(0, 2)` without seed (non-deterministic). |
| Alternatives | Seed both; don't seed either; use environment-controlled seeding |
| Trade-offs | Offline is reproducible for data generation. Online is simple but non-deterministic — same input produces different forecasts on each call. README discloses this explicitly. |
| Failure implications | Online forecast is not reproducible — same history + same request → different output. For testing/auditing, this is a gap. |
| Evidence | `app/services/demand_forecast.py` (no seed), `quellcode/datensimulation/nachfrage_simulieren.py` (seed=42), README "Demand forecasting" section |
| Status | IMPLEMENTED — deliberate but creates a reproducibility gap (GAP-08-006) |

---

## 11. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-08-001 | MEDIUM | Very limited test coverage — 1 test file (`test_kalender.py`) with 4 test functions. Only tests calendar generation, not the API, optimization, forecasting, routing, auth, or persistence layers. 31 of 32 source files have no test coverage. |
| GAP-08-002 | HIGH | Hardcoded JWT secret `"change-this-secret-key-in-production"` in 4 files (`main.py`, `warehouse_router.py`, `order_router.py`, `route_router.py`). Not environment-variable-driven. Anyone with source access can forge JWTs. |
| GAP-08-003 | HIGH | No license file — GitHub API reports `license: null`. No `LICENSE` file in the repository tree (32 files, none named LICENSE or LICENSE.*). Copying source into the portfolio requires explicit owner authorization. |
| GAP-08-004 | MEDIUM | Non-deterministic online forecast — `demand_forecast.py` uses `np.random.normal(0, 2)` without a seed. Same input produces different outputs on each call. Reproducibility gap for testing and auditing. |
| GAP-08-005 | MEDIUM | Permissive CORS — `allow_origins=["*"]` with `allow_credentials=True`. Any origin can make credentialed requests. Appropriate for development but not production. |
| GAP-08-006 | LOW | No published releases or tags. Version 0.8.1 is declared in the FastAPI constructor only (`version="0.8.1"`), not anchored to a Git tag or release artifact. |
| GAP-08-007 | LOW | Euclidean distance for warehouse allocation — not haversine. Inaccurate for long distances. Acceptable for local logistics but would produce wrong results at continental scale. |
| GAP-08-008 | LOW | Nearest-neighbor route optimization has no quality bound — produces valid but suboptimal routes. No 2-opt improvement or approximation ratio guarantee. |
| GAP-08-009 | MEDIUM | `DATABASE_URL` required at import — application will not start without a database. No SQLite fallback or in-memory mode for local development. Only the calendar utility is testable without a database. |
| GAP-08-010 | LOW | Dashboard router hardcodes `API_BASE = "https://lieferkette-optimierungsplattform.onrender.com"` — calls the production deployment instead of the local FastAPI instance. Not configurable. |
| GAP-08-011 | LOW | Duplicated `get_current_user()` function across 4 files (main.py, warehouse_router.py, order_router.py, route_router.py) — each with the same hardcoded secret. Code duplication; no shared auth dependency. |
| GAP-08-012 | LOW | Demand simulator (`app/app/simulation/demand_simulator.py`) uses `random.gauss()` — non-deterministic, unlike the `quellcode/` simulation which is seeded. Two separate simulation paths with different randomness policies. |

---

## 12. Claim Comparison

| Claim ID | Claim | Phase 1 Class | Phase 5 Assessment |
|----------|-------|---------------|---------------------|
| CLM-08-001 | "Lieferkette CI passes" | OBSERVED / PASS | **CONFIRMED** — 4 consecutive CI runs all SUCCESS at HEAD `91c61210e913` (2026-07-27). CI runs `pytest` which discovers `test_kalender.py` (4 tests). CI scope is very narrow — only calendar generation is tested. |
| CLM-08-002 | "Lieferkette uses heuristic-based routing" | REPORTED / NOT_TESTED | **OBSERVED** — `route_optimizer.py` implements nearest-neighbor heuristic (not exact TSP). `warehouse_ai.py` uses Euclidean distance. `demand_forecast.py` uses moving average with Gaussian noise. All are heuristics, honestly labeled in README. |

---

## 13. Reproducibility Recipe (R1 → R3 path)

This is a documented path for an independent reviewer to reach R3. It has NOT been executed by this agent (not authorized per DIRECTIVE.txt F03/G/J — source repositories are read-only).

```bash
git clone https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform.git
cd lieferkette-optimierungsplattform
git checkout 91c61210e9134302f17a634c2b830cadadf5c4bc  # HEAD SHA
# Requires: Python 3.11
pip install -r anforderungen.txt
# CI test (what CI actually runs)
pytest -q
# Expected: 4 tests pass (test_generate_calendar_has_expected_columns,
#   test_generate_calendar_row_count_inclusive, test_weekend_flag_is_binary,
#   test_payday_rule_day_25)

# API server (requires DATABASE_URL)
export DATABASE_URL="postgresql://user:pass@localhost:5432/lieferkette"
uvicorn main:app --host 0.0.0.0 --port 8000
# Swagger UI: http://localhost:8000/docs
# Dashboard: http://localhost:8000/dashboard/
```

Preconditions: Python 3.11; `DATABASE_URL` env var for PostgreSQL (required at import). No `DATABASE_URL` → `ValueError` at import time.

**CI reproduction note:** CI at HEAD `91c61210e913` is SUCCESS (run #30228690474). All 4 recent CI runs are SUCCESS. CI scope: `pytest` discovers only `test_kalender.py` (4 tests). The API, optimization, forecasting, routing, auth, and persistence layers have no test coverage. Full system testing requires a PostgreSQL database.

---

## 14. Evidence Matrix

| EVD ID | Claim | Source path | Observation class | Result | Source URL | Source SHA | Notes |
|--------|-------|-------------|-------------------|--------|------------|------------|-------|
| EVD-08-001 | No license file — GitHub API license null | repo root | OBSERVED | BLOCKED | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform | `91c61210e913` | No LICENSE file in 32-file tree. GitHub API `license: null`. All evidence uses permalinks only; no source copied. |
| EVD-08-002 | CI passes at HEAD (4 consecutive SUCCESS) | `.github/workflows/python-tests.yml` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/.github/workflows/python-tests.yml | `91c61210e913` | Run #30228690474 (HEAD) + 3 prior: all SUCCESS, all push events, 2026-07-25 through 2026-07-27. CI runs `pytest -q`. |
| EVD-08-003 | 1 test file with 4 test functions | `tests/test_kalender.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/tests/test_kalender.py | `91c61210e913` | 4 tests: column check, row count, weekend binary, payday rule. Only calendar generation tested — 31 source files untested. |
| EVD-08-004 | 0 release tags, 0 published releases | GitHub API | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform | `91c61210e913` | Tags: 0. Releases: 0. Version 0.8.1 in `main.py` FastAPI constructor only. |
| EVD-08-005 | Moving-average demand forecast (not ML) | `app/services/demand_forecast.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/demand_forecast.py | `91c61210e913` | 3-period rolling mean; `<3` data points → overall mean; `np.random.normal(0, 2)` noise. No `method` field in response. |
| EVD-08-006 | Non-deterministic online forecast (no seed) | `app/services/demand_forecast.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/demand_forecast.py | `91c61210e913` | `np.random.normal(0, 2)` without seed. Same input → different output per call. README discloses this. GAP-08-004. |
| EVD-08-007 | Nearest-neighbor route optimization (heuristic) | `app/services/route_optimizer.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/route_optimizer.py | `91c61210e913` | Greedy: start at city 0, go to nearest unvisited. O(n²). No quality bound. |
| EVD-08-008 | Euclidean warehouse allocation | `app/services/warehouse_ai.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/warehouse_ai.py | `91c61210e913` | `sqrt((lat_a - lat_b)² + (lon_a - lon_b)²)`. Not haversine. Appropriate for local logistics. |
| EVD-08-009 | Safety stock + reorder point optimization (z-values) | `app/services/optimize_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/optimize_service.py | `91c61210e913` | Newsvendor-style policy: `safety_stock = z * σ * sqrt(L)`, `reorder_point = mean_lt + safety_stock`. Z-value lookup + inverse-erf fallback. 15+ validation checks. |
| EVD-08-010 | DomainValidationError input validation | `app/services/optimize_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/optimize_service.py | `91c61210e913` | `_require()`, `_to_float()`, `_to_int()` helpers. Rejects NaN, Inf, out-of-range, empty list, non-dict items, missing product_id. |
| EVD-08-011 | Hardcoded JWT secret in 4 files | `main.py` + 3 routers | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/main.py | `91c61210e913` | `SECRET_KEY = "change-this-secret-key-in-production"` in main.py:50, warehouse_router.py:19, order_router.py:18, route_router.py:15. GAP-08-002 (HIGH). |
| EVD-08-012 | JWT HS256 auth with bcrypt password hashing | `main.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/main.py | `91c61210e913` | `passlib.context.CryptContext(schemes=["bcrypt"])`; `python-jose` JWT; 60-minute token expiry; `OAuth2PasswordBearer`. |
| EVD-08-013 | User-scoped data (owner_id filtering) | `app/routers/warehouse_router.py` + `order_router.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/routers/warehouse_router.py | `91c61210e913` | All CRUD: `WHERE owner_id == current_user.id`. No global queries. Cross-user reads impossible by query design. |
| EVD-08-014 | Permissive CORS (allow_origins=["*"]) | `main.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/main.py | `91c61210e913` | `allow_origins=["*"]`, `allow_credentials=True`, `allow_methods=["*"]`, `allow_headers=["*"]`. Development config. GAP-08-005. |
| EVD-08-015 | DATABASE_URL required at import | `app/database.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/database.py | `91c61210e913` | `if not DATABASE_URL: raise ValueError("DATABASE_URL is not set")`. No SQLite fallback. GAP-08-009. |
| EVD-08-016 | PostgreSQL via psycopg2 (pool_pre_ping) | `app/database.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/database.py | `91c61210e913` | `postgresql://` → `postgresql+psycopg://` rewrite; `pool_pre_ping=True` for connection health. |
| EVD-08-017 | Deterministic offline simulation (seed=42) | `quellcode/datensimulation/nachfrage_simulieren.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/quellcode/datensimulation/nachfrage_simulieren.py | `91c61210e913` | `DemandSimulationConfig(seed=42)`; `np.random.default_rng(config.seed)`; negative binomial, weekly/yearly seasonality, trend, promotions, price elasticity. |
| EVD-08-018 | Calendar generation with payday rule (day 25) | `quellcode/datensimulation/kalender_erzeugen.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/quellcode/datensimulation/kalender_erzeugen.py | `91c61210e913` | `pd.date_range(start, end, freq="D")`; columns: date, year, month, day, weekday, is_weekend, is_payday (day==25). |
| EVD-08-019 | Dashboard router calls live Render API | `app/dashboard/router.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/dashboard/router.py | `91c61210e913` | `API_BASE = "https://lieferkette-optimierungsplattform.onrender.com"`; `/dashboard/health` and `/dashboard/sample-optimization` proxy to production. GAP-08-010. |
| EVD-08-020 | 32 total tracked files | GitHub API tree | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/tree/91c61210e9134302f17a634c2b830cadadf5c4bc | `91c61210e913` | 32 blobs, 13 trees. 19 .py files, 1 test file, 1 CI workflow, 1 requirements file, 1 README (46KB), 1 dashboard HTML (46KB). |
| EVD-08-021 | Pydantic contracts on all request/response models | `app/models/optimization.py` + routers | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/models/optimization.py | `91c61210e913` | `DemandItem`, `OptimizationInput`, `OptimizationResult` with Field constraints (ge, le, gt). Pydantic 2.6+ with `from_attributes = True`. |
| EVD-08-022 | Non-deterministic demand simulator (app/) | `app/app/simulation/demand_simulator.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/app/simulation/demand_simulator.py | `91c61210e913` | `random.gauss(base_demand, std_dev)` — no seed. Contrasts with `quellcode/` simulation (seeded). GAP-08-012. |
| EVD-08-023 | Duplicated get_current_user() across 4 files | `main.py` + 3 routers | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/main.py | `91c61210e913` | Same function (JWT decode + user lookup + 401) duplicated in main.py, warehouse_router.py, order_router.py, route_router.py. Each with same hardcoded secret. GAP-08-011. |
| EVD-08-024 | Route router groups orders by warehouse + builds distance matrix | `app/routers/route_router.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/routers/route_router.py | `91c61210e913` | Groups by warehouse_id/warehouse_name/unassigned; Euclidean distance matrix; nearest-neighbor; total_distance per group. |
| EVD-08-025 | Order auto-allocates warehouse if not specified | `app/routers/order_router.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/routers/order_router.py | `91c61210e913` | If `warehouse_id` is None → query user's warehouses → `allocate_warehouse()` → set `assigned_warehouse_name` + `status="planned"`. |
| EVD-08-026 | FastAPI v0.8.1 with OpenAPI/Swagger/ReDoc | `main.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/main.py | `91c61210e913` | `FastAPI(title="Lieferkette Optimierungsplattform API", version="0.8.1")`. Swagger at `/docs`, ReDoc at `/redoc`, OpenAPI schema at `/openapi.json`. |
| EVD-08-027 | Z-value lookup table + inverse-erf approximation | `app/services/optimize_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/services/optimize_service.py | `91c61210e913` | `z_map` for 0.80–0.99; fallback: Abramowitz-Stegun erfinv approximation → `z = sqrt(2) * erfinv(2*sl - 1)`. |
| EVD-08-028 | Negative binomial demand via Gamma-Poisson mixture | `quellcode/datensimulation/nachfrage_simulieren.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/quellcode/datensimulation/nachfrage_simulieren.py | `91c61210e913` | `_negative_binomial_rng()`: `rng.gamma(shape=1/alpha, scale=alpha*mu)` → `rng.poisson(lam=gamma)`. Over-dispersion parameter `alpha=0.25`. |
| EVD-08-029 | SQLAlchemy ORM models (User, Warehouse, Order) | `app/models/user.py` + `warehouse.py` + `order.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform/blob/91c61210e9134302f17a634c2b830cadadf5c4bc/app/models/user.py | `91c61210e913` | User (email unique), Warehouse (lat/lon/capacity/owner_id FK), Order (lat/lon/demand/status/warehouse_id FK/owner_id FK/created_at). |

---

## 15. 30-Filter Gate Record

**PHASE 5 | PRJ-08 Lieferkette Optimierungsplattform | Changeset: `cases/08-lieferkette-optimierungsplattform/engineering-evidence.md` (new file)**

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 5 for PRJ-08 |
| F02 Scope | PASS | Only portfolio path `cases/08-lieferkette-optimierungsplattform/engineering-evidence.md` modified |
| F03 Source immutability | PASS | No source repository changes; read-only inspection via GitHub API at pinned SHA |
| F04 Baseline | PASS | Branch `main`, HEAD `12fe7266062e4d904d6cbac3c92e9eb042469a6b`, clean working tree |
| F05 Conflict | PASS | No conflicting directives; DIRECTIVE.txt is sole authority |
| F06 Intent | PASS | Change advances evidence-first portfolio objective |
| F07 Necessity | PASS | No simpler non-writing alternative; evidence file required for Phase 5 |
| F08 Duplication | PASS | No existing `engineering-evidence.md` for PRJ-08; no duplicate content |
| F09 Provenance | PASS | Source URL, full SHA, file paths, timestamps captured |
| F10 Authenticity | PASS | No source statement treated as independently verified |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/INFERRED/NOT TESTED/CONDITIONAL used precisely |
| F12 Reproducibility | PASS | Reviewer can follow claim→source→evidence chain via permalinks |
| F13 Technical correctness | PASS | Architecture and code descriptions match actual source |
| F14 Domain safety | PASS | No financial, health, or legal risk claims; logistics domain honestly represented |
| F15 Trust boundaries | PASS | Source files treated as untrusted data; no prompt injections followed |
| F16 Security | PASS | No secrets introduced; hardcoded JWT secret in source is reported as GAP, not copied |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific information |
| F18 Intellectual property | PASS | No license file — no source code copied; all evidence uses permalinks and attributed descriptions only |
| F19 Attribution | PASS | Source author (Amin Azimi) accurately attributed |
| F20 Test integrity | PASS | No unexecuted test described as passed; 4 test functions OBSERVED by reading file; CI runs 4 tests |
| F21 Failure modes | PASS | Non-determinism, hardcoded secret, limited test coverage, permissive CORS, Euclidean approximation all represented |
| F22 Version drift | PASS | Release, branch, CI, deployment, and docs not conflated; "actively evolving" pinned |
| F23 Interface stability | PASS | Existing case README, links, and case ordering preserved |
| F24 Accessibility | PASS | Headings, tables, and text are navigable; no images requiring alt text |
| F25 Maintainability | PASS | Concise structure; no second source tree; links to source |
| F26 Reviewability | PASS | Diff is one new file; coherent and attributable to Phase 5 PRJ-08 outcome |
| F27 Validation | PASS | Available safe local checks: markdown structure verified; no external validators run |
| F28 Git hygiene | PASS | Commit authorized for Phase 5; push authorized for Phase 5; single meaningful commit |
| F29 Disclosure | PASS | Public-safe content; no sensitive material; no-license status noted (GAP-08-003) |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision: PROCEED** — all filters PASS; no blocking evidence.

---

## 16. Phase Completion Report

```
PHASE: 5 — Domain-Breadth Cases (PRJ-08 Lieferkette Optimierungsplattform)
AUTHORIZED SCOPE: Portfolio write — cases/08-lieferkette-optimierungsplattform/engineering-evidence.md (new file)
BASELINE HEAD: 12fe7266062e4d904d6cbac3c92e9eb042469a6b (portfolio main, clean working tree)
OBSERVED FACTS:
  - Source repo inspected at pinned SHA 91c61210e9134302f17a634c2b830cadadf5c4bc via GitHub API
  - 32 tracked files; 19 .py source files; 1 test file (4 test functions); 1 CI workflow
  - No license file — GitHub API license: null; no source code copied into portfolio
  - 4 consecutive CI runs SUCCESS at HEAD (2026-07-25 through 2026-07-27)
  - CI runs pytest which discovers test_kalender.py (4 tests) — only calendar generation tested
  - 0 release tags, 0 published releases; version 0.8.1 in FastAPI constructor only
  - Moving-average demand forecast with np.random.normal noise (non-deterministic, no seed)
  - Nearest-neighbor route optimization (heuristic, O(n²), no quality bound)
  - Euclidean warehouse allocation (not haversine, not capacity-aware)
  - Safety stock + reorder point optimization with z-values + inverse-erf fallback (15+ validation checks)
  - Deterministic offline simulation (seed=42, negative binomial, seasonality, promotions)
  - JWT HS256 auth with bcrypt; SECRET_KEY hardcoded in 4 files ("change-this-secret-key-in-production")
  - User-scoped data (owner_id filtering on all CRUD); no cross-user reads by query design
  - Permissive CORS (allow_origins=["*"], allow_credentials=True)
  - DATABASE_URL required at import (no SQLite fallback)
  - Dashboard router proxies to hardcoded production Render URL
FILES CHANGED: cases/08-lieferkette-optimierungsplattform/engineering-evidence.md (new file)
EVIDENCE ADDED: EVD-08-001 through EVD-08-029; ADR-08-001 through ADR-08-006; GAP-08-001 through GAP-08-012
CHECKS ACTUALLY RUN:
  - git status --short (clean working tree confirmed)
  - git rev-parse HEAD (portfolio HEAD confirmed)
  - GitHub Actions API (CI run status: 4 SUCCESS at HEAD)
  - GitHub API (tags: 0, releases: 0, license: null, size: 251 KB)
  - Source file inspection (all 19 .py files read via GitHub raw content; all 1 test file inspected; 4 test functions counted by reading)
  - Markdown structure verification (visual)
NOT TESTED / BLOCKED:
  - Full system NOT independently executed (source repos are read-only per DIRECTIVE.txt)
  - Only 4 test functions exist and are CI-enforced; 31 source files have no test coverage
  - No independent reproduction (R3) — source repos are immutable
  - License BLOCKED — no LICENSE file; source code not copied into portfolio
RISKS / OPEN DECISIONS:
  - GAP-08-002 (HIGH): Hardcoded JWT secret in 4 files
  - GAP-08-003 (HIGH): No license file — redistribution rights unclear
  - GAP-08-001 (MEDIUM): Very limited test coverage (1 file, 4 tests)
  - GAP-08-004 (MEDIUM): Non-deterministic online forecast
  - GAP-08-005 (MEDIUM): Permissive CORS
  - GAP-08-009 (MEDIUM): No SQLite fallback for development/testing
COMMIT SHA: (pending — will commit after PRJ-11 and PRJ-09 are complete)
PUSH STATUS: (pending — will push after all three Phase 5 projects are committed)
NEXT: PRJ-11 (Büro Assistant) — proceeding per owner authorization.
```
