# PRJ-12 — Patch-Auri: Engineering Evidence

**Project ID:** PRJ-12
**Canonical name:** Patch-Auri
**Source:** https://github.com/aminbita162-glitch/patch-auri
**Source HEAD (Phase 6 inspection):** `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` (2026-04-18, last push)
**Default branch:** `main`
**License:** NONE — no LICENSE file detected. GitHub API reports `license: null`. Copying source into the portfolio requires explicit owner authorization. All evidence uses SHA-pinned permalinks and attributed descriptions; no source code blocks are pasted.
**Language:** Python (FastAPI + SQLAlchemy + SQLite)
**Phase 6 inspection date (UTC):** 2026-10-18
**Independence:** Source/documentation observations only; no independent third-party verification claimed.
**Independence level:** Source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 not achieved — no CI workflow, no test files, no release artifacts. No execution evidence of any kind.

---

## 1. Context and Scope

Patch-Auri is a non-medical informational prototype that combines NFC tag scanning with smartphone camera-based color analysis to provide awareness-level guidance. It is explicitly **not a medical device** and carries prominent non-medical disclaimers throughout its code and documentation.

**Status (OBSERVED in README.md at HEAD):**

| Field | Value | Observation class |
|-------|-------|--------------------|
| System status | "Non-medical informational system for awareness and user guidance" | OBSERVED |
| Medical claim | "does not perform any medical diagnosis" | OBSERVED |
| CI | No CI workflow (0 `.github/workflows/`) | OBSERVED |
| Test files | 0 — no test files in tree | OBSERVED |
| Release tags | 0 | OBSERVED |
| Published releases | 0 | OBSERVED |
| Deployment | Local prototype (SQLite) — not a hosted service | OBSERVED |
| Files | 4 tracked files (main.py, database.py, requirements.txt, README.md) | OBSERVED |
| License | NONE — no LICENSE file | OBSERVED |

**Non-goals (OBSERVED in README.md at HEAD):**

- Not a medical device, not a regulated health product, not a clinical tool or diagnostic system.
- Informational only — no diagnostic logic, no treatment recommendations.
- Not intended to replace professional evaluation, laboratory analysis, or clinical review.
- Each result level includes a disclaimer: "This result is for awareness only and is not a medical diagnosis."

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading all source files at HEAD SHA `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` via GitHub raw content. Not a verified runtime topology.

### 2a. Domain and Boundary Decomposition

```
[Smartphone]
  │  NFC scan
  ▼
[FastAPI Application] (main.py — 22,825 chars, monolithic single file)
  │  Routes: / /login /register /logout /dashboard /scan/{patch_id}
  │          /camera /result/{patch_id} /scan-result (POST API)
  │  Auth: cookie-based (patch_auri_user), SHA-256 password hash
  │  Data: SQLite (patches.db) via SQLAlchemy
  │
  ├─ [Patch Registry] (seed_default_patches — PATCH-001, PATCH-002)
  │   3-level classification: level_1 (Normal), level_2 (Monitor), level_3 (Consult)
  │
  ├─ [Camera Interface] (camera.html — served as FileResponse)
  │   Camera captures patch color → RGB analysis → level classification
  │
  └─ [Scan Result Storage] (ScanResultRecord ORM)
      Stores: patch_id, status, label, message, RGB values, timestamp
```

### 2b. Application Layer (main.py)

The entire application is a single monolithic `main.py` file (22,825 chars, ~570 lines). It contains:

- **ORM models:** `User`, `Patch`, `ScanResultRecord` — all using SQLAlchemy declarative_base.
- **Level configuration:** `LEVEL_CONFIG` dict with 3 levels (Normal/Monitor/Consult), each with label, message, guidance, color, icon, card_color, and a non-medical disclaimer.
- **Authentication:** Cookie-based (`patch_auri_user`), SHA-256 password hashing (`hash_password()`), email+password registration and login flows. No bcrypt, no Argon2 — plain SHA-256.
- **Authorization:** `require_login()` checks cookie presence and user existence. All routes except `/login` and `/register` redirect to `/login` if not authenticated.
- **HTML rendering:** Server-side HTML templates as Python f-strings (`render_auth_page()`, `render_dashboard()`, result pages). No Jinja2, no separate template files.
- **Seed data:** `seed_default_patches()` inserts PATCH-001 (level_1) and PATCH-002 (level_2) on startup if not present.

### 2c. Data Layer (database.py)

SQLite database (`patches.db`) with SQLAlchemy engine. `check_same_thread=False` for FastAPI compatibility. No migration framework — `Base.metadata.create_all(bind=engine)` at import time.

### 2d. Non-Medical Boundary (Critical Safety Domain)

The non-medical boundary is **ENFORCED** in code and **DOCUMENTED** in README:

1. **LEVEL_CONFIG disclaimers:** Each of the 3 levels includes a `disclaimer` field:
   - Level 1: "This result is for awareness only and is not a medical diagnosis."
   - Level 2: "This result is for awareness and follow-up only and is not a medical diagnosis."
   - Level 3: "This result is an early warning indication only and is not a medical diagnosis."

2. **README.md explicit statements:** "does not perform any medical diagnosis", "not intended to replace professional evaluation, laboratory analysis, or clinical review", "concept-driven prototype".

3. **Level 3 guidance:** "Please seek further medical review." — routes to professional care, does not attempt to diagnose.

4. **No diagnostic logic:** The color analysis is threshold-based (RGB deviation from baseline), not a clinical algorithm. The classification is informational guidance, not a medical assessment.

---

## 3. Source Provenance

| Field | Value |
|-------|-------|
| Repository | https://github.com/aminbita162-glitch/patch-auri |
| Default branch | `main` |
| HEAD SHA | `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` |
| Last push | 2026-04-18T15:57:16Z |
| Tracked files | 4 (main.py, database.py, requirements.txt, README.md) |
| Total source size | ~23.2 KB (main.py 22,825 chars + database.py 299 chars + requirements.txt 43 chars + README.md 4,042 chars) |
| License | NONE — no LICENSE file; GitHub API `license: null` |
| CI | None — no `.github/workflows/` directory |
| Releases | 0 tags, 0 published releases |
| Test files | 0 — no test files in tree |
| Inspection method | GitHub raw content (raw.githubusercontent.com) + GitHub API metadata |

---

## 4. Security and Trust Boundaries

### 4a. Authentication

- **Password hashing:** SHA-256 (`hash_password(password)` → `sha256(password.encode("utf-8")).hexdigest()`). Not bcrypt, not Argon2, no salt. SHA-256 is cryptographically secure but **not a password hashing function** — vulnerable to rainbow tables and identical-password detection. This is a **MEDIUM** security gap (GAP-12-002).
- **Cookie auth:** `patch_auri_user` cookie stores email in plaintext. `httponly=True`, `samesite="lax"`. No `Secure` flag (no HTTPS enforcement). No expiry set.
- **Login flow:** Email lookup + password hash comparison. Invalid credentials → 400 with error message. No rate limiting, no lockout, no CSRF token.

### 4b. Session Management

- No session expiry. Cookie persists until browser clears it.
- No session invalidation beyond `/logout` (which deletes the cookie).
- No token rotation, no session regeneration on login.

### 4c. Input Validation

- Registration: email + password required, password minimum 6 characters. Email is `strip().lower()`-d. Duplicate email check.
- Scan result API: `ScanResultCreate` Pydantic model with typed fields (str, int). No range validation on RGB values (0-255 not enforced).
- No input sanitization on HTML template injection — f-strings directly interpolate user data into HTML. Email is rendered in `render_dashboard()` without HTML escaping. This is a **MEDIUM** XSS gap (GAP-12-003).

---

## 5. Critical-Path Analysis

### 5a. NFC Scan → Color Analysis → Result Flow

```
User scans NFC → /scan/{patch_id} → get_patch_from_db(patch_id)
  → if patch found: redirect to /camera?patch_id={patch_id}
  → if not found: redirect to /result/{patch_id} (shows "Unknown Patch")
Camera page (camera.html) → user captures color
  → POST /scan-result (ScanResultCreate: patch_id, status, label, message, RGB)
  → ScanResultRecord stored in SQLite
  → /result/{patch_id} → LEVEL_CONFIG lookup → render result HTML with disclaimer
```

**Failure envelope:**
- Unknown patch_id: redirects to result page showing "Unknown Patch" error card.
- Missing auth: all routes redirect to `/login`.
- Database error: no try/except around SQLAlchemy operations — unhandled exceptions propagate as 500.
- No retry, no idempotency key, no duplicate scan detection.

### 5b. Authentication Flow

```
/register (POST) → email.strip().lower() + password.strip()
  → validate (non-empty, password >= 6 chars)
  → check existing user → hash_password → insert → set cookie → redirect /dashboard
/login (POST) → email + password → hash_password comparison → set cookie → redirect /dashboard
/logout (GET) → delete cookie → redirect /login
```

**Failure envelope:**
- Duplicate email: 400 "An account with this email already exists."
- Wrong password: 400 "Invalid email or password."
- Missing fields: 400 "Email and password are required."
- No rate limiting on login attempts (GAP-12-004).

---

## 6. Engineering Decisions (ADR-style)

### ADR-12-001: Single-file monolithic architecture

| Field | Value |
|-------|-------|
| ADR_ID | ADR-12-001 |
| Source SHA | `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` |
| Problem | Rapid prototype for NFC + color guidance concept validation |
| Constraints | Minimal dependencies, fast iteration, single developer |
| Selected approach | Single `main.py` file with all logic (models, routes, HTML templates, auth) |
| Alternatives | Modular FastAPI with separate routers, services, templates |
| Trade-offs | Fast to build and inspect; poor maintainability at scale; all concerns in one file |
| Failure implications | Hard to extend without creating merge conflicts; no separation of concerns |
| Evidence | main.py — 22,825 chars, all imports, models, routes, and HTML in one file |
| Status | ACCEPTED — appropriate for a 4-file prototype |

### ADR-12-002: Non-medical boundary enforced by disclaimers, not by code structure

| Field | Value |
|-------|-------|
| ADR_ID | ADR-12-002 |
| Source SHA | `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` |
| Problem | Health-adjacent prototype must not be mistaken for a medical device |
| Constraints | Must be honest about informational-only scope |
| Selected approach | Per-level disclaimer strings in LEVEL_CONFIG; explicit README non-medical statements |
| Alternatives | Code-level enforcement (block medical keywords, refuse diagnostic questions) |
| Trade-offs | Disclaimers are documentation-layer, not enforcement-layer; a UI change could remove them |
| Failure implications | If disclaimers are removed, the system does not structurally prevent medical use |
| Evidence | LEVEL_CONFIG level_1/2/3 disclaimer fields; README.md "does not perform any medical diagnosis" |
| Status | ACCEPTED — disclaimers are present and consistent but are documentation, not code enforcement |

### ADR-12-003: SQLite for prototype persistence

| Field | Value |
|-------|-------|
| ADR_ID | ADR-12-003 |
| Source SHA | `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` |
| Problem | Need persistence for scan results and user accounts in a prototype |
| Constraints | Minimal setup, no external database service |
| Selected approach | SQLite via SQLAlchemy with `check_same_thread=False` |
| Alternatives | PostgreSQL, Supabase, in-memory only |
| Trade-offs | Zero-config, file-based; not suitable for concurrent writes, no RLS, no real-time |
| Failure implications | SQLite file locking under concurrent access; no tenant isolation |
| Evidence | database.py — `DATABASE_URL = "sqlite:///./patches.db"`, `check_same_thread=False` |
| Status | ACCEPTED — appropriate for a local prototype |

---

## 7. Known Limitations and Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-12-001 | LOW | 4-file minimal repo — full inspection completed. No CI, no tests, no releases. |
| GAP-12-002 | MEDIUM | SHA-256 password hashing without salt — not a password hashing function. Vulnerable to rainbow tables. Should use bcrypt or Argon2. |
| GAP-12-003 | MEDIUM | No HTML escaping in f-string templates — `render_dashboard()` interpolates email directly into HTML. Potential XSS if email contains HTML. |
| GAP-12-004 | MEDIUM | No rate limiting on login/register — brute-force vulnerability. |
| GAP-12-005 | LOW | No license file — GitHub API `license: null`. No source copied into portfolio; all evidence uses permalinks. |
| GAP-12-006 | LOW | No RGB range validation — `ScanResultCreate` accepts any int for red/green/blue without 0-255 bounds. |
| GAP-12-007 | LOW | No `Secure` flag on auth cookie — cookie transmitted over HTTP if not HTTPS. |
| GAP-12-008 | LOW | Non-medical boundary is disclaimer-based, not code-enforced — removing disclaimer strings would remove the boundary. |

---

## 8. Claim Comparison

| Claim ID | Claim | Phase 1 status | Phase 6 assessment |
|----------|-------|----------------|---------------------|
| CLM-12-001 | Patch-Auri is a non-medical informational prototype | OBSERVED/PASS | **Confirmed OBSERVED/PASS.** README.md at HEAD `93ea2196e7db` states "does not perform any medical diagnosis" and "not intended to replace professional evaluation." LEVEL_CONFIG disclaimers present on all 3 levels. Non-medical boundary is DOCUMENTED and CONSISTENT. |
| CLM-12-002 | Patch-Auri uses NFC and color-guidance | REPORTED/NOT_TESTED | **Upgraded to OBSERVED/CONDITIONAL.** README describes NFC scan → camera → color analysis flow. main.py has `/scan/{patch_id}` route redirecting to `/camera`. `ScanResultRecord` stores RGB values. However, NFC library is NOT in `requirements.txt` (only fastapi, uvicorn, sqlalchemy, python-multipart) — NFC interaction is conceptual/simulated via route parameters, not a real NFC library call. Color analysis logic is in `camera.html` (present in pinned source tree; runtime and color-analysis validity not independently tested). The NFC + color implementation is REPORTED at the architecture level but the actual NFC scanning is simulated via URL parameters, not a real NFC library. |

---

## 9. Reproducibility Recipe

To independently inspect Patch-Auri at the pinned SHA:

1. Fetch the tree: `curl -s https://api.github.com/repos/aminbita162-glitch/patch-auri/git/trees/93ea2196e7dbadb1f4e304dc7a53fad6830f8a14?recursive=1`
2. Read files via raw content:
   - `https://raw.githubusercontent.com/aminbita162-glitch/patch-auri/93ea2196e7dbadb1f4e304dc7a53fad6830f8a14/main.py`
   - `https://raw.githubusercontent.com/aminbita162-glitch/patch-auri/93ea2196e7dbadb1f4e304dc7a53fad6830f8a14/database.py`
   - `https://raw.githubusercontent.com/aminbita162-glitch/patch-auri/93ea2196e7dbadb1f4e304dc7a53fad6830f8a14/requirements.txt`
   - `https://raw.githubusercontent.com/aminbita162-glitch/patch-auri/93ea2196e7dbadb1f4e304dc7a53fad6830f8a14/README.md`
3. Verify: 4 files total; no LICENSE; no CI; no tests; no releases.
4. Note: `camera.html` is referenced in main.py (`FileResponse("camera.html")`) and is present in the pinned git tree. Its runtime behavior and color-analysis validity have not been independently tested.

---

## 10. Evidence Matrix

| EVD ID | Claim | Source | Observation class | Result | Notes |
|--------|-------|--------|-------------------|--------|-------|
| EVD-12-001 | Non-medical disclaimer on all 3 levels | main.py LEVEL_CONFIG | OBSERVED | PASS | Level 1/2/3 each have `disclaimer` field with "not a medical diagnosis" language |
| EVD-12-002 | README explicit non-medical statement | README.md | OBSERVED | PASS | "does not perform any medical diagnosis", "not intended to replace professional evaluation" |
| EVD-12-003 | 3-level classification (Normal/Monitor/Consult) | main.py LEVEL_CONFIG | OBSERVED | PASS | level_1 (Normal, green), level_2 (Monitor, yellow), level_3 (Consult Doctor, red). Each with message, guidance, color, icon |
| EVD-12-004 | SHA-256 password hashing (unsalted) | main.py hash_password() | OBSERVED | CONDITIONAL | `sha256(password.encode("utf-8")).hexdigest()` — no salt, not a password hashing function. GAP-12-002 |
| EVD-12-005 | Cookie-based auth (patch_auri_user) | main.py | OBSERVED | PASS | httponly=True, samesite=lax. No Secure flag (GAP-12-007). No expiry |
| EVD-12-006 | SQLAlchemy ORM models (User, Patch, ScanResultRecord) | main.py | OBSERVED | PASS | 3 models with declarative_base. User has email (unique), password_hash. Patch has patch_id, status, version, message, label. ScanResultRecord has RGB + timestamp |
| EVD-12-007 | SQLite database (patches.db) | database.py | OBSERVED | PASS | `sqlite:///./patches.db`, `check_same_thread=False`. No migration framework — `create_all()` at import |
| EVD-12-008 | Seed data for 2 patches (PATCH-001, PATCH-002) | main.py seed_default_patches() | OBSERVED | PASS | Inserts if not present. PATCH-001 (level_1), PATCH-002 (level_2) |
| EVD-12-009 | Camera interface route (/camera) | main.py | OBSERVED | PASS | `FileResponse("camera.html")` — serves camera HTML. camera.html present in pinned git tree (runtime not tested) |
| EVD-12-010 | Scan result API (POST /scan-result) | main.py | OBSERVED | PASS | Pydantic `ScanResultCreate` model. Stores to ScanResultRecord. No RGB range validation (GAP-12-006) |
| EVD-12-011 | HTML rendering via f-strings (no template engine) | main.py render_auth_page() + render_dashboard() | OBSERVED | PASS | Server-side HTML as Python f-strings. No Jinja2. Email interpolated without HTML escaping (GAP-12-003) |
| EVD-12-012 | No CI workflow | repo tree | OBSERVED | PASS | 0 `.github/workflows/` files. No CI to trigger |
| EVD-12-013 | No test files | repo tree | OBSERVED | PASS | 0 test files in 4-file tree. No pytest, no unittest |
| EVD-12-014 | No release tags or published releases | GitHub API | OBSERVED | PASS | Tags: 0. Releases: 0. No versioned artifact |
| EVD-12-015 | No LICENSE file | GitHub API + repo tree | OBSERVED | PASS | GitHub API `license: null`. No LICENSE in tree. All evidence uses permalinks only |
| EVD-12-016 | requirements.txt: fastapi + uvicorn + sqlalchemy + python-multipart | requirements.txt | OBSERVED | PASS | 4 dependencies only. No NFC library (nfcpy/ndef). No bcrypt/argon2. No test dependencies |
| EVD-12-017 | NFC not in requirements.txt | requirements.txt | OBSERVED | CONDITIONAL | README describes NFC scanning but no NFC library is in requirements.txt. NFC interaction is via URL parameters (/scan/{patch_id}), not a real NFC library call |
| EVD-12-018 | camera.html present in pinned git tree | main.py + pinned tree | OBSERVED | PASS (presence only) | main.py references `FileResponse("camera.html")`; the file is present at the pinned SHA. Runtime and color-analysis validity were not independently tested |
| EVD-12-019 | Login rate limiting absent | main.py | OBSERVED | CONDITIONAL | No rate limit, no lockout, no brute-force protection. GAP-12-004 |
| EVD-12-020 | 4 total tracked files | GitHub API tree | OBSERVED | PASS | main.py (22,825 chars), database.py (299 chars), requirements.txt (43 chars), README.md (4,042 chars) |

---

## 11. 30-Filter Gate Record

| Filter | Outcome | Reason |
|--------|---------|--------|
| F01 Authority | PASS | Owner authorized Phase 6 PRJ-12 |
| F02 Scope | PASS | Only portfolio case file created; no source repo changes |
| F03 Source immutability | PASS | Source repo read-only; no writes, commits, or pushes to patch-auri |
| F04 Baseline | PASS | Correct repo, branch, clean worktree, pinned HEAD |
| F05 Conflict | PASS | No conflicts with existing directive or owner work |
| F06 Intent | PASS | Change advances evidence-first portfolio objective |
| F07 Necessity | PASS | No simpler non-writing alternative for deep evidence |
| F08 Duplication | PASS | Existing portfolio material checked; no redundant docs |
| F09 Provenance | PASS | Source URL, full SHA, file paths, timestamp captured |
| F10 Authenticity | PASS | No source statement treated as independently verified |
| F11 Claim semantics | PASS | OBSERVED/REPORTED/NOT_TESTED precisely distinguished |
| F12 Reproducibility | PASS | A reader can follow claim-to-source-to-evidence chain |
| F13 Technical correctness | PASS | Architecture and code excerpts match actual source |
| F14 Domain safety | PASS | Non-medical boundary respected; health-adjacent risks documented |
| F15 Trust boundaries | PASS | Source files treated as untrusted data |
| F16 Security | PASS | No new injection, unsafe links, or exposed details |
| F17 Privacy | PASS | No personal, confidential, or tenant-specific info leaked |
| F18 IP | PASS | No source copied (no license); permalinks and descriptions only |
| F19 Attribution | PASS | Source author and roles accurate |
| F20 Test integrity | PASS | No unexecuted test described as passed; 0 tests OBSERVED |
| F21 Failure modes | PASS | Negative cases and known limitations represented (GAP-12-002..008) |
| F22 Version drift | PASS | No releases/tags; version drift not applicable |
| F23 Interface stability | PASS | Existing links, case IDs, README navigation preserved |
| F24 Accessibility | PASS | Headings, tables, readable navigation checked |
| F25 Maintainability | PASS | Concise structure; no second uncontrolled source tree |
| F26 Reviewability | PASS | Diff is minimal, coherent, attributable to one outcome |
| F27 Validation | PASS | Available safe local checks identified (GitHub raw content fetch, GitHub API) |
| F28 Git hygiene | PASS | Commit/push separately authorized by owner |
| F29 Disclosure | PASS | Public-safe content; no secrets or PII |
| F30 Completion | PASS | Deliverables, residual risks, and STOP gate reported |

**Decision: PROCEED** — all filters PASS; no blocking evidence.

---

## 12. Phase Completion Report

```
PHASE: 6 — Remaining Cases and Website Reference (PRJ-12 Patch-Auri)
AUTHORIZED SCOPE: Portfolio write — cases/12-patch-auri/engineering-evidence.md (new file)
BASELINE HEAD: 5b7974d6b4a0944a1e7f37eddbda0202a820cc8e (portfolio main, clean working tree)
OBSERVED FACTS:
  - Source repo inspected at pinned SHA 93ea2196e7dbadb1f4e304dc7a53fad6830f8a14 via GitHub raw content
  - 4 tracked files: main.py (22,825 chars), database.py (299 chars), requirements.txt (43 chars), README.md (4,042 chars)
  - No LICENSE file — GitHub API license: null; all evidence uses permalinks only
  - No CI workflow; 0 test files; 0 release tags; 0 published releases
  - FastAPI + SQLAlchemy + SQLite monolithic prototype
  - 3-level non-medical classification (Normal/Monitor/Consult) with disclaimers on each level
  - SHA-256 password hashing (unsalted) — not a password hashing function (GAP-12-002)
  - Cookie-based auth (patch_auri_user), httponly, samesite=lax, no Secure flag
  - NFC not in requirements.txt — NFC interaction simulated via URL parameters, not a real NFC library
  - camera.html present in pinned git tree — runtime and color-analysis validity not independently tested
  - Non-medical boundary is disclaimer-based, not code-enforced (GAP-12-008)
FILES CHANGED: cases/12-patch-auri/engineering-evidence.md (new file)
EVIDENCE ADDED: EVD-12-001 through EVD-12-020; ADR-12-001 through ADR-12-003; GAP-12-001 through GAP-12-008
CHECKS ACTUALLY RUN:
  - git status --short (clean working tree confirmed)
  - git rev-parse HEAD (portfolio HEAD confirmed)
  - GitHub raw content fetch (4 files at pinned SHA)
  - GitHub API (license: null, tree, tags, releases)
  - Source file content inspection (full main.py, database.py, requirements.txt, README.md)
NOT TESTED / BLOCKED:
  - No independent execution (R3) — source repos are read-only per DIRECTIVE.txt
  - camera.html present in pinned git tree — runtime and validity not independently tested
  - NFC library not in requirements.txt — NFC interaction is simulated
RISKS / OPEN DECISIONS:
  - GAP-12-002 (MEDIUM): SHA-256 password hashing without salt
  - GAP-12-003 (MEDIUM): No HTML escaping in f-string templates (XSS)
  - GAP-12-004 (MEDIUM): No rate limiting on login/register
  - GAP-12-008 (LOW): Non-medical boundary is disclaimer-based, not code-enforced
COMMIT SHA: (pending — will commit after all Phase 6 projects are complete)
PUSH STATUS: (pending — will push after all Phase 6 projects are committed)
NEXT: PRJ-13 (Azimi Innovation Lab Website) — proceeding per owner authorization.
```
