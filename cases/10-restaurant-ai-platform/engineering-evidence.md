# PRJ-10 — Restaurant AI Platform: Engineering Evidence

**Project ID:** PRJ-10
**Canonical name:** Restaurant AI Platform
**Source:** https://github.com/aminbita162-glitch/restaurant-ai-platform
**Source HEAD (Phase 4 inspection):** `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` (2026-10-02T21:21:38Z)
**Default branch:** `main`
**License:** MIT — `LICENSE` file present, Copyright (c) 2026 Amin Azimi / Azimi Innovation Lab. SPDX-recognized MIT license confirmed at source inspection.
**Language:** Python 3.9+ (CI uses 3.11)
**Framework:** Flask 3.0.0 + Gunicorn 21.2.0
**Phase 4 inspection date (UTC):** 2026-10-17
**Reviewer:** Second-agent review (Hermes) — SECOND_AGENT_REVIEW; not an independent third-party review
**Independence level:** SECOND_AGENT_REVIEW — all findings are this agent's read-only source inspection via local clone at pinned SHA `c49649a0b44e`
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 partially achieved — CI run SUCCESS at HEAD `c49649a0b44e` observed via GitHub Actions API (run #37066364111, 2026-10-02). 5 consecutive CI runs all SUCCESS. The CI run artifact is linked but not independently reproduced by this reviewer.

---

## 1. Context and Scope

Restaurant AI Platform is a backend-first restaurant operations intelligence pipeline that transforms sales data into forecasting, staffing, inventory, waste, and AI-insight outputs via a modular Flask API. The system processes data through a 12-step pipeline orchestrator, persists pipeline runs per-tenant, and exposes a mobile manager approval page. It is explicitly a **development baseline** — not a hardened, fully-authenticated, or multi-tenant-isolated deployment.

**Status (OBSERVED in README.md at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| System status | "Development baseline" | OBSERVED |
| CI | Smoke CI — 5 consecutive SUCCESS runs at HEAD | OBSERVED |
| Test count | 43 test functions across 11 test files | OBSERVED |
| Release tags | 0 | OBSERVED |
| Published releases | 0 | OBSERVED |
| Frontend | Mobile manager HTML page (`/manager`) — not a customer SaaS | OBSERVED |
| Deployment | Render (demo) — `https://restaurant-ai-platform.onrender.com` | REPORTED |

**Directive-driven development (OBSERVED in `DIRECTIVE.txt` at HEAD):**

The source repository has its own directive-driven development workflow (`DIRECTIVE.txt`), locked on 2026-10-02. It specifies phases C0–C16 (C0–C14 complete, C15 demo install pack) and a separate R1–R9 phase map (commercial completion). The directive enforces hard locks: no invented `restaurant_001`/`location_001`, `method` fields must match actual algorithms, no side-effecting `GET /pipeline/run`, no production-ready/enterprise-ready/million-tenant claims, and honest labels throughout. Phase reports to owner in Persian/Farsi; code and commits in English.

**Non-goals (OBSERVED across README.md, DIRECTIVE.txt, and source comments):**

- Not production-ready, enterprise-ready, or million-tenant proven (README + test_r9_honesty.py enforce).
- Forecasting is heuristic (day-of-week buckets), not a trained ML model.
- Staffing is a fixed-ratio rule (1 per 600 sales), not a constraint optimizer.
- Inventory requires explicit recipe/BOM — no MRP or demand-driven optimization.
- Waste total comes from supplied `waste_events`; absent events return null, not a ratio estimate.
- API auth is `X-Api-Key` + optional `X-Tenant-Token` (env-mapped) — no full IdP, SSO, or RBAC.
- No customer-facing frontend dashboard.
- No live POS/ERP connectors.

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading all source files at HEAD SHA `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` via local clone. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
External Clients (untrusted)
  │  POST /api/v1/pipeline/run (auth: X-Api-Key + optional X-Tenant-Token)
  │  GET  /api/v1/pipeline/last-run?restaurant_id=...&location_id=...
  │  GET  /manager (mobile HTML page)
  │  POST /manager/approve (auth: X-Api-Key)
  │
  ▼
Flask App (app_factory.py → api_serving.py Blueprint)
  │  _check_auth(): 401/503 on missing/invalid X-Api-Key
  │  _resolve_tenant_token(): env TENANT_TOKENS → (restaurant_id, location_id)
  │  _check_rate_limit(): per-tenant sliding window (5 req/60s, in-process)
  │  enqueue_job(): writes to Postgres pipeline_jobs table (DATABASE_URL required)
  │
  ▼
Orchestrator (orchestrator.py)
  │  PIPELINE_ORDER: 12 steps, sequential, context dict shared
  │  run_step(): per-step handler, structured logging (run_id, step, duration_ms)
  │  _log_event(): every log line includes run_id + duration_ms
  │
  ├─ 1_data_ingestion     → data_ingestion.py (CSV file or demo=true)
  ├─ 2_data_warehouse     → data_warehouse.py (shapes records)
  ├─ 3_feature_engineering→ feature_engineering.py (derives features)
  ├─ 4_feature_store_sync→ feature_store_sync.py (syncs features)
  ├─ model_registry       → model_registry.py (registers model)
  ├─ 5_ml_prediction      → ml_prediction.py (dow_heuristic forecast + backtest gate)
  ├─ 6_optimization       → optimization.py (ratio rule shift planner)
  ├─ gpt_insight          → gpt_insight.py (OpenAI call + schema gate)
  ├─ 7_api_serving        → api_serving.py (status/health)
  ├─ 8_dashboard_update   → dashboard_update.py (BOM inventory + waste + manager payload)
  └─ 9_model_training     → model_training.py (model training)
  │
  ▼
Persistence (core/persistence.py)
  │  DATABASE_URL set  → Postgres (pipeline_runs table, tenant-scoped)
  │  DATABASE_URL unset → SQLite ephemeral (/tmp, dev/test only)
  │  get_last_run(): requires both restaurant_id + location_id (no cross-tenant)
  │
  ▼
Worker (worker.py)
  │  DATABASE_URL set  → claims jobs from Postgres (FOR UPDATE SKIP LOCKED)
  │  DATABASE_URL unset → returns 0, logs warning (not a cross-process substitute)
```

### 2b. Key Architectural Decisions

1. **Pipeline-first architecture** — All operational intelligence flows through a single ordered pipeline with a shared context dict. Each step reads upstream artifacts from `context[step_name]`, never re-invoking upstream modules (enforced after C4 — EVD-10-006).

2. **Tenant scoping everywhere** — `restaurant_id` + `location_id` are required on all pipeline runs and persistence reads. Empty strings are rejected with `MISSING_TENANT_KEYS` (400). No defaults invented. Persistence queries are tenant-scoped (`WHERE restaurant_id = ? AND location_id = ?`). Cross-tenant reads are impossible by query design.

3. **GET is side-effect free** — `GET /pipeline/run` returns instructions to use POST; it never creates or runs a job. Enforced by code (EVD-10-007) and tested (test_r2_worker.py).

4. **Async job execution** — `POST /pipeline/run` returns 202 + `job_id`; the pipeline is not run in the request process. A separate worker process claims jobs from Postgres. Without `DATABASE_URL`, `enqueue_job()` returns `QUEUE_NO_DATABASE` — the in-process queue is explicitly not a cross-process substitute.

5. **Honest method labels** — `method` fields must match the actual algorithm: `dow_heuristic` (not `ml_model`), `rule` (not `optimizer`), `bom_rule` (not `MRP`), `measured` (waste from events, not a ratio estimate). Enforced by directive hard locks and verified in source.

6. **AI output schema gate** — GPT insight output is schema-validated (7 required keys, 3 actions, non-empty strings). Invalid/missing → `degraded` with `method=rule` fallback. Raw model text never passes as the business action list.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Source URL | https://github.com/aminbita162-glitch/restaurant-ai-platform |
| HEAD SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| HEAD commit message | "docs: public service links" |
| HEAD commit date | 2026-10-02T21:21:38Z |
| Default branch | `main` |
| License | MIT (Copyright (c) 2026 Amin Azimi / Azimi Innovation Lab) |
| Total tracked files | 57 (excluding .git) |
| Source .py files (backend/) | 19 in restaurant_ai_platform/ + 2 in core/ |
| Test files | 11 (43 test functions total) |
| CI workflow | `.github/workflows/ci.yml` (Smoke CI) |
| CI runs at HEAD | 5 consecutive SUCCESS (all pushes to main since 2026-10-02) |
| CI run at HEAD | Run #37066364111 — SUCCESS — `c49649a0b44e` — 2026-10-02T21:21:41Z |
| Release tags | 0 |
| Published releases | 0 |
| Deployment | Render (demo) — `https://restaurant-ai-platform.onrender.com` |
| Inspection method | Local clone at pinned SHA + GitHub Actions API for CI metadata |
| Observation class | OBSERVED (all file content); OBSERVED (CI run status via API) |

**Commit history (git log --oneline -20 at HEAD):**

```
c49649a docs: public service links
7d2201c feat: postgres job queue for worker
ffe86b6 docs: manager page in capability matrix
9d9a56b docs: commercial install honesty
e73c5aa feat: manager approval page
8401891 feat: tenant from token
10f5d0a feat: measured waste events
437eaa3 feat: bom purchase from on-hand
54636ed feat: printable shift blocks
5219aab feat: sales row contract
2ffe9a9 feat: separate pipeline worker
92ce519 Update DIRECTIVE.txt
229b7a0 feat: live Postgres when DATABASE_URL set
7ffabf0 docs: demo install pack
2363d64 docs: honest capability matrix
2a39485 feat: manager decision payload
9b3f8d6 feat: structured run logs
13b28ab feat: ai output schema gate
9cdf03d feat: tighten tenant API gate
36fa138 feat: waste numeric contract
```

---

## 4. Pipeline Architecture and Step Decomposition

The pipeline is executed by `orchestrator.run_pipeline()` which iterates `PIPELINE_ORDER` (12 steps) sequentially, storing each step's result in a shared `context` dict under the step name. Downstream steps read upstream artifacts from `context` — never re-invoking upstream modules (enforced after C4).

### 4a. Pipeline Steps (OBSERVED in `orchestrator.py`)

| Order | Step key | Module | Handler | Role |
|-------|----------|--------|---------|------|
| 0 | `real_data_ingestion` | `real_data_ingestion` | `_run_real_data_ingestion` | Optional live-data ingestion |
| 1 | `1_data_ingestion` | `data_ingestion` | `_run_data_ingestion` | CSV sales file loader (tenant-scoped) |
| 2 | `2_data_warehouse` | `data_warehouse` | `_run_data_warehouse` | Shapes sales/inventory/attendance records |
| 3 | `3_feature_engineering` | `feature_engineering` | `_run_feature_engineering` | Derives features from records |
| 4 | `4_feature_store_sync` | `feature_store_sync` | `_run_feature_store_sync` | Syncs features for downstream |
| 5 | `model_registry` | `model_registry` | `_run_model_registry` | Registers the forecasting model |
| 6 | `5_ml_prediction` | `ml_prediction` | `_run_ml_prediction` | Day-of-week heuristic forecast + backtest |
| 7 | `6_optimization` | `optimization` | `_run_optimization` | Staffing ratio rule shift planner |
| 8 | `gpt_insight` | `gpt_insight` | `_run_gpt_insight` | OpenAI call + schema validation gate |
| 9 | `7_api_serving` | `api_serving` | `_run_api_serving` | API status |
| 10 | `8_dashboard_update` | `dashboard_update` | `_run_dashboard_update` | BOM inventory + waste + manager payload |
| 11 | `9_model_training` | `model_training` | `_run_model_training` | Model training |

### 4b. Context Flow (OBSERVED in `orchestrator.py` lines 620–700)

Each step's result is stored as `context[step_name] = step_data`. The orchestrator extracts key data from specific steps:
- After `1_data_ingestion`: extracts `restaurant_id`, `location_id`, `sales` from step data
- After `2_data_warehouse`: extracts `sales_records`, `inventory_records`, `attendance_records`
- After `3_feature_engineering`: extracts `generated_features`
- Ingestion errors (`error_code` present) stop the pipeline early with `stopped_early=True`

---

## 5. Data Ingestion and Sales Contract (Phase R3)

**OBSERVED in `data_ingestion.py` (260 lines):**

Data ingestion loads tenant-scoped CSV sales files. The contract enforces:

- **Required fields:** `date`, `net_sales`, `restaurant_id`, `location_id` — missing any returns `DATA_INGESTION_MISSING_REQUIRED_FIELD` with the field name.
- **`net_sales` must be numeric** — non-numeric returns `DATA_INGESTION_MISSING_REQUIRED_FIELD` with field `net_sales`.
- **Optional fields:** `channel`, `orders`, `guests`, `currency` — carried if present, ignored if absent.
- **File resolution:** Most specific first — `<restaurant_id>__<location_id>__sales.csv`, then `<restaurant_id>__sales.csv`. The shared `upload_sales.csv` is never used as a production source.
- **Demo mode:** `demo=True` must be set explicitly by the caller. It is the ONLY path to sample data. Never implicit.
- **Missing tenant keys:** Empty `restaurant_id` or `location_id` → `DATA_INGESTION_MISSING_TENANT_KEYS` error.

Error codes (OBSERVED): `DATA_INGESTION_FILE_NOT_FOUND`, `DATA_INGESTION_READ_FAILED`, `DATA_INGESTION_EMPTY`, `DATA_INGESTION_INVALID_ROW`, `DATA_INGESTION_MISSING_REQUIRED_FIELD`.

No silent synthetic fill — invalid or missing data returns a stable error code.

---

## 6. Forecast and Backtest Gate (Phase C5/C6)

### 6a. Day-of-Week Heuristic Forecast (OBSERVED in `ml_prediction.py`, 353 lines)

The forecast is a **heuristic** — not a trained ML model. Method is always `dow_heuristic`.

**Algorithm:**
1. Extract daily sales totals from context `sales` list (requires `daily_sales_total` numeric on each row).
2. Require ≥7 rows (`MIN_SALES_ROWS = 7`). Fewer → `FORECAST_INSUFFICIENT_DATA`.
3. Build per-weekday averages (`_build_dow_averages`): index 0=Mon … 6=Sun. When dates are parseable, uses weekday buckets. When no dates parseable, falls back to flat overall average — labeled `dow_method=flat_fallback` (still `dow_heuristic`, never `ml_model`).
4. Apply cumulative 2% daily growth (`DOW_GROWTH = 0.02`): `grown = base * (1.02 ^ (i+1))`.
5. Output: 7-day forecast with `day_index`, `date`, `weekday`, `predicted_sales` per day.

**Output fields:** `method=dow_heuristic`, `dow_method=dow_bucketed|flat_fallback`, `sales_rows_used`, `avg_daily_sales_used`, `horizon=7`, `forecast[]`, `backtest{}`, `labor_recommendation_allowed`, `mape_threshold=0.50`.

### 6b. Backtest Quality Gate (OBSERVED in `ml_prediction.py` `_backtest()`)

**Algorithm:**
1. Requires ≥14 rows (`MIN_BACKTEST_ROWS = 14`). Fewer → `labor_recommendation_allowed=false`, `backtest_reason=insufficient_history`.
2. Split: last 7 rows as test, preceding rows as train.
3. Re-run the day-of-week heuristic on the training set.
4. Forecast 7 days from the training set's last date.
5. Compute MAPE (Mean Absolute Percentage Error) — skips pairs where actual=0.
6. `labor_recommendation_allowed = mape <= MAPE_THRESHOLD (0.50)`.
7. When MAPE > 50%, downstream optimization returns `LABOR_RECOMMENDATION_DISALLOWED` with no headcount.

**Output fields:** `mape` (fraction), `mape_pct` (percentage), `backtest_rows`, `backtest_reason`, `labor_recommendation_allowed`.

This is a quality gate — not a claim of forecast accuracy. It gates labor recommendations behind a measured error threshold.

---

## 7. Staffing Optimization (Phase C7/R4)

**OBSERVED in `optimization.py` (209 lines):**

Staffing is a **fixed-ratio rule** — not a constraint optimizer. Method is always `rule`; approval is always `proposed`.

**Algorithm:**
1. Read forecast from `context["5_ml_prediction"]` — never re-invokes `ml_prediction.run()` (C4 rule).
2. Check `labor_recommendation_allowed` from the forecast artifact. If `false` → `LABOR_RECOMMENDATION_DISALLOWED` with no headcount.
3. Per forecast day: `day_total = max(1, int(predicted_sales / SALES_PER_STAFF))` where `SALES_PER_STAFF = 600`.
4. Distribute daily total across 6 two-hour blocks (`SHIFT_BLOCKS`: 10:00–12:00 through 20:00–22:00) via `_distribute_staff()` — integer division with remainder spread evenly. Sum of `n` across blocks equals the daily total; no headcount invented.
5. Roles: cycle through `[cook, floor, cashier]` (or caller-supplied role list) per block.
6. `estimated_labor_cost` included only when `hourly_cost` is supplied by the caller — never invented.

**Output fields:** `method=rule`, `approval=proposed`, `staffing_plan[]` (each block: `day_index`, `date`, `block_index`, `start`, `end`, `role`, `n`), `labor_recommendation_allowed`, `roles_used`.

---

## 8. Inventory and BOM (Phase C8/R5)

**OBSERVED in `dashboard_update.py` (531 lines):**

Inventory is a **BOM rule** — not a fixed-ratio MRP. Method is always `bom_rule`.

**Contract:**
1. `recipes`/`bom` must be present in context — each recipe has `name`, `expected_meals`, `ingredients[]` (each with `name`, `unit`, `quantity_per_meal`).
2. `on_hand` dict must be present — maps ingredient name → quantity on hand (≥0).
3. Missing BOM → `INVENTORY_MISSING_BOM`. Invalid BOM → `INVENTORY_INVALID_BOM`. Missing on_hand → `INVENTORY_MISSING_ON_HAND`.
4. Per ingredient: `total_quantity = quantity_per_meal * expected_meals`; `suggested_purchase = max(0, total_quantity - on_hand_qty)`. Never below zero.
5. No kg quantities invented — every value traces to explicit BOM entries and caller-supplied on_hand.

**Output fields:** `inventory_plan[]`, `inventory_method=bom_rule`, `inventory_method_description`.

---

## 9. Waste Events Contract (Phase C9/R6)

**OBSERVED in `dashboard_update.py` `_process_waste_events()`:**

Waste total comes from supplied `waste_events` — not a ratio estimate.

**Contract:**
- `waste_events` is a list of `{item, qty, reason, ts}`.
- When events present: `method=measured`, `waste_total = sum(qty)`.
- When events absent or empty: `method=null`, `waste_total=null`, `error_code=WASTE_NO_EVENTS_PROVIDED`. No kg invented from a sales ratio.
- Invalid events (non-dict, non-numeric qty) → `WASTE_INVALID_INPUT`.

**Validation gate:** `_validate_waste_input(sales)` checks sales rows carry numeric `net_sales`. Returns `WASTE_MISSING_INPUT` or `WASTE_INVALID_INPUT` on missing/invalid.

---

## 10. Tenant Isolation and Auth (Phase C10/R7)

### 10a. API Auth (OBSERVED in `api_serving.py`)

**Auth layers:**
1. `X-Api-Key` header (required on all protected routes):
   - If `RESTAURANT_AI_API_KEY` env var empty → 503 `AUTH_NOT_CONFIGURED`.
   - If missing/mismatch → 401 `AUTH_UNAUTHORIZED`.
   - Key value is never logged.
2. `X-Tenant-Token` header (optional, for tenant resolution):
   - Env `TENANT_TOKENS` format: `token:restaurant_id:location_id` pairs (comma-separated).
   - Token present → resolves `(restaurant_id, location_id)`. Body tenant ids are **ignored**.
   - Unknown token → 401 `AUTH_UNAUTHORIZED`.
3. Per-tenant rate limit: `_check_rate_limit()` — sliding window, 5 requests per 60 seconds per `restaurant_id:location_id`. Exceeded → 429 `RATE_LIMITED` with `Retry-After` header. In-process only (threading.Lock + dict).

**Tenant key enforcement:**
- `restaurant_id` and `location_id` are required on `POST /pipeline/run` and `GET /pipeline/last-run`.
- Empty strings → `MISSING_TENANT_KEYS` (400).
- No `DEFAULT_RESTAURANT_ID` or `DEFAULT_LOCATION_ID` constants — tenant ids are never defaulted.

### 10b. Persistence Tenant Scoping (OBSERVED in `core/persistence.py`)

- `get_last_run(restaurant_id, location_id)`: both required. If either absent → returns `PERSISTENCE_MISSING_TENANT_KEYS` error dict.
- Postgres: `WHERE restaurant_id = %s AND location_id = %s` — no cross-tenant reads possible by query design.
- SQLite: same WHERE clause. Tenant isolation is identical regardless of backend.
- `save_run(payload)`: stores with `restaurant_id` and `location_id` columns. Tenant A's run cannot be read by tenant B.

### 10c. Tenant Isolation Assessment

| Boundary | Enforced? | Evidence |
|----------|-----------|----------|
| API requires tenant keys | YES | `_collect_tenant_from_query()` / `_collect_tenant_from_json()` — empty → `MISSING_TENANT_KEYS` |
| Persistence is tenant-scoped | YES | `WHERE restaurant_id = ? AND location_id = ?` on both Postgres and SQLite |
| Cross-tenant reads impossible | YES (by query design) | No global query path; both keys always required |
| Token resolves to authoritative tenant | YES | Body tenant ignored when token present; unknown token → 401 |
| Rate limit is per-tenant | YES | `_rate_limit_key(restaurant_id, location_id)` — per-tenant sliding window |
| In-process state isolation | PARTIAL | Rate limiter and approval store use in-process dicts + threading.Lock — not cross-process isolated |

**INFERRED:** Tenant isolation is enforced at the application layer (API + persistence query scoping). There is no database-level RLS or per-tenant database cell. In-process state (rate limiter, approval store) is thread-safe but not cross-process isolated. The README honestly discloses this: "Multi-tenant isolation at scale: Planned — In-process state; not million-tenant isolated cells."

---

## 11. AI Output Schema Gate (Phase C11)

**OBSERVED in `gpt_insight.py` (325 lines):**

The GPT insight step calls OpenAI with a structured prompt and validates the response against a schema gate.

**Schema validation (`_validate_insight_schema`):**
- Required keys: `summary`, `staffing`, `inventory`, `waste`, `notes`, `risk_level`, `actions` (7 keys).
- `summary` and `risk_level` must be non-empty strings.
- `actions` must be a list of exactly 3 strings.
- Invalid (missing keys, wrong action count, non-string values, unparseable JSON) → `gpt_insight_status=degraded`, `method=rule`, with `_degraded_insight()` fallback (3 safe actions, `risk_level=unknown`).
- Valid → `method=openai`.
- Raw model text never passes as the business action list.

**OpenAI call (`_call_openai_json`):**
- Requires `OPENAI_API_KEY` env var. Absent → `skipped` with `reason=OPENAI_API_KEY_not_set`.
- Uses `response_format={"type": "json_object"}` for structured output.
- Model defaults to `gpt-3.5-turbo`; temperature defaults to 0.2.

---

## 12. Manager Approval Page (Phase C13/R8)

**OBSERVED in `api_serving.py` and `backend/static/manager.html`:**

- `GET /manager` serves a mobile HTML page from `backend/static/manager.html`.
- `POST /manager/approve` sets `approval=approved` for a `run_id` — only after confirmed POST + valid `X-Api-Key`.
- `POST /manager/approve` without auth → 401.
- Approval store is in-process (`_approval_store` dict).
- `GET /manager/approval-status?run_id=...` returns current approval status (defaults to `proposed`).

---

## 13. Structured Logging and Observability (Phase C12)

**OBSERVED in `orchestrator.py` `_log_event()` (lines 281–301):**

Every log line includes:
- `run_id=` — unique per pipeline run
- `duration_ms=` — elapsed time (0 for start events, real delta for done/error)
- `step=` — present on step-level events, absent on pipeline-level events

**Log events:** `step_start`, `step_done`, `step_skip`, `step_error`, `pipeline_start`, `pipeline_done`, `pipeline_dry_run`, `pipeline_error_validation`, `pipeline_stopped_ingestion_error`.

All 11 call sites in the orchestrator pass `duration_ms` as a first-class parameter to `_log_event()`. The test `test_structured_logs_include_run_id_step_duration_ms` verifies that every log line has `run_id=`, `duration_ms=`, and step-level lines have `step=`.

**Observability gap:** Structured logs exist but there is no alerting, tracing, or metrics dashboard (OBSERVED in README "Production observability: Planned").

---

## 14. Worker and Job Queue (Phase R2)

**OBSERVED in `worker.py` (85 lines) and `orchestrator.py` job queue helpers:**

- `enqueue_job(options)`: When `DATABASE_URL` is set, writes a job to the Postgres `pipeline_jobs` table and returns the `job_id`. When absent, returns `QUEUE_NO_DATABASE` error dict.
- `worker.run_once()`: When `DATABASE_URL` is set, atomically claims one queued job using `FOR UPDATE SKIP LOCKED` SQL. Executes the pipeline. Marks the job done or error. When `DATABASE_URL` is absent, returns 0 and logs a warning.
- The API process does not execute the pipeline — `POST /pipeline/run` only enqueues. The worker is a separate process.
- `GET /pipeline/run` never creates or runs a job — it returns instructions to use POST.

**Job queue schema (OBSERVED in `orchestrator.py` `_pg_init_jobs_table()`):**
- `pipeline_jobs` table: `id`, `job_id`, `status` (queued/running/done/error), `queued_at`, `started_at`, `ended_at`, `restaurant_id`, `location_id`, `options_json`, `result_json`, `error`.
- Index on `status` for efficient queue polling.

---

## 15. Engineering Decisions (ADR-Style)

### ADR-10-001: Day-of-Week Heuristic Over Trained ML Model

| Field | Value |
|-------|-------|
| ADR_ID | ADR-10-001 |
| Source SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| Problem | Forecast daily restaurant sales with limited historical data |
| Constraints | Small data (weeks of daily totals); no GPU; must run on Render free tier; must be honest about method |
| Selected approach | Day-of-week bucketed averages + 2% daily growth compounding |
| Alternatives | Trained time-series model (ARIMA/prophet); flat average |
| Trade-offs | Heuristic is transparent and explainable; does not claim ML; fails gracefully with `flat_fallback` when dates absent. Cannot capture seasonality beyond weekly patterns. |
| Failure implications | Over-simplification for multi-seasonal data; growth factor is arbitrary (2%), not data-driven |
| Evidence | `ml_prediction.py` lines 216–352; `method=dow_heuristic` always; `dow_method` distinguishes `dow_bucketed` vs `flat_fallback` |
| Status | IMPLEMENTED — honest label enforced by directive and code |

### ADR-10-002: Backtest Gate as Labor Quality Control

| Field | Value |
|-------|-------|
| ADR_ID | ADR-10-002 |
| Source SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| Problem | Prevent unreliable forecasts from driving staffing decisions |
| Constraints | Must be a quality gate, not a claim of accuracy; must degrade gracefully with insufficient data |
| Selected approach | Hold-out MAPE with 50% threshold; `labor_recommendation_allowed=false` when MAPE > 50% or < 14 rows |
| Alternatives | No gate (trust the heuristic); fixed threshold without backtest |
| Trade-offs | 50% MAPE is generous (50% error is large) but prevents catastrophically wrong staffing. Threshold is a policy choice, not optimized. |
| Failure implications | 50% threshold may allow noisy forecasts; backtest only covers 7 days; < 14 rows → no backtest at all |
| Evidence | `ml_prediction.py` `_backtest()` lines 148–213; `optimization.py` reads `labor_recommendation_allowed` and returns `LABOR_RECOMMENDATION_DISALLOWED` |
| Status | IMPLEMENTED — tested in test_r4_shift.py |

### ADR-10-003: BOM Rule Over Fixed-Ratio MRP

| Field | Value |
|-------|-------|
| ADR_ID | ADR-10-003 |
| Source SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| Problem | Calculate inventory purchase requirements from sales forecasts |
| Constraints | Must not invent kg; must require explicit recipe/BOM; must subtract on-hand |
| Selected approach | `suggested_purchase = max(0, quantity_per_meal * expected_meals - on_hand)` |
| Alternatives | Fixed ratio (X kg per Y sales); MRP with demand-driven optimization |
| Trade-offs | Requires caller to supply recipes and on_hand — no auto-inference. Transparent: every quantity traces to a BOM entry. |
| Failure implications | Missing BOM or on_hand → error, not a fallback; caller must supply both |
| Evidence | `dashboard_update.py` `_validate_bom()`, `_build_bom_inventory_plan()`; `inventory_method=bom_rule` |
| Status | IMPLEMENTED — tested in test_r5_bom.py |

### ADR-10-004: Waste as Measured Events, Not Ratio Estimate

| Field | Value |
|-------|-------|
| ADR_ID | ADR-10-004 |
| Source SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| Problem | Track food waste accurately |
| Constraints | Must not invent kg from a sales ratio; absent events must be null, not a guess |
| Selected approach | `waste_events` list with `{item, qty, reason, ts}`; `waste_total = sum(qty)` when present; `null` when absent |
| Alternatives | Waste = X% of sales (ratio estimate); waste model from historical data |
| Trade-offs | Requires caller to supply events — no auto-estimation. When absent, total is null (honest), not a ratio guess. |
| Failure implications | Absent events → no waste data; caller must supply events for meaningful waste tracking |
| Evidence | `dashboard_update.py` `_process_waste_events()`; `waste_method=measured|None`; `WASTE_NO_EVENTS_PROVIDED` error |
| Status | IMPLEMENTED — tested in test_r6_waste.py |

### ADR-10-005: Token-Resolved Tenant Over Body-Supplied Tenant

| Field | Value |
|-------|-------|
| ADR_ID | ADR-10-005 |
| Source SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| Problem | Resolve tenant identity securely from API requests |
| Constraints | Must not default tenant ids; body tenant must not override token tenant; unknown token → 401 |
| Selected approach | `X-Tenant-Token` header → env `TENANT_TOKENS` map → `(restaurant_id, location_id)`. Body tenant ignored when token present. |
| Alternatives | Body-only tenant; query-param-only tenant; JWT-based tenant |
| Trade-offs | Token map is env-based (no DB lookup); simple and testable. Body tenant ids are ignored when token present — prevents tenant spoofing. |
| Failure implications | Token map is in-process env; no token rotation; no tenant-level RBAC |
| Evidence | `api_serving.py` `_resolve_tenant_token()`, `_parse_tenant_tokens()`; tested in test_r7_tenant_token.py |
| Status | IMPLEMENTED |

### ADR-10-006: AI Output Schema Gate with Degraded Fallback

| Field | Value |
|-------|-------|
| ADR_ID | ADR-10-006 |
| Source SHA | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| Problem | Prevent unvalidated AI output from passing as business actions |
| Constraints | Must validate JSON schema; must fall back to safe rule-based output on failure; must never pass raw text as actions |
| Selected approach | 7-key schema validation + 3-action list validation; invalid → `degraded` with `_degraded_insight()` (safe actions, `risk_level=unknown`, `method=rule`) |
| Alternatives | No validation (trust the model); reject invalid output entirely (no fallback) |
| Trade-offs | Degraded fallback keeps the pipeline running with safe defaults. Schema is strict (7 keys, 3 actions) but simple. |
| Failure implications | Degraded output is static — no dynamic content; `risk_level=unknown` is honest |
| Evidence | `gpt_insight.py` `_validate_insight_schema()`, `_degraded_insight()`; `method=openai` on valid, `method=rule` on degraded |
| Status | IMPLEMENTED |

---

## 16. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-10-001 | MEDIUM | Smoke CI only runs `test_fail_closed.py` (4 of 43 tests). The remaining 39 test functions (test_r1–r9, test_g2) are NOT run in CI. CI scope is explicitly "smoke" — full test suite is not CI-enforced. |
| GAP-10-002 | MEDIUM | No published releases or tags. Version is implicit ("development baseline"). No `v1.0.0` or similar release artifact anchors the codebase to a versioned state. |
| GAP-10-003 | LOW | Forecast is a heuristic (day-of-week averages + 2% growth), not a trained model. The 2% growth factor is arbitrary, not data-driven. Backtest MAPE threshold is 50% — generous. |
| GAP-10-004 | LOW | Staffing is a fixed-ratio rule (1 per 600 sales), not a constraint optimizer. No shift-level demand modeling; no role-specific optimization. |
| GAP-10-005 | MEDIUM | No database-level tenant isolation (no RLS, no per-tenant database). Tenant scoping is application-layer only (WHERE clauses). In-process state (rate limiter, approval store) is thread-safe but not cross-process isolated. |
| GAP-10-006 | LOW | No production observability — structured logs exist but no alerting, tracing, or metrics dashboard. |
| GAP-10-007 | LOW | GPT insight quality depends on external model. `OPENAI_API_KEY` not set → skipped (not an error). When set, output is schema-validated with degraded fallback. |
| GAP-10-008 | MEDIUM | `_run_model_registry` (orchestrator.py line 757) still has `restaurant_id = str(context.get("restaurant_id") or "restaurant_001")` and `location_id = str(context.get("location_id") or "location_001")` — these default to invented tenant ids when context keys are absent. This contradicts the directive hard lock "do not invent restaurant_001 / location_001". However, this code path only executes when the `model_registry` step runs without upstream ingestion having populated context — which is an edge case not reached in the normal pipeline flow. |
| GAP-10-009 | LOW | `api_serving.py` `run()` function (line 663) still references `GET /pipeline/run?execute=1&confirm=yes` in `expected_endpoints` — a stale reference to the old side-effecting GET pattern that was explicitly disabled. Cosmetic only; the actual GET handler does not execute. |
| GAP-10-010 | LOW | No license-independent reproduction. CI uses `pip install Flask==3.0.0 pytest` — a minimal dependency set that does not include `openai`, `psycopg2`, or `python-dotenv`. CI tests only exercise the fail-closed path (no OpenAI, no Postgres). |

---

## 17. Claim Comparison

| Claim ID | Claim | Phase 1 Class | Phase 4 Assessment |
|----------|-------|---------------|---------------------|
| CLM-10-001 | "Phase-based delivery (Phase 1 baseline)" | REPORTED | **OBSERVED** — directive-driven development with C0–C14 complete, R1–R9 complete. README honestly states "Development baseline." Phase progression is real and source-pinned. |
| CLM-10-002 | "Smoke CI passes" | OBSERVED / PASS | **CONFIRMED** — 5 consecutive CI runs all SUCCESS at HEAD `c49649a0b44e` (2026-10-02). CI runs `test_fail_closed.py` (4 tests). Scope is "smoke" — 39/43 tests NOT run in CI. |

---

## 18. Reproducibility Recipe (R1 → R3 path)

This is a documented path for an independent reviewer to reach R3. It has NOT been executed by this agent (not authorized per DIRECTIVE.txt F03/G/J — source repositories are read-only).

```bash
git clone https://github.com/aminbita162-glitch/restaurant-ai-platform.git
cd restaurant-ai-platform
git checkout c49649a0b44e59d17407f11269dd0ec8cfc1d5fa  # HEAD SHA
# Requires: Python 3.9+ (CI uses 3.11)
pip install Flask==3.0.0 pytest
# CI smoke test (what CI actually runs)
python backend/tests/test_fail_closed.py
# OR: python -m pytest backend/tests/test_fail_closed.py -v
# Expected: 4 tests pass (test_ingestion_missing_file, test_last_run_missing_tenant_keys, test_forecast_under_7_rows, test_structured_logs)

# Full test suite (NOT run in CI — run locally for R3)
pip install openai psycopg2-binary python-dotenv gunicorn requests
cd backend
python -m pytest tests/ -v
# Expected: 43 test functions across 11 files — REPORTED (not independently executed by this reviewer)

# API server (demo mode)
gunicorn app:app --bind 0.0.0.0:8000
# In separate terminal:
python worker.py  # requires DATABASE_URL for live use; returns 0 without it
```

Preconditions: Python 3.9+; `RESTAURANT_AI_API_KEY` env var for API auth; `DATABASE_URL` for Postgres/worker; `OPENAI_API_KEY` for GPT insight (otherwise skipped).

**CI reproduction note:** CI at HEAD `c49649a0b44e` is SUCCESS (run #37066364111). All 5 recent CI runs are SUCCESS. CI scope is "smoke" — only 4 of 43 test functions are CI-enforced. Full test suite execution is NOT TESTED by this reviewer.

---

## 19. Evidence Matrix

| EVD ID | Claim | Source path | Observation class | Result | Source URL | Source SHA | Notes |
|--------|-------|-------------|-------------------|--------|------------|------------|-------|
| EVD-10-001 | MIT license confirmed | `LICENSE` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/LICENSE | `c49649a0b44e` | Copyright (c) 2026 Amin Azimi / Azimi Innovation Lab. Full MIT text. |
| EVD-10-002 | 12-step pipeline with shared context dict | `backend/restaurant_ai_platform/orchestrator.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/orchestrator.py | `c49649a0b44e` | `PIPELINE_ORDER` list, `run_step()` per-step handler, `context[step_name]` storage |
| EVD-10-003 | Day-of-week heuristic forecast (method=dow_heuristic) | `backend/restaurant_ai_platform/ml_prediction.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/ml_prediction.py | `c49649a0b44e` | `method` always `dow_heuristic`; `dow_method` distinguishes `dow_bucketed` vs `flat_fallback`; never `ml_model` |
| EVD-10-004 | Backtest quality gate with MAPE threshold | `backend/restaurant_ai_platform/ml_prediction.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/ml_prediction.py | `c49649a0b44e` | `_backtest()` splits last 7 rows as test; `MAPE_THRESHOLD=0.50`; `labor_recommendation_allowed` gates downstream staffing |
| EVD-10-005 | Staffing is a fixed-ratio rule (1 per 600 sales) | `backend/restaurant_ai_platform/optimization.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/optimization.py | `c49649a0b44e` | `SALES_PER_STAFF=600`; `method=rule`; `approval=proposed`; `_distribute_staff()` — sum of n equals daily total |
| EVD-10-006 | Single forecast artifact — no duplicate forecast | `backend/restaurant_ai_platform/optimization.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/optimization.py | `c49649a0b44e` | Reads `context["5_ml_prediction"]`; does NOT import or call `ml_prediction.run()`. Comment: "Do NOT import or rerun ml_prediction here." |
| EVD-10-007 | GET /pipeline/run has no side effects | `backend/restaurant_ai_platform/api_serving.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `pipeline_run_browser()` returns instructions only; never creates or runs a job. Tested in test_r2_worker.py. |
| EVD-10-008 | POST /pipeline/run returns 202 + job_id (async) | `backend/restaurant_ai_platform/api_serving.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `pipeline_run_post()` calls `orchestrator.enqueue_job(options)`; returns 202 with `status=queued` |
| EVD-10-009 | Worker claims jobs atomically (FOR UPDATE SKIP LOCKED) | `backend/restaurant_ai_platform/orchestrator.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/orchestrator.py | `c49649a0b44e` | `_pg_claim_next_job()` uses `FOR UPDATE SKIP LOCKED`; `worker.py` runs `run_once()` in a poll loop |
| EVD-10-010 | Tenant keys required — no defaults | `backend/restaurant_ai_platform/api_serving.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `_collect_tenant_from_query()` / `_collect_tenant_from_json()` — empty → `MISSING_TENANT_KEYS` (400). No `DEFAULT_RESTAURANT_ID` constant. |
| EVD-10-011 | Persistence is tenant-scoped (no cross-tenant reads) | `backend/restaurant_ai_platform/core/persistence.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/core/persistence.py | `c49649a0b44e` | `get_last_run(restaurant_id, location_id)` — both required; `WHERE restaurant_id = ? AND location_id = ?`; missing → `PERSISTENCE_MISSING_TENANT_KEYS` |
| EVD-10-012 | Token-resolved tenant overrides body tenant | `backend/restaurant_ai_platform/api_serving.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `_resolve_tenant_token()` → env `TENANT_TOKENS`; body tenant ignored when token present; unknown → 401 |
| EVD-10-013 | Per-tenant rate limit (5 req/60s) | `backend/restaurant_ai_platform/api_serving.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `_check_rate_limit()` — sliding window, threading.Lock, `_RATE_LIMIT_MAX_REQUESTS=5`, `_RATE_LIMIT_WINDOW_S=60`; exceeded → 429 with `Retry-After` |
| EVD-10-014 | BOM inventory with on-hand subtraction | `backend/restaurant_ai_platform/dashboard_update.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/dashboard_update.py | `c49649a0b44e` | `_validate_bom()` + `_build_bom_inventory_plan()` — `suggested_purchase = max(0, total_qty - on_hand)`; `inventory_method=bom_rule`; no MRP |
| EVD-10-015 | Waste as measured events (not ratio) | `backend/restaurant_ai_platform/dashboard_update.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/dashboard_update.py | `c49649a0b44e` | `_process_waste_events()` — `method=measured` when events present; `waste_total=null` when absent; `WASTE_NO_EVENTS_PROVIDED`; no kg invented |
| EVD-10-016 | AI output schema gate with degraded fallback | `backend/restaurant_ai_platform/gpt_insight.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/gpt_insight.py | `c49649a0b44e` | `_validate_insight_schema()` — 7 keys, 3 actions; invalid → `degraded`, `method=rule`; valid → `method=openai`; raw text never passes |
| EVD-10-017 | Structured logging (run_id, step, duration_ms) | `backend/restaurant_ai_platform/orchestrator.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/orchestrator.py | `c49649a0b44e` | `_log_event()` — first-class `duration_ms` param; all 11 call sites pass it; `run_id=` always present |
| EVD-10-018 | Manager approval page (mobile HTML) | `backend/restaurant_ai_platform/api_serving.py` + `backend/static/manager.html` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `GET /manager` serves HTML; `POST /manager/approve` requires auth; approval store in-process |
| EVD-10-019 | Data ingestion sales contract (required fields) | `backend/restaurant_ai_platform/data_ingestion.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/data_ingestion.py | `c49649a0b44e` | `_REQUIRED_ROW_FIELDS = (date, net_sales, restaurant_id, location_id)`; `_validate_row()`; missing → `DATA_INGESTION_MISSING_REQUIRED_FIELD`; demo=true is only demo path |
| EVD-10-020 | 43 test functions across 11 test files | `backend/tests/` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/tree/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/tests | `c49649a0b44e` | test_fail_closed(4), test_g2_worker_queue(3), test_r1_persistence(2), test_r2_worker(2), test_r3_sales_contract(6), test_r4_shift(7), test_r5_bom(5), test_r6_waste(5), test_r7_tenant_token(4), test_r8_manager(3), test_r9_honesty(2) |
| EVD-10-021 | CI runs only test_fail_closed.py (4 of 43 tests) | `.github/workflows/ci.yml` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/.github/workflows/ci.yml | `c49649a0b44e` | `pip install Flask==3.0.0 pytest`; `python backend/tests/test_fail_closed.py`. 39/43 tests NOT CI-enforced. |
| EVD-10-022 | 5 consecutive CI runs SUCCESS at HEAD | GitHub Actions API | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/actions | `c49649a0b44e` | Run #37066364111 (HEAD) + 4 prior: all SUCCESS, all push events, all 2026-10-02 |
| EVD-10-023 | 0 release tags, 0 published releases | GitHub API | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform | `c49649a0b44e` | Tags: 0. Releases: 0. Version is implicit ("development baseline"). |
| EVD-10-024 | README honestly discloses limitations | `README.md` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/README.md | `c49649a0b44e` | "Not production-ready, enterprise-ready, or million-tenant proven." Implemented/Experimental/Planned matrix. test_r9_honesty.py enforces no forbidden phrases. |
| EVD-10-025 | Postgres persistence when DATABASE_URL set | `backend/restaurant_ai_platform/core/persistence.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/core/persistence.py | `c49649a0b44e` | `_DATABASE_URL` evaluated at import; `save_run()` / `get_last_run()` route to Postgres or SQLite based on presence |
| EVD-10-026 | 2-hour shift blocks (6 per day, 10:00–22:00) | `backend/restaurant_ai_platform/optimization.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/optimization.py | `c49649a0b44e` | `SHIFT_BLOCKS` = 6 tuples; each block has `start`, `end`, `role`, `n`; sum of n equals daily total |
| EVD-10-027 | OpenAI call with JSON response format | `backend/restaurant_ai_platform/gpt_insight.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/gpt_insight.py | `c49649a0b44e` | `_call_openai_json()` — `response_format={"type": "json_object"}`; model defaults `gpt-3.5-turbo`; temp 0.2; skipped when `OPENAI_API_KEY` absent |
| EVD-10-028 | Manager payload (compact decision package) | `backend/restaurant_ai_platform/dashboard_update.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/dashboard_update.py | `c49649a0b44e` | `_build_manager_payload()` — forecast_summary, staffing_blocks, inventory_list, waste, actions, risk_level, `approval=proposed` |
| EVD-10-029 | _run_model_registry has invented tenant defaults | `backend/restaurant_ai_platform/orchestrator.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/orchestrator.py | `c49649a0b44e` | Line 757: `restaurant_id = str(context.get("restaurant_id") or "restaurant_001")` — contradicts directive hard lock. Edge case; not reached in normal pipeline flow. GAP-10-008. |
| EVD-10-030 | Expected_endpoints has stale GET execute reference | `backend/restaurant_ai_platform/api_serving.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/restaurant-ai-platform/blob/c49649a0b44e59d17407f11269dd0ec8cfc1d5fa/backend/restaurant_ai_platform/api_serving.py | `c49649a0b44e` | `run()` line 686: `"/pipeline/run?execute=1&confirm=yes (GET)"` — stale reference to disabled pattern. Cosmetic only. GAP-10-009. |

---

## 20. Phase 4 Filter Gate Record

**PHASE 4 | PRJ-10 Restaurant AI Platform | Changeset: `cases/10-restaurant-ai-platform/engineering-evidence.md` (new file)**

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 4 for PRJ-10 |
| F02 Scope | PASS | Only portfolio path `cases/10-restaurant-ai-platform/engineering-evidence.md` modified |
| F03 Source immutability | PASS | No source repository changes; read-only inspection only via local clone at pinned SHA |
| F04 Baseline | PASS | Branch `main`, HEAD `f1eed692672b5fd93345d4ad844e530fcd406320`, clean working tree |
| F05 Conflict | PASS | No conflicting directives; DIRECTIVE.txt is sole authority |
| F06 Intent | PASS | Change advances evidence-first portfolio objective |
| F07 Necessity | PASS | No simpler non-writing alternative; evidence file required for Phase 4 |
| F08 Duplication | PASS | No existing `engineering-evidence.md` for PRJ-10; no duplicate content |
| F09 Provenance | PASS | Source URL, full SHA, file paths, timestamps captured |
| F10 Authenticity | PASS | No source statement treated as independently verified |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/INFERRED/NOT TESTED/CONDITIONAL used precisely |
| F12 Reproducibility | PASS | Reviewer can follow claim→source→evidence chain via permalinks |
| F13 Technical correctness | PASS | Architecture and code descriptions match actual source |
| F14 Domain safety | PASS | No real-money, health, or legal risk claims; restaurant ops domain honestly represented |
| F15 Trust boundaries | PASS | Source files treated as untrusted data; no prompt injections followed |
| F16 Security | PASS | No secrets, tokens, or unsafe links introduced |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific information |
| F18 Intellectual property | PASS | MIT license permits copying; source descriptions are attributed; minimal excerpts used |
| F19 Attribution | PASS | Source author (Amin Azimi) accurately attributed |
| F20 Test integrity | PASS | No unexecuted test described as passed; 43 test count OBSERVED by counting functions; CI runs only 4/43 — documented as GAP-10-001 |
| F21 Failure modes | PASS | Negative cases, CI scope limitations, stale references, and edge-case defaults represented |
| F22 Version drift | PASS | Release, branch, CI, deployment, and docs not conflated; "development baseline" pinned |
| F23 Interface stability | PASS | Existing case README, links, and case ordering preserved |
| F24 Accessibility | PASS | Headings, tables, and text are navigable; no images requiring alt text |
| F25 Maintainability | PASS | Concise structure; no second source tree; links to source |
| F26 Reviewability | PASS | Diff is one new file; coherent and attributable to Phase 4 PRJ-10 outcome |
| F27 Validation | PASS | Available safe local checks: markdown structure verified; no external validators run |
| F28 Git hygiene | PASS | Commit authorized; push authorized; single meaningful commit |
| F29 Disclosure | PASS | Public-safe content; no sensitive material; MIT license noted |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision: PROCEED** — all filters PASS; no blocking evidence.

---

## 21. Phase Completion Report

```
PHASE: 4 — High-Risk Engineering Cases (PRJ-10 Restaurant AI Platform)
AUTHORIZED SCOPE: Portfolio write — cases/10-restaurant-ai-platform/engineering-evidence.md (new file)
BASELINE HEAD: f1eed692672b5fd93345d4ad844e530fcd406320 (portfolio main, clean working tree)
OBSERVED FACTS:
  - Source repo inspected at pinned SHA c49649a0b44e59d17407f11269dd0ec8cfc1d5fa via local clone
  - 57 tracked files; 19 source .py + 2 core/ .py; 11 test files with 43 test functions
  - MIT license confirmed (Copyright (c) 2026 Amin Azimi / Azimi Innovation Lab)
  - 5 consecutive CI runs SUCCESS at HEAD (Smoke CI, run #37066364111)
  - CI runs only test_fail_closed.py (4 of 43 tests) — 39 tests NOT CI-enforced
  - 0 release tags, 0 published releases
  - 12-step pipeline with shared context dict; honest method labels throughout
  - Tenant isolation: application-layer (WHERE clauses), no DB-level RLS
  - Backtest gate: MAPE ≤ 50% for labor recommendations; < 14 rows → disallowed
  - BOM inventory: explicit recipe + on_hand; no MRP; no invented kg
  - Waste: measured events only; absent → null, not a ratio estimate
  - AI output schema gate: 7 keys, 3 actions; degraded fallback with method=rule
  - GAP-10-008: _run_model_registry has "restaurant_001" default (edge case, contradicts directive)
  - GAP-10-009: stale "execute=1&confirm=yes" reference in expected_endpoints (cosmetic)
FILES CHANGED: cases/10-restaurant-ai-platform/engineering-evidence.md (new file — 640 lines)
EVIDENCE ADDED: EVD-10-001 through EVD-10-030; ADR-10-001 through ADR-10-006; GAP-10-001 through GAP-10-010
CHECKS ACTUALLY RUN:
  - git status --short (clean working tree confirmed)
  - git rev-parse HEAD (portfolio HEAD confirmed)
  - GitHub Actions API (CI run status: 5 SUCCESS at HEAD)
  - GitHub API (tags: 0, releases: 0)
  - Source file inspection (all 19+2 .py files read; all 11 test files inspected; test function count: 43)
  - Markdown structure verification (visual)
NOT TESTED / BLOCKED:
  - Full test suite NOT independently executed (source repos are read-only per DIRECTIVE.txt)
  - CI only runs 4/43 tests — 39 test functions NOT TESTED in CI
  - No independent reproduction (R3) — source repos are immutable
RISKS / OPEN DECISIONS:
  - GAP-10-001 (MEDIUM): CI scope is smoke-only; 39/43 tests not CI-enforced
  - GAP-10-008 (MEDIUM): _run_model_registry edge case with invented tenant default
  - GAP-10-005 (MEDIUM): No DB-level tenant isolation; application-layer only
  - GAP-10-002 (MEDIUM): No published releases or tags
COMMIT SHA: (pending owner authorization to commit)
PUSH STATUS: NOT AUTHORIZED — awaiting owner authorization
NEXT PHASE: NOT STARTED — AWAITING OWNER.
```

---

## 22. Stop Gate

Phase 4 (PRJ-10 Restaurant AI Platform) is complete. The engineering evidence file has been created with source-pinned observations, 30 evidence records, 6 ADRs, 10 known gaps, claim comparison, reproducibility recipe, and 30-filter gate record. All source repositories remain untouched.

**NEXT PHASE: NOT STARTED — AWAITING OWNER.**
