# PRJ-09 — Personal AI Assistant: Engineering Evidence

**Project ID:** PRJ-09
**Canonical name:** Personal AI Assistant
**Source:** https://github.com/aminbita162-glitch/personal-ai-assistant
**Source HEAD (Phase 5 inspection):** `455a3ebb4f84a6ca648d7a8481f864b21680f235` (2026-04-30T22:17:58Z, commit message: "Delete index.html")
**Default branch:** `main`
**License:** **NONE** — no `LICENSE` file detected; GitHub API reports `license: null`. Source code is not copied into the portfolio; links only. All evidence uses SHA-pinned permalinks and attributed descriptions; no source code blocks are pasted.
**Language:** Python 3.11.9
**Framework:** Flask 3.0.3 + psycopg2 2.9.9 + OpenAI 1.30.1 + bcrypt 4.1.2
**Phase 5 inspection date (UTC):** 2026-10-18
**Independence:** Source/documentation observations only; no independent third-party verification claimed.
**Independence level:** Source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 not achieved — no CI workflow, no test files, no test execution. All claims are source-level only.

---

## 1. Context and Scope

Personal AI Assistant is an MVP-stage personal productivity application that integrates task management, calendar appointments, reminders, and OpenAI-powered AI assistance. It is a Flask backend with PostgreSQL persistence, bcrypt password hashing, bearer-token authentication, and a static HTML frontend.

**Status (OBSERVED in README.md at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| System status | "🚧 In Development (MVP Phase)" | OBSERVED |
| CI | No CI workflow (0 `.github/workflows/` files) | OBSERVED |
| Test count | 0 test files, 0 test functions | OBSERVED |
| Release tags | 0 | OBSERVED |
| Published releases | 0 | OBSERVED |
| Deployment | None — no verified live URL; `DATABASE_URL` required at runtime | OBSERVED |
| Last pushed | 2026-04-30 (stale ~6 months at inspection date) | OBSERVED |
| Version | 1.0.0 (in `main.py` `/app-info` endpoint and `data/app_config.json`) | OBSERVED |

**Non-goals (OBSERVED across README.md):**

- Not a completed product — "🚧 In Development (MVP Phase)".
- Not a live hosted service — no verified live URL.
- Not a multi-user platform (despite having user accounts — the README says "Not a multi-user platform").
- Future plans include notifications, calendar integration, advanced AI planning, and a mobile app (Flutter) — all not implemented.
- Setup instructions: "Coming Soon" — no deployment documentation.

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading all 29 source files at HEAD SHA `455a3ebb4f84a6ca648d7a8481f864b21680f235` via local clone. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
External Clients (untrusted)
  │  POST /signup (email + password → bcrypt hash + auth_token)
  │  POST /login (email + password → bcrypt verify + new auth_token)
  │  GET  /me (Bearer token → user profile)
  │  POST /logout (Bearer token → auth_token = NULL)
  │  GET/POST/PUT/DELETE /tasks (Bearer token → user-scoped CRUD)
  │  GET/POST /appointments (Bearer token → user-scoped CRUD)
  │  GET /reminders (Bearer token → due-soon tasks + appointments)
  │  POST /ai (message → OpenAI → reply)
  │  POST /ai-to-task (message → OpenAI → extracted task → DB)
  │  POST /smart-ai (message → OpenAI → decide: task or reply)
  │  POST /transcribe-voice (audio file → OpenAI Whisper → transcript)
  │  GET /exchange-rates (frankfurter.app API)
  │  GET /health, /db-check, /init-db, /app-info
  │
  ▼
Flask Application (main.py, v1.0.0)
  │  CORS: Access-Control-Allow-Origin: "*" (permissive)
  │  Static files: templates/index.html, static/css, static/js
  │  get_connection(): psycopg2.connect(DATABASE_URL) — required at runtime
  │
  ├─ User Routes (routes/user_routes.py)
  │  signup: bcrypt.hashpw + secrets.token_hex(32) auth_token
  │  login: bcrypt.checkpw + new auth_token (rotation on each login)
  │  me: token lookup → user profile
  │  logout: SET auth_token = NULL WHERE auth_token = token
  │
  ├─ Task Routes (routes/task_routes.py)
  │  CRUD: user-scoped (WHERE user_id = %s)
  │  Auth: get_bearer_token() → get_authenticated_user() → user lookup by auth_token
  │  Validation: build_task_payload() with validate_task_title(), normalize_status/priority
  │
  ├─ Calendar Routes (routes/calendar_routes.py)
  │  CRUD: user-scoped appointments table (title, description, time, location, status)
  │
  ├─ Reminder Routes (routes/reminder_routes.py)
  │  GET /reminders: due-soon tasks + upcoming appointments (1-hour window)
  │
  ├─ AI Routes (routes/ai_routes.py)
  │  POST /ai: generate_ai_reply(message) → OpenAI chat.completions
  │  POST /ai-to-task: extract_task_from_message(message) → OpenAI → JSON → insert task
  │  POST /smart-ai: decide_smart_action(message) → OpenAI → "task" or "reply"
  │  POST /transcribe-voice: OpenAI audio.transcriptions.create (Whisper)
  │  GET /ai-browser: same as /ai but via query param (no auth check)
  │
  ├─ AI Service (services/ai_service.py)
  │  OpenAI client: os.getenv("OPENAI_API_KEY") → OpenAI(api_key)
  │  Chat model: os.getenv("OPENAI_CHAT_MODEL", "gpt-4o-mini")
  │  generate_ai_reply: system prompt + user message, temperature 0.7
  │  extract_task_from_message: task extraction prompt, JSON parsing, fallback
  │  decide_smart_action: "task" or "reply" decision, JSON parsing, fallback
  │  extract_first_json_object: brace-matching JSON parser (handles LLM wrapping)
  │
  ├─ Auth Service (services/auth_service.py)
  │  get_current_user: Bearer token → DB lookup → (user, error, status)
  │
  ▼
PostgreSQL (psycopg2)
  │  Tables: users (id, name, email, password, auth_token, created_at)
  │         tasks (id, title, description, status, priority, due_date, created_at, user_id)
  │         appointments (id, title, description, appointment_time, location, status, created_at, user_id)
  │  Schema: CREATE TABLE IF NOT EXISTS (runtime schema init)
  │  No migrations — schema is ensured at runtime via ensure_*_schema()
```

### 2b. Key Architectural Decisions

1. **Flask + psycopg2 + raw SQL** — No ORM (no SQLAlchemy); all SQL is hand-written with parameterized queries. `CREATE TABLE IF NOT EXISTS` is called on every request path (`ensure_tasks_schema`, `ensure_users_schema`, `ensure_appointments_schema`).

2. **Bearer token auth with DB lookup** — Auth tokens are `secrets.token_hex(32)` (256-bit random hex). Stored in the `users.auth_token` column. Looked up on every authenticated request. Token rotates on each login. Logout sets `auth_token = NULL`.

3. **OpenAI integration** — `OPENAI_API_KEY` env var → `OpenAI(api_key)`. Chat model defaults to `gpt-4o-mini`. Three AI endpoints: chat, task extraction, and smart decision (task vs. reply). Voice transcription via `gpt-4o-mini-transcribe`.

4. **User-scoped task/appointment queries** — All task and appointment CRUD filters by `user_id`. Cross-user reads are prevented by `WHERE user_id = %s` clauses.

5. **No test suite, no CI** — 0 test files, 0 test functions, 0 CI workflows. The README states "Setup (Coming Soon)" and "In Development (MVP Phase)."

6. **Permissive CORS** — `Access-Control-Allow-Origin: "*"` on all responses via `@app.after_request`.

7. **External API dependency** — `/exchange-rates` calls `https://api.frankfurter.app/latest` (European Central Bank rates). `/ai` calls OpenAI. Both are external dependencies with no caching, retry, or fallback.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Source URL | https://github.com/aminbita162-glitch/personal-ai-assistant |
| HEAD SHA | `455a3ebb4f84a6ca648d7a8481f864b21680f235` |
| HEAD commit message | "Delete index.html" |
| HEAD commit date | 2026-04-30T22:17:58Z |
| Default branch | `main` |
| License | **NONE** — no LICENSE file; GitHub API `license: null` |
| Total tracked files | 29 (excluding .git) |
| Source .py files | 14 (main.py + routes/ + services/ + utils/) |
| Test files | 0 |
| Test functions | 0 |
| CI workflow | **NONE** — 0 `.github/workflows/` files |
| CI runs | **NONE** — 0 CI runs |
| Release tags | 0 |
| Published releases | 0 |
| Version | 1.0.0 (`/app-info` endpoint + `data/app_config.json`) |
| Deployment | None — no verified live URL; `DATABASE_URL` required |
| Inspection method | Local clone at pinned SHA (after `git fetch aminbita/main`) |
| Observation class | OBSERVED (all file content); OBSERVED (no tests, no CI, no license) |

**Commit history (git log --oneline -15 at HEAD):**

```
455a3eb Delete index.html
53f63e7 Create index.html
1c88d25 fix: add newline at end of index.html
1270902 add frontend index file
f55e3b8 Update main.py
2eb9512 Update main.py
fd18df6 Update app.js
2038174 Update index.html
2b285a6 Update app.js
6ab4576 Update app.js
7fd684f Update index.html
c49c775 Update app.js
ce685c9 Update index.html
be3a7df Update index.html
18489e7 Update app.js
```

**Key observation:** The commit history shows sequential file-level edits ("Update main.py", "Update app.js", "Update index.html") — consistent with GitHub web editor edits, not a structured development workflow. The last commit deletes `index.html`, suggesting a cleanup or migration step.

---

## 4. AI Integration and Side Effects (Phase 5 Lens: Scopes, Approval Gates, Retries)

### 4a. OpenAI Integration (OBSERVED in `services/ai_service.py`, 225 lines)

**Three AI endpoints:**

| Endpoint | Function | Model | Temperature | Purpose |
|----------|----------|-------|-------------|---------|
| `POST /ai` | `generate_ai_reply()` | `gpt-4o-mini` | 0.7 | Chat reply |
| `POST /ai-to-task` | `extract_task_from_message()` | `gpt-4o-mini` | 0.1 | Extract task from natural language → JSON |
| `POST /smart-ai` | `decide_smart_action()` | `gpt-4o-mini` | 0.2 | Decide: create task or reply |

**Voice transcription:** `POST /transcribe-voice` — OpenAI `audio.transcriptions.create(model="gpt-4o-mini-transcribe")`. Audio file saved to `tempfile.NamedTemporaryFile(suffix=".webm")`, transcribed, then deleted in `finally` block.

**JSON parsing:** `extract_first_json_object()` — brace-matching parser that handles LLM response wrapping (text before/after JSON). Falls back to a default task if parsing fails.

**Multilingual instruction:** All prompts include `get_language_instruction()` — "You must ALWAYS detect the user's language from their message and reply in the SAME language. Support ALL languages."

### 4b. Side Effects Assessment

| AI endpoint | Side effect | Auth required? | Fallback on failure? |
|-------------|-------------|----------------|----------------------|
| `POST /ai` | None (returns text) | **NO** — no auth check | None — 500 on error |
| `POST /ai-to-task` | Inserts task into DB | YES — `get_current_user()` | Fallback: title=first 80 chars, priority=medium |
| `POST /smart-ai` | Inserts task or returns reply | YES — `get_current_user()` | Fallback: action=reply, calls `generate_ai_reply()` |
| `POST /transcribe-voice` | None (returns transcript) | YES — `get_current_user()` | None — 500 on error |
| `GET /ai-browser` | None (returns text) | **NO** — no auth check | None — 500 on error |
| `GET /ai-to-task-browser` | Inserts task into DB | YES — `get_current_user()` | Fallback |

**Key observation:** `POST /ai` and `GET /ai-browser` do NOT require authentication. Any unauthenticated user can call the OpenAI API through these endpoints, consuming the operator's OpenAI tokens. This is a cost-control and security gap.

### 4c. No Retries, No Rate Limiting, No Caching

- OpenAI calls have no retry logic — a failure returns HTTP 500.
- No rate limiting on any endpoint (no `slowapi`, no `flask-limiter`).
- No response caching — every call hits OpenAI.
- No token quota or budget enforcement — unlimited OpenAI spending possible.
- 10-second timeout on `requests.get` for exchange rates (frankfurter.app) but no timeout on OpenAI calls.

### 4d. Audit Trail

- **None.** There is no audit log, no event log, no decision record. AI calls, task creation, and user actions are not logged beyond Flask's default request logging.
- No `app/policy/` directory. No `app/agents/` directory. No governance layer.

---

## 5. Connector Permissions and Auth (Phase 5 Lens)

### 5a. Authentication (OBSERVED in `routes/user_routes.py` + `services/auth_service.py`)

**Signup:**
- Email (lowercased) + password → `bcrypt.hashpw(password, bcrypt.gensalt())` → stored as `password` column.
- `auth_token = secrets.token_hex(32)` — 256-bit random hex string.
- Duplicate email → 409 "User already exists".

**Login:**
- Email + password → `bcrypt.checkpw(password, user["password"])` → verify.
- On success: generate new `auth_token`, `UPDATE users SET auth_token = %s WHERE id = %s` — token rotates on each login.
- On failure: 401 "Invalid password".

**Auth middleware:**
- `get_bearer_token()`: extracts `Bearer <token>` from `Authorization` header.
- `get_current_user(get_connection)`: token → DB lookup → `(user, error, status)` or `(None, error_dict, 401)`.
- `get_authenticated_user(get_connection)` in `task_routes.py`: same pattern, different function name (duplicated).

### 5b. Auth Token Security

| Property | Value | Observation |
|----------|-------|--------------|
| Token generation | `secrets.token_hex(32)` (256-bit) | GOOD — cryptographically secure |
| Token storage | Plaintext in `users.auth_token` column | **GAP** — not hashed; DB compromise exposes all active tokens |
| Token rotation | New token on each login | GOOD — prevents token reuse |
| Token revocation | `logout()` sets `auth_token = NULL` | GOOD — explicit invalidation |
| Token in response | `/signup` and `/login` return `auth_token` in JSON | **GAP** — token visible in response body |
| No JWT | Tokens are opaque hex strings, not JWTs | Design choice — simpler, but no claims/expiry |

### 5c. User-Scoped Data Isolation (OBSERVED)

**Task CRUD:**
- `GET /tasks`: `WHERE user_id = %s` (authenticated user).
- `POST /tasks`: `user_id = authenticated_user["id"]`.
- `PUT /tasks/<id>`: `WHERE id = %s AND user_id = %s`.
- `DELETE /tasks/<id>`: `WHERE id = %s AND user_id = %s`.

**Appointment CRUD:**
- Same pattern: `WHERE user_id = %s` on all queries.

| Boundary | Enforced? | Evidence |
|----------|-----------|----------|
| User-scoped task reads | YES | `WHERE user_id = %s` on all task queries |
| User-scoped appointment reads | YES | `WHERE user_id = %s` on all appointment queries |
| Cross-user reads impossible | YES (by query design) | No global queries; all filtered by user_id |
| Auth on AI endpoints | **PARTIAL** | `/ai` and `/ai-browser` have no auth; `/ai-to-task`, `/smart-ai`, `/transcribe-voice` require auth |
| Auth on task/appointment CRUD | YES | All require `get_authenticated_user()` or `get_current_user()` |

---

## 6. Database and Schema (Phase 5 Lens)

### 6a. Runtime Schema Init (OBSERVED)

**No migrations** — schema is ensured at runtime via `CREATE TABLE IF NOT EXISTS` + `ALTER TABLE ADD COLUMN IF NOT EXISTS`:

| Function | Table | Columns |
|----------|-------|---------|
| `ensure_users_schema()` | `users` | id, name, email (UNIQUE), password, auth_token, created_at |
| `ensure_tasks_schema()` | `tasks` | id, title, description, status, priority, due_date, created_at, user_id |
| `ensure_appointments_schema()` | `appointments` | id, title, description, appointment_time, location, status, created_at, user_id |

**Key observation:** `ensure_*_schema()` is called on every request to the respective route. This is idempotent (`IF NOT EXISTS`) but adds a DB round-trip per request. No Alembic, no migration versioning.

### 6b. Connection Management (OBSERVED in `main.py`)

```python
def get_connection():
    database_url = os.getenv("DATABASE_URL")
    if not database_url:
        raise ValueError("DATABASE_URL is not set")
    return psycopg2.connect(database_url)
```

**Key observation:** `DATABASE_URL` is required at runtime (not import time). Each request opens a new connection (`psycopg2.connect()`), uses it, and closes it. No connection pool, no `pool_pre_ping`, no health check. Connection failures return 500.

### 6c. External API Dependency (OBSERVED in `main.py`)

`/exchange-rates` calls `https://api.frankfurter.app/latest?from=USD&to=EUR,GBP,CAD,AUD,JPY` with a 10-second timeout. No caching, no retry, no fallback. If the API is down → 500.

---

## 7. Engineering Decisions (ADR-Style)

### ADR-09-001: Flask + psycopg2 Over FastAPI + SQLAlchemy

| Field | Value |
|-------|-------|
| ADR_ID | ADR-09-001 |
| Source SHA | `455a3ebb4f84a6ca648d7a8481f864b21680f235` |
| Problem | Build a personal productivity API with task management, calendar, and AI |
| Constraints | MVP scope; solo developer; Python; simple deployment |
| Selected approach | Flask 3.0.3 + psycopg2 + raw SQL with parameterized queries |
| Alternatives | FastAPI + SQLAlchemy (used in other Azimi projects); Django; pure psycopg2 with dataclasses |
| Trade-offs | Flask is simple and well-known; raw SQL is transparent but requires manual schema management. No ORM means no migration framework, no model validation, no relationship loading. |
| Failure implications | Schema drift across deployments (no migration versioning); SQL injection risk if parameterization is missed (mitigated by `%s` placeholders everywhere) |
| Evidence | `main.py`, `routes/`, `services/` — all use psycopg2 directly with `cursor.execute(sql, params)` |
| Status | IMPLEMENTED — MVP approach; not production-hardened |

### ADR-09-002: Opaque Bearer Token Over JWT

| Field | Value |
|-------|-------|
| ADR_ID | ADR-09-002 |
| Source SHA | `455a3ebb4f84a6ca648d7a8481f864b21680f235` |
| Problem | Authenticate users for API access |
| Constraints | MVP; no JWT library dependency desired; simple token lookup |
| Selected approach | `secrets.token_hex(32)` stored as plaintext in `users.auth_token`; looked up on each request |
| Alternatives | JWT with claims and expiry; hashed tokens in DB; session table with TTL |
| Trade-offs | Simple — no JWT parsing, no expiry, no claims. But: token is stored in plaintext (DB compromise exposes all active tokens); no expiry (tokens persist until logout or new login rotation). |
| Failure implications | Stolen tokens are valid indefinitely until rotated by a new login. No automatic expiry. DB compromise = full account takeover. |
| Evidence | `routes/user_routes.py` `generate_auth_token()`, `auth_token` column in users table |
| Status | IMPLEMENTED — security gap (plaintext token storage, no expiry) |

### ADR-09-003: OpenAI as Primary AI Provider (No Fallback)

| Field | Value |
|-------|-------|
| ADR_ID | ADR-09-003 |
| Source SHA | `455a3ebb4f84a6ca648d7a8481f864b21680f235` |
| Problem | Provide AI chat, task extraction, and smart decision capabilities |
| Constraints | MVP; OpenAI is the most accessible LLM API; `gpt-4o-mini` is cost-effective |
| Selected approach | `OpenAI(api_key=os.getenv("OPENAI_API_KEY"))` with `gpt-4o-mini` default model |
| Alternatives | Local model (Ollama); multiple providers with fallback; Azure OpenAI; Anthropic |
| Trade-offs | OpenAI is simple and capable but creates a hard external dependency. No fallback, no retry, no rate limit, no budget enforcement. API key is consumed by any unauthenticated caller via `/ai` and `/ai-browser`. |
| Failure implications | OpenAI outage → all AI endpoints return 500. No degradation strategy. Cost is unbounded (no quota). |
| Evidence | `services/ai_service.py` `get_openai_client()`, `generate_ai_reply()`, `extract_task_from_message()`, `decide_smart_action()` |
| Status | IMPLEMENTED — MVP approach; no production safety controls |

---

## 8. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-09-001 | HIGH | No test suite — 0 test files, 0 test functions. No test infrastructure. The MVP has no automated test coverage. |
| GAP-09-002 | HIGH | No license file — GitHub API reports `license: null`. No LICENSE file in the 29-file tree. Source code is not copied into the portfolio; links only. |
| GAP-09-003 | HIGH | No CI workflow — 0 `.github/workflows/` files. No automated testing or deployment pipeline. |
| GAP-09-004 | MEDIUM | Unauthenticated AI endpoints — `POST /ai` and `GET /ai-browser` do not require authentication. Any unauthenticated user can consume the operator's OpenAPI tokens. Cost-control and security gap. |
| GAP-09-005 | MEDIUM | Auth token stored in plaintext — `users.auth_token` is a plaintext hex string. DB compromise exposes all active tokens. Not hashed (unlike PRJ-11 which uses SHA-256 hashed tokens). |
| GAP-09-006 | MEDIUM | No auth token expiry — tokens persist indefinitely until rotated by a new login or explicitly logged out. No TTL, no automatic expiration. |
| GAP-09-007 | MEDIUM | Permissive CORS — `Access-Control-Allow-Origin: "*"` on all responses. Any origin can make requests. |
| GAP-09-008 | MEDIUM | No rate limiting — no `slowapi`, no `flask-limiter`. Unlimited requests to all endpoints, including OpenAI-consuming AI endpoints. |
| GAP-09-009 | MEDIUM | No audit trail — no event log, no decision record, no audit table. AI calls and user actions are not tracked. |
| GAP-09-010 | LOW | No connection pooling — each request opens a new `psycopg2.connect()`. No `pool_pre_ping`, no connection reuse. Performance concern at scale. |
| GAP-09-011 | LOW | No migration framework — schema is ensured via `CREATE TABLE IF NOT EXISTS` + `ALTER TABLE ADD COLUMN IF NOT EXISTS` on every request. No Alembic, no versioned migrations. |
| GAP-09-012 | LOW | Stale codebase — last pushed 2026-04-30, ~6 months stale at inspection date (2026-10-18). Commit history shows sequential web-editor edits. |
| GAP-09-013 | LOW | External API dependency without caching — `/exchange-rates` calls frankfurter.app on every request. No caching, no retry, no fallback. |
| GAP-09-014 | LOW | Duplicated auth functions — `get_bearer_token()` and `get_authenticated_user()` are duplicated across `routes/task_routes.py`, `routes/calendar_routes.py`, `routes/user_routes.py`, and `services/auth_service.py`. Code duplication. |
| GAP-09-015 | LOW | `services/services/auth_service.py` — duplicate directory nesting (`services/services/`). Contains a copy of `auth_service.py`. Likely an accidental commit. |

---

## 9. Claim Comparison

| Claim ID | Claim | Phase 1 Class | Phase 5 Assessment |
|----------|-------|---------------|---------------------|
| CLM-09-001 | "Personal AI Assistant integrates OpenAI for responses" | REPORTED / NOT_TESTED | **OBSERVED** — `services/ai_service.py` imports `from openai import OpenAI`. `get_openai_client()` reads `OPENAI_API_KEY` env var. `generate_ai_reply()` calls `client.chat.completions.create()`. `extract_task_from_message()` and `decide_smart_action()` also use OpenAI. `requirements.txt` includes `openai==1.30.1`. Integration confirmed at source level. |

---

## 10. Reproducibility Recipe (R1 → R3 path)

This is a documented path to reach R3 independently. It has NOT been executed (not performed; source repositories are treated as read-only).

```bash
git clone https://github.com/aminbita162-glitch/personal-ai-assistant.git
cd personal-ai-assistant
git checkout 455a3ebb4f84a6ca648d7a8481f864b21680f235  # HEAD SHA
# Requires: Python 3.11.9, PostgreSQL
pip install -r requirements.txt
# Set environment variables
export DATABASE_URL="postgresql://user:pass@localhost:5432/personal_ai"
export OPENAI_API_KEY=sk-...
# Start the server
python main.py  # or: gunicorn main:app --bind 0.0.0.0:8000
# Health check
curl http://localhost:8000/health
# Initialize database tables
curl -X POST http://localhost:8000/init-db
# Signup
curl -X POST http://localhost:8000/signup -H "Content-Type: application/json" -d '{"name":"Test","email":"test@test.com","password":"password123"}'
# AI chat (unauthenticated)
curl -X POST http://localhost:8000/ai -H "Content-Type: application/json" -d '{"message":"Hello"}'

# NOTE: No tests exist. There is nothing to run with pytest.
```

Preconditions: Python 3.11.9; `DATABASE_URL` env var for PostgreSQL; `OPENAI_API_KEY` for AI endpoints (otherwise `ValueError`). No test suite — `pytest` will collect 0 tests.

**Reproduction note:** This is an MVP with no tests, no CI, and no deployment documentation. The app will start with `DATABASE_URL` + `OPENAI_API_KEY` but has no test verification path. Source inspection at pinned SHA is the only evidence level (R1).

---

## 11. Evidence Matrix

| EVD ID | Claim | Source path | Observation class | Result | Source URL | Source SHA | Notes |
|--------|-------|-------------|-------------------|--------|------------|------------|-------|
| EVD-09-001 | No license file — GitHub API license null | repo root | OBSERVED | BLOCKED | https://github.com/aminbita162-glitch/personal-ai-assistant | `455a3ebb4f84` | No LICENSE file in 29-file tree. GitHub API `license: null`. No source copied. |
| EVD-09-002 | No CI workflow — 0 .github/workflows | repo tree | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/tree/455a3ebb4f84a6ca648d7a8481f864b21680f235 | `455a3ebb4f84` | 29 files, 0 in .github/workflows/. No CI to trigger. |
| EVD-09-003 | 0 test files, 0 test functions | repo tree | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/tree/455a3ebb4f84a6ca648d7a8481f864b21680f235 | `455a3ebb4f84` | No tests/ directory. No test files in tree. No pytest, no unittest. |
| EVD-09-004 | 0 release tags, 0 published releases | GitHub API | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant | `455a3ebb4f84` | Tags: 0. Releases: 0. Version 1.0.0 in `/app-info` + `data/app_config.json` only. |
| EVD-09-005 | OpenAI integration confirmed at source | `services/ai_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/services/ai_service.py | `455a3ebb4f84` | `from openai import OpenAI`; `get_openai_client()`; `generate_ai_reply()`; `extract_task_from_message()`; `decide_smart_action()`. |
| EVD-09-006 | Flask 3.0.3 + psycopg2 + bcrypt stack | `requirements.txt` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/requirements.txt | `455a3ebb4f84` | `Flask==3.0.3`, `psycopg2-binary==2.9.9`, `openai==1.30.1`, `bcrypt==4.1.2`, `gunicorn==21.2.0`, `httpx==0.27.2`, `requests==2.31.0`. |
| EVD-09-007 | Bcrypt password hashing | `routes/user_routes.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/user_routes.py | `455a3ebb4f84` | `bcrypt.hashpw(password, bcrypt.gensalt())` on signup; `bcrypt.checkpw()` on login. |
| EVD-09-008 | Auth token: secrets.token_hex(32) (256-bit) | `routes/user_routes.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/user_routes.py | `455a3ebb4f84` | `generate_auth_token()` returns `secrets.token_hex(32)`. Token rotates on each login. Logout sets `auth_token = NULL`. |
| EVD-09-009 | Auth token stored as plaintext in DB | `routes/user_routes.py` + `routes/user_routes.py` schema | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/user_routes.py | `455a3ebb4f84` | `auth_token TEXT` column in users table — plaintext, not hashed. GAP-09-005. |
| EVD-09-010 | User-scoped task queries (WHERE user_id) | `routes/task_routes.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/task_routes.py | `455a3ebb4f84` | All CRUD: `WHERE user_id = %s`. Cross-user reads impossible by query design. |
| EVD-09-011 | Unauthenticated AI endpoints (/ai, /ai-browser) | `routes/ai_routes.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/ai_routes.py | `455a3ebb4f84` | `/ai` and `/ai-browser` have no `get_current_user()` call. Any user can consume OpenAI tokens. GAP-09-004. |
| EVD-09-012 | Permissive CORS (allow-origin: *) | `main.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/main.py | `455a3ebb4f84` | `@app.after_request` sets `Access-Control-Allow-Origin: "*"`. GAP-09-007. |
| EVD-09-013 | DATABASE_URL required at runtime | `main.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/main.py | `455a3ebb4f84` | `get_connection()`: `if not database_url: raise ValueError("DATABASE_URL is not set")`. |
| EVD-09-014 | Runtime schema init (CREATE TABLE IF NOT EXISTS) | `routes/task_routes.py` + `routes/user_routes.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/task_routes.py | `455a3ebb4f84` | `ensure_tasks_schema()` + `ensure_users_schema()` + `ensure_appointments_schema()`. Called per request. No Alembic. |
| EVD-09-015 | OpenAI chat model defaults to gpt-4o-mini | `services/ai_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/services/ai_service.py | `455a3ebb4f84` | `get_chat_model()` returns `os.getenv("OPENAI_CHAT_MODEL", "gpt-4o-mini")`. |
| EVD-09-016 | AI task extraction with JSON parsing fallback | `services/ai_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/services/ai_service.py | `455a3ebb4f84` | `extract_first_json_object()` — brace-matching parser. Fallback: `title=user_message[:80]`, `priority=medium`, `status=pending`. |
| EVD-09-017 | Smart AI decision (task vs reply) | `services/ai_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/services/ai_service.py | `455a3ebb4f84` | `decide_smart_action()` → OpenAI JSON with `action: "task" or "reply"`. If task → `extract_task_from_message()`. If reply → `generate_ai_reply()`. |
| EVD-09-018 | Voice transcription via OpenAI Whisper | `routes/ai_routes.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/ai_routes.py | `455a3ebb4f84` | `client.audio.transcriptions.create(model="gpt-4o-mini-transcribe", file=audio)`. Temp file deleted in `finally`. |
| EVD-09-019 | External API: frankfurter.app exchange rates | `main.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/main.py | `455a3ebb4f84` | `/exchange-rates` → `requests.get("https://api.frankfurter.app/latest?from=USD&to=EUR,GBP,CAD,AUD,JPY", timeout=10)`. No caching/retry. |
| EVD-09-020 | No rate limiting, no audit, no retry on AI calls | `services/ai_service.py` + `main.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/services/ai_service.py | `455a3ebb4f84` | No `slowapi`, no `flask-limiter`, no retry logic, no audit log. GAP-09-008, GAP-09-009. |
| EVD-09-021 | 29 total tracked files | repo tree | OBSERVED | PASS | https://github.com//aminbita162-glitch/personal-ai-assistant/tree/455a3ebb4f84a6ca648d7a8481f864b21680f235 | `455a3ebb4f84` | 29 files: 14 .py, 1 README, 1 requirements.txt, 1 .python-version, 1 runtime.txt, 1 data/app_config.json, 3 static files, 1 template. |
| EVD-09-022 | Version 1.0.0 in /app-info + app_config.json | `main.py` + `data/app_config.json` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/main.py | `455a3ebb4f84` | `/app-info` returns `"version": "1.0.0"`. `data/app_config.json`: `"version": "1.0.0"`. |
| EVD-09-023 | Multilingual AI instruction in all prompts | `services/ai_service.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/services/ai_service.py | `455a3ebb4f84` | `get_language_instruction()`: "You must ALWAYS detect the user's language... Support ALL languages..." Added to all AI prompts. |
| EVD-09-024 | Duplicated auth functions across 4 files | `routes/task_routes.py` + `routes/user_routes.py` + `services/auth_service.py` + `services/services/auth_service.py` | OBSERVED | CONDITIONAL | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/task_routes.py | `455a3ebb4f84` | `get_bearer_token()` and `get_authenticated_user()`/`get_current_user()` duplicated. Plus `services/services/` nested directory. GAP-09-014, GAP-09-015. |
| EVD-09-025 | No migration framework (runtime CREATE TABLE) | `routes/task_routes.py` + `routes/user_routes.py` | OBSERVED | PASS | https://github.com/aminbita162-glitch/personal-ai-assistant/blob/455a3ebb4f84a6ca648d7a8481f864b21680f235/routes/task_routes.py | `455a3ebb4f84` | `ensure_tasks_schema()` called per request. `CREATE TABLE IF NOT EXISTS` + `ALTER TABLE ADD COLUMN IF NOT EXISTS`. No Alembic. GAP-09-011. |
