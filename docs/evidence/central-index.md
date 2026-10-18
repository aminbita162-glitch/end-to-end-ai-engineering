# Central Evidence Index

**Phase:** 6 — Remaining Cases and Website Reference (PRJ-06, PRJ-07, PRJ-12, PRJ-13 — complete)
**Created:** 2026-10-17
**Phase 1 portfolio HEAD:** `0e8a84cd20322a48da0b722981d1e535394ee6c3`
**Phase 2 portfolio HEAD:** `3270b21a350eafa3a152b750faaafbca47756785`
**Phase 3 (PRJ-03) portfolio HEAD:** `4810335126309abf64278576c35cc5744a98b263`
**Phase 3 (PRJ-01) portfolio HEAD:** `d3ba70b3a46fdabf2751525f4d835a6738a435d9`
**Phase 3 (PRJ-02) portfolio HEAD:** `281f30d9022a13793a78575767c33c87f9b10925`
**Phase 4 (PRJ-05) portfolio HEAD:** `3bbf086613c0d7d037ec6dc7fb0e0857f35dec87`
**Phase 4 (PRJ-10) portfolio HEAD:** `f1eed692672b5fd93345d4ad844e530fcd406320`
**Phase 5 (PRJ-08) portfolio HEAD:** `12fe7266062e4d904d6cbac3c92e9eb042469a6b`
**Independence:** Source/documentation observations only; no independent reproduction claimed.
**Status:** Authoritative navigation index for all portfolio evidence.

This file is the single entry point for navigating from a portfolio claim to its evidence. It summarises the 13 projects, links all evidence files, and shows the current observation maturity for each project. Deep-dive case evidence is added in Phases 3–8.

---

## How to Navigate

**5-minute review:** Read the [Project Status Summary](#2-project-status-summary) table. Check the `evidence_maturity` column. Follow a specific project row to its source SHA and Phase 1 gaps.

**15-minute review:** Follow links in the [Per-Project Evidence Map](#3-per-project-evidence-map) for flagship cases (PRJ-01, PRJ-02, PRJ-03). Read the linked `reviewer-notes.md` sections for CI findings and critical gaps.

**60-minute review:** Start with [schema.md](schema.md) to understand ID conventions, then [provenance-rules.md](provenance-rules.md) for how observations were made. Cross-reference `claims.csv` claim IDs against `reviewer-notes.md` gap IDs. Phase 3+ case files will add `EVD-` and `ADR-` records.

---

## 1. Evidence File Inventory

| File | Phase | Contents | Schema conformance |
|------|-------|----------|--------------------|
| [source-inventory.csv](source-inventory.csv) | 1 | Per-project source SHA, language, license, CI, releases | Conforms to Phase 1 fields; `head_sha_observed_at_utc` corrected to 2026-10-17 in Phase 2 |
| [claims.csv](claims.csv) | 1 | 34 source-to-claim records (CLM-01 through CLM-13); observation class and result status per claim | CLM- IDs assigned; conforms to Phase 1 fields |
| [verification-status.csv](verification-status.csv) | 1 | Per-project CI health, test evidence, deployment URL, consistency assessment, material gaps | Summary view; detailed gaps in reviewer-notes.md |
| [reviewer-notes.md](reviewer-notes.md) | 1 | SHA register, CI/test matrix, license register, 10 priority-ranked gaps, architectural signals, F01–F30 record, 5 open decisions | Narrative; gap IDs (GAP-) and open decision IDs defined |
| [schema.md](schema.md) | 2 | Unique ID scheme (PRJ/CLM/EVD/GAP/ADR), required fields, artifact kinds, observation class/status taxonomy, evidence strength (E0–E5), reproducibility levels (R0–R5), independence levels | Normative; applies to Phase 2+ |
| [provenance-rules.md](provenance-rules.md) | 2 | Source SHA pinning (P-01–P-05), timestamp rules (P-06–P-08), provenance layers, independence disclosure (P-09–P-11), missing evidence (P-12–P-14), IP/sensitivity (P-15–P-16), observation record format | Normative; applies to Phase 2+ |
| [central-index.md](central-index.md) | 2 | This file — navigable index linking all evidence | Updated each phase |

Phase 3+ adds per-project case evidence files at `cases/<case>/engineering-evidence.md` with `EVD-` IDs referenced below. Phase 3 covers PRJ-03 (IBM Supply Chain), PRJ-01 (AILORA), and PRJ-02 (Aegis Agent). All three flagship cases are now complete.

---

## 2. Project Status Summary

Columns: **evidence_maturity** = highest reproducibility level achieved (R0–R5, per [schema.md §6](schema.md#6-reproducibility-levels)). **ci_status** = latest observed CI run status at Phase 1 HEAD SHA. **test_evidence** = source-level test file presence (OBSERVED / NOT_OBSERVED). **phase_1_gaps** = count of named gaps in reviewer-notes.md. **deep_dive_phase** = earliest phase authorized for deep engineering case work.

| ID | Project | Source available | Phase 1 HEAD SHA | evidence_maturity | ci_status | test_evidence | license | phase_1_gaps | deep_dive_phase |
|----|---------|-----------------|-----------------|-------------------|-----------|---------------|---------|-------------|-----------------|
| PRJ-01 | AILORA | PUBLIC | `46ca4a939bee` | **R2** | **FAIL** (HEAD) / SUCCESS (qualifying SHA `9571b168`) | OBSERVED | NOASSERTION | GAP-01-001 (HIGH — CI failure at freeze HEAD) | **COMPLETE** |
| PRJ-02 | Aegis Agent | PUBLIC | `69f159771c35` | **R1** | **ALL FAIL** | OBSERVED (269 files) | NOASSERTION | GAP-02-001 (CRITICAL) | **COMPLETE** |
| PRJ-03 | IBM Supply Chain | PUBLIC | `d00c5ebb2794` | **R1** | NO CI | OBSERVED | MIT | GAP-03-001 **RESOLVED** (Phase 3) | **COMPLETE** |
| PRJ-04 | AlinaMatrix Enterprise | PUBLIC | `e7cc7df20164` | R1 | PASS | OBSERVED | **NONE** | GAP-04-001 (HIGH) | PHASE 4 |
| PRJ-05 | Kian Trading Intelligence | PUBLIC | `54ef6ca4d8fd` | R1 | PASS (recently fixed) | OBSERVED | NOASSERTION | GAP-05-001 (HIGH) | PHASE 4 |
| PRJ-06 | Ahmad & Amin Technology | PUBLIC | `fbbfc307fd98` | **R1** | NO CI | MINIMAL (5 tests) | NOASSERTION | GAP-06-001 (HIGH — no CI) | **COMPLETE** |
| PRJ-07 | NimaOS AI Agent | **NO SOURCE** | NOT_AVAILABLE | **R0** | NO CI | NONE | UNKNOWN | GAP-07-001 (**CRITICAL** — no source) | **COMPLETE** |
| PRJ-08 | Lieferkette Optimierungsplattform | PUBLIC | `91c61210e913` | **R1** | PASS | MINIMAL (1 file) | **NONE** | GAP-08-001 (MEDIUM) | **COMPLETE** |
| PRJ-09 | Personal AI Assistant | PUBLIC | `455a3ebb4f84` | R1 | NO CI | NONE | **NONE** | GAP-09-001 (HIGH) | **COMPLETE** |
| PRJ-10 | Restaurant AI Platform | PUBLIC | `c49649a0b44e` | R1 | PASS (smoke) | OBSERVED | MIT | GAP-10-001 (MEDIUM — CI scope smoke-only) | **COMPLETE** |
| PRJ-11 | Büro Assistant | PUBLIC | `893db187185d` | R1 | NO CI | OBSERVED (856 functions) | NOASSERTION | GAP-11-001 (MEDIUM — no CI) | **COMPLETE** |
| PRJ-12 | Patch-Auri | PUBLIC | `93ea2196e7db` | **R1** | NO CI | NONE | **NONE** | GAP-12-001 (LOW — 4-file repo) | **COMPLETE** |
| PRJ-13 | Azimi Innovation Lab Website | **WEBSITE ONLY** | NOT_AVAILABLE | **R0** | N/A | N/A | UNKNOWN | GAP-13-001 (HIGH — no source) | **COMPLETE** |

**Notes:**
- "NOASSERTION" = license file present but not SPDX-recognized. Copying source requires rights confirmation.
- "NONE" (license) = no license file detected. Copying source into portfolio requires explicit owner authorization.
- All SHAs are observed-at-UTC 2026-10-17T00:00:00Z. Full SHAs in [source-inventory.csv](source-inventory.csv).
- evidence_maturity R1 means source and configuration inspected at immutable revision. No independent execution (R3) or longitudinal measurement (R5) has been performed.

---

## 3. Per-Project Evidence Map

### PRJ-01 AILORA ✓ Phase 3 Complete
- **Source:** https://github.com/azimilab2025-ai/ailora — HEAD `46ca4a939bee0396e90abef91905b6d1b95d470c`
- **Phase 3 case file:** [cases/01-ailora/engineering-evidence.md](../../cases/01-ailora/engineering-evidence.md)
- **Phase 1 claims:** CLM-01-001 through CLM-01-005 in [claims.csv](claims.csv)
- **Phase 3 evidence:** EVD-01-001 through EVD-01-017 in [cases/01-ailora/engineering-evidence.md §16](../../cases/01-ailora/engineering-evidence.md#16-evidence-matrix)
- **GAP-01-001 updated:** CI FAILURE at freeze HEAD `46ca4a939bee` (run `33390090146`, 2026-08-31) confirmed OBSERVED. Qualifying CI run `33025990734` at SHA `9571b168` (2026-08-27) is SUCCESS. Status: HIGH (was CRITICAL — downgraded because a qualifying run is linked at R2; freeze HEAD failure is documented).
- **Architecture:** Advisory-only SSA platform; no spacecraft command path (ADR-001); two-path auth (HS256 Render / RS256 foundation, ADR-002); PostgreSQL tenant RLS + application-layer context injection; SHA-256 audit hash chain; Ed25519 supply-chain admission; Oya disabled library agent with `max_requested_tools=0`.
- **Evidence maturity:** R2 — qualifying CI run artifact `33025990734` OBSERVED at SHA `9571b168`. R3 not reached — no independent clean-room reproduction.
- **Remaining gaps:** GAP-01-001 (CI failure at freeze HEAD), GAP-01-002 (RS256/OIDC not operational), GAP-01-003 (SCI-001 external gate), GAP-01-004 (OPS-001 external gate), GAP-01-005 (LEGAL-001 external gate), GAP-01-006 (no jti revocation on HS256 path), GAP-01-007 (ADR-003 RLS partial), GAP-01-008 (test count REPORTED only).

### PRJ-02 Aegis Agent ✓ Phase 3 Complete
- **Source:** https://github.com/aminazimi42-coder/aegis-agent — HEAD `69f159771c35b04b250fd282189f99b06de0c76d`
- **Phase 3 case file:** [cases/02-aegis-agent/engineering-evidence.md](../../cases/02-aegis-agent/engineering-evidence.md)
- **Phase 1 claims:** CLM-02-001 through CLM-02-004 in [claims.csv](claims.csv)
- **Phase 3 evidence:** EVD-02-001 through EVD-02-025 in [cases/02-aegis-agent/engineering-evidence.md §12](../../cases/02-aegis-agent/engineering-evidence.md#12-evidence-matrix)
- **GAP-02-001 (CRITICAL):** All three CI workflows (CI, Release, Security) failing at HEAD `69f159771c35` (2026-10-02). Root cause not inspected. 725-passing-tests claim in RELEASES.md is REPORTED, not independently verified.
- **GAP-02-002 (HIGH):** No published GitHub releases; 2 tags (v1.0.0-rc2, v0.6.0-a) — version ambiguity (rc1 in pyproject.toml vs rc2 tag).
- **Architecture:** Local-first multi-agent cognitive twin; 6 specialist agents (Alina/Kian/Bita/Aylin/Ahmad/Amin) with keyword-based routing; human approval gate at 4 layers (risk classification L0–L3, HumanAuthority 4-level evaluator, hash-bound SHA-256 approve, tenant binding); EchoProvider default LLM with optional HttpProvider (env-gated, 8s timeout, Echo fallback); `complete_safe` gateway with tool allow-list, forbidden-claims rejection, secret redaction, quota enforcement; optional Fernet encryption at rest (PBKDF2 480K iterations); session idle lock with Keychain PIN; JSONL audit with correlation IDs; Helm chart for K8s; Render demo explicitly not the product.
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 not reached — all CI workflows failing at HEAD.
- **Remaining gaps:** GAP-02-001 (CRITICAL — triple CI failure), GAP-02-002 (HIGH — no published release, version ambiguity), GAP-02-003 (MEDIUM — license NOASSERTION), GAP-02-004 (MEDIUM — heuristic risk evaluation), GAP-02-005 (LOW — monolithic server.py), GAP-02-006 (LOW — runtime sandbox stub).

### PRJ-03 IBM Supply Chain Challenge ✓ Phase 3 Complete
- **Source:** https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026 — HEAD `d00c5ebb27948ce9e61cd63add127868f65e36dd`
- **Phase 3 case file:** [cases/03-supply-chain-challenge-ibm-2026/engineering-evidence.md](../../cases/03-supply-chain-challenge-ibm-2026/engineering-evidence.md)
- **Phase 1 claims:** CLM-03-001 through CLM-03-004 in [claims.csv](claims.csv)
- **Phase 3 evidence:** EVD-03-001 through EVD-03-015 in [claims.csv](claims.csv)
- **GAP-03-001 RESOLVED:** SHA `e39d3b1df0b9ee2d711f2f7eff98df7c888e5d139` OBSERVED in commit log (Phase 3). Status upgraded from BLOCKED to CONDITIONAL.
- **OD-01 RESOLVED:** IBM SHA verification unblocked in Phase 3 — SHA exists, message "Align SQLAlchemy enums with PostgreSQL values".
- **Architecture:** 4-layer clean architecture (Router→Service→Repository→Model), multi-tenant org scoping via JWT dependency injection, pluggable AI module interfaces, cross-dialect UUID TypeDecorator (ADR-03-001), SQLAlchemy ENUM migration alignment (ADR-03-002).
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 not reached — no CI run artifact. 69-test claim REPORTED/CONDITIONAL (SHA confirmed; independent execution NOT TESTED).
- **Remaining gaps:** GAP-03-002 (no CI), GAP-03-003 (expiry test), GAP-03-004 (role coverage), GAP-03-005 (Docker), GAP-03-006 (R3 reproduction).

### PRJ-04 AlinaMatrix Enterprise ✓ Phase 4 Complete
- **Source:** https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise — HEAD `e7cc7df20164f2806e93bed6f5a665865e72133b`
- **Phase 4 case file:** [cases/04-alina-matrix-enterprise/engineering-evidence.md](../../cases/04-alina-matrix-enterprise/engineering-evidence.md)
- **Phase 1 claims:** CLM-04-001 through CLM-04-003 in [claims.csv](claims.csv)
- **Phase 4 evidence:** EVD-04-001 through EVD-04-030 in [cases/04-alina-matrix-enterprise/engineering-evidence.md §16](../../cases/04-alina-matrix-enterprise/engineering-evidence.md#16-evidence-matrix)
- **GAP-04-001 RESOLVED:** DeterministicFakeProvider boundary is ENFORCED in code — pure function with SHA-256 seed from pinned hashes; no network path; no randomness. Three distinct providers (DeterministicFakeProvider, PinnedModelProvider, LiveModelProvider) with distinct invariants. LiveModelProvider is env-gated + budget-enforced + schema-validated. ADR-0003 R07 directive enforced. Status: RESOLVED (was HIGH).
- **Architecture:** Four-agent M03 document-engineering pipeline (AlinaArchitect/AlinaGuard/AlinaDocEngine/AlinaOptimizer) with frozen capabilities and unconditional release/export block; 17-state workflow machine with enforced legal transitions; PostgreSQL 16 RLS with FORCE + fail-closed (unset tenant_id → no rows); server-derived UUID-validated tenant context; four-eyes approval enforced at API boundary; 8-condition release gate; Merkle provenance ledger (SHA-256, sorted leaves, tamper detection); append-only audit via RLS; revocation as event (not published-byte mutation); bigint budget enforcement (past 2^53); guard validates M03 draft against schema + quote-lock + 4 soft rules; negative evidence recording (silence = violation); export with watermark-intact verification (4 named meta tags); 14 forward-only SQL migrations; 12-threat model (v0.1, all open at Phase 1, 6 with unit test evidence); expert acceptance protocol with all blank scores; case study is a truthful blocker (no real customer); status exactly "Enterprise Candidate — Active Development."
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 partial — CI run SUCCESS at HEAD `e7cc7df20164` observed via GitHub Actions API (run #43). Not independently reproduced.
- **Remaining gaps:** GAP-04-002 (version 1.0.0 not anchored to release), GAP-04-003 (938 test count REPORTED not independently re-counted), GAP-04-004 (determinism invariant is stated policy not proven fact — Phase 8 replay test NOT TESTED), GAP-04-005 (threat model v0.1, 6/12 threats with unit evidence), GAP-04-006 (security unit-level only, no penetration testing), GAP-04-007 (no regional disaster recovery), GAP-04-008 (expert acceptance protocol all blank — no expert review performed).

### PRJ-05 Kian Trading Intelligence ✓ Phase 4 Complete
- **Source:** https://github.com/aminbita162-glitch/kian-trading-intelligence — HEAD `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb`
- **Phase 4 case file:** [cases/05-kian-trading-intelligence/engineering-evidence.md](../../cases/05-kian-trading-intelligence/engineering-evidence.md)
- **Phase 1 claims:** CLM-05-001 through CLM-05-002 in [claims.csv](claims.csv)
- **Phase 4 evidence:** EVD-05-001 through EVD-05-030 in [cases/05-kian-trading-intelligence/engineering-evidence.md §16](../../cases/05-kian-trading-intelligence/engineering-evidence.md#16-evidence-matrix)
- **GAP-05-001 RESOLVED:** LIVE mode enforcement is ENFORCED in code — `OperatingModeConfig` singleton defaults to `SIMULATION` with `live_authorized=False`; `set_mode(LIVE)` without `live_authorized` raises `PermissionError`; `is_live` is True only when both `mode==LIVE` and `live_authorized==True`. Status: RESOLVED (was HIGH).
- **Architecture:** Independent deterministic Risk Kernel (AD-004, 750 lines, 9 sequential safety checks, thread-safe); execution engine enforces "no order reaches adapter without risk authorization"; `OperatingModeConfig` simulation-only guard; signed risk tokens (AD-036, HMAC-SHA256, 30s TTL, nonce replay protection); trust-domain separation (kernel key ≠ vault key); graduated autonomy L0–L5 with independent kill plane (AD-038); `E9OwnerReviewPackage.live_trading_authorized` invariant `False`; formal TLA+ models for order/session/reservation/SAFE_HALT; simulation code outside deployable tree (E3).
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 partially — CI run SUCCESS at HEAD `54ef6ca4d8fd` (OBSERVED via GitHub Actions API). Not independently reproduced.
- **Remaining gaps:** GAP-05-002 (`test-house/financial-safety/` empty), GAP-05-003 (CI recently fixed, 2 prior failures), GAP-05-004 (2003 test count REPORTED, not independently re-counted), GAP-05-005 (no coverage threshold in CI), GAP-05-006 (no published releases/tags), GAP-05-007 (no formal STRIDE threat model), GAP-05-008 (emergency stop deactivation is caller responsibility), GAP-05-009 (kill plane in-process only), GAP-05-010 (HMAC symmetric only), GAP-05-011 (license PROPRIETARY — no source copied), GAP-05-012 (`formal_models` PLANNED), GAP-05-013 (strategy_validation_gate/agent_runtime_mesh/decision_ledger PLANNED).

### PRJ-06 Ahmad & Amin Technology ✓ Phase 6 Complete
- **Source:** https://github.com/aminbita162-glitch/ahmad-amin-technology — HEAD `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6`
- **Phase 6 case file:** [cases/06-ahmad-amin-technology/engineering-evidence.md](../../cases/06-ahmad-amin-technology/engineering-evidence.md)
- **Phase 1 claims:** CLM-06-001 through CLM-06-002 in [claims.csv](claims.csv)
- **Phase 6 evidence:** EVD-06-001 through EVD-06-020 in [cases/06-ahmad-amin-technology/engineering-evidence.md §10](../../cases/06-ahmad-amin-technology/engineering-evidence.md#10-evidence-matrix)
- **Architecture:** Private two-person Next.js 15 app (Amin/Germany, Ahmad/Iran); HMAC-SHA256 whitelist session cookie (httpOnly, 18h max age); production guard (503 if DEMO_MODE=true or SESSION_SECRET < 32 chars); Supabase offline gate (isSupabaseConfigured() — all 3 env vars, returns offline:true when absent, never fakes); OpenAI model room server-only (offline:true when OPENAI_API_KEY empty, does NOT imitate); rate limit 30/hr per handle (in-memory Map); polling fallback 3000ms (shouldPoll pure function, 5 unit tests); service worker (network-first GET only, POST/​/api/chat never cached); health route booleans only (no secret values); Meridian Seal hand-written CSS (no Tailwind, ink #54656f/paper #efeae2/amin #00a884/ahmad #cf9466/fault #e0635a); 44 files, 6653 lines TS/TSX/CSS; phase seal system (Phases 1-7, R7); release seal BLOCKED (OPENAI_API_KEY empty, 3 Vercel blockers).
- **Evidence maturity:** R1 (source inspected via local clone at pinned SHA). R2 not achieved — no CI, 5 unit tests only.
- **Remaining gaps:** GAP-06-001 (HIGH — no CI), GAP-06-002 (MEDIUM — no releases), GAP-06-003 (MEDIUM — release seal BLOCKED), GAP-06-004 (MEDIUM — Supabase not live), GAP-06-005 (LOW — NOASSERTION license), GAP-06-006 (LOW — in-memory rate limit), GAP-06-007 (LOW — 5 tests only), GAP-06-008 (LOW — OPENAI_API_KEY absent).

### PRJ-07 NimaOS AI Agent ✓ Phase 6 Complete
- **Source:** https://nimaos-ai-agent.vercel.app/translator — NO GITHUB REPO
- **Phase 6 case file:** [cases/07-nimaos-ai-agent/engineering-evidence.md](../../cases/07-nimaos-ai-agent/engineering-evidence.md)
- **Phase 1 claims:** CLM-07-001 through CLM-07-002 in [claims.csv](claims.csv)
- **Phase 6 evidence:** EVD-07-001 through EVD-07-015 in [cases/07-nimaos-ai-agent/engineering-evidence.md §10](../../cases/07-nimaos-ai-agent/engineering-evidence.md#10-evidence-matrix)
- **Architecture:** Live Vercel personal agent interface; 7 navigation sections (Chat, Dashboard, Email, Calendar, Files, Desktop, Translator, Settings); translator explicitly mocked ("Translations are currently mocked. Connect Ollama with a translation model"); desktop actions explicitly mocked ("Desktop actions are currently mocked. For real execution, the macOS control worker must run locally"); Ollama listed as provider in settings (not connected); 5 agents, "ask before_action" security, audit log; no source repo found — all claims E0/REPORTED.
- **Evidence maturity:** R0 — no source artifact; live URL content only (HTTP 200 on 2026-10-18).
- **Remaining gaps:** GAP-07-001 (CRITICAL — no source repository), GAP-07-002 (HIGH — architecture INFERRED from live URL), GAP-07-003 (MEDIUM — mock/live UNKNOWN on 4/7 surfaces), GAP-07-004 (MEDIUM — no auth observed), GAP-07-005 (LOW — root URL returns error page), GAP-07-006 (LOW — "5/5 active" agents unverified).

### PRJ-08 Lieferkette Optimierungsplattform ✓ Phase 5 Complete
- **Source:** https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform — HEAD `91c61210e9134302f17a634c2b830cadadf5c4bc`
- **Phase 5 case file:** [cases/08-lieferkette-optimierungsplattform/engineering-evidence.md](../../cases/08-lieferkette-optimierungsplattform/engineering-evidence.md)
- **Phase 1 claims:** CLM-08-001 through CLM-08-002 in [claims.csv](claims.csv)
- **Phase 5 evidence:** EVD-08-001 through EVD-08-029 in [cases/08-lieferkette-optimierungsplattform/engineering-evidence.md §14](../../cases/08-lieferkette-optimierungsplattform/engineering-evidence.md#14-evidence-matrix)
- **Architecture:** FastAPI 0.8.1 logistics decision-support platform; user-scoped data (owner_id filtering); heuristic decision services (moving-average forecast, nearest-neighbor routing, Euclidean warehouse allocation, safety stock with z-values + inverse-erf); 14 Alembic migrations; deterministic offline demand simulation (seed=42, negative binomial); non-deterministic online forecast (np.random.normal without seed); hardcoded JWT secret in 4 files; permissive CORS; DATABASE_URL required at import.
- **Evidence maturity:** R1 (source inspected at pinned SHA). R2 partial — CI run SUCCESS at HEAD `91c61210e913` (run #30228690474, 2026-07-27). 4 consecutive CI runs all SUCCESS. Not independently reproduced.
- **Remaining gaps:** GAP-08-001 (MEDIUM — 1 test file, 4 tests, 31 source files untested), GAP-08-002 (HIGH — hardcoded JWT secret), GAP-08-003 (HIGH — no license), GAP-08-004 (MEDIUM — non-deterministic forecast), GAP-08-005 (MEDIUM — permissive CORS), GAP-08-006 (LOW — no releases), GAP-08-007 (LOW — Euclidean not haversine), GAP-08-008 (LOW — nearest-neighbor no quality bound), GAP-08-009 (MEDIUM — no SQLite fallback), GAP-08-010 (LOW — hardcoded dashboard API_BASE), GAP-08-011 (LOW — duplicated get_current_user), GAP-08-012 (LOW — non-deterministic demand_simulator).

### PRJ-09 Personal AI Assistant ✓ Phase 5 Complete
- **Source:** https://github.com/aminbita162-glitch/personal-ai-assistant — HEAD `455a3ebb4f84a6ca648d7a8481f864b21680f235`
- **Phase 5 case file:** [cases/09-personal-ai-assistant/engineering-evidence.md](../../cases/09-personal-ai-assistant/engineering-evidence.md)
- **Phase 1 claims:** CLM-09-001 in [claims.csv](claims.csv)
- **Phase 5 evidence:** EVD-09-001 through EVD-09-025 in [cases/09-personal-ai-assistant/engineering-evidence.md §11](../../cases/09-personal-ai-assistant/engineering-evidence.md#11-evidence-matrix)
- **Architecture:** Flask 3.0.3 MVP; psycopg2 raw SQL; OpenAI integration (gpt-4o-mini, task extraction, smart decision, voice transcription via Whisper); bcrypt password hashing; bearer token auth (secrets.token_hex(32), plaintext in DB); user-scoped task/appointment queries; runtime schema init (CREATE TABLE IF NOT EXISTS); no tests, no CI, no license, no audit trail, no rate limiting; unauthenticated AI endpoints (/ai, /ai-browser); permissive CORS; stale (~6 months).
- **Evidence maturity:** R1 (source inspected at pinned SHA). R2 not achieved — no CI, no tests. All claims source-level only.
- **Remaining gaps:** GAP-09-001 (HIGH — no tests), GAP-09-002 (HIGH — no license), GAP-09-003 (HIGH — no CI), GAP-09-004 (MEDIUM — unauthenticated AI), GAP-09-005 (MEDIUM — plaintext auth token), GAP-09-006 (MEDIUM — no token expiry), GAP-09-007 (MEDIUM — permissive CORS), GAP-09-008 (MEDIUM — no rate limiting), GAP-09-009 (MEDIUM — no audit trail), GAP-09-010 (LOW — no connection pooling), GAP-09-011 (LOW — no migration framework), GAP-09-012 (LOW — stale), GAP-09-013 (LOW — no exchange rate caching), GAP-09-014 (LOW — duplicated auth), GAP-09-015 (LOW — services/services/ nested dir).

### PRJ-10 Restaurant AI Platform ✓ Phase 4 Complete
- **Source:** https://github.com/aminbita162-glitch/restaurant-ai-platform — HEAD `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa`
- **Phase 4 case file:** [cases/10-restaurant-ai-platform/engineering-evidence.md](../../cases/10-restaurant-ai-platform/engineering-evidence.md)
- **Phase 1 claims:** CLM-10-001 through CLM-10-002 in [claims.csv](claims.csv)
- **Phase 4 evidence:** EVD-10-001 through EVD-10-030 in [cases/10-restaurant-ai-platform/engineering-evidence.md §19](../../cases/10-restaurant-ai-platform/engineering-evidence.md#19-evidence-matrix)
- **GAP-10-001 (MEDIUM):** CI scope is "smoke" — only `test_fail_closed.py` (4 of 43 tests) is CI-enforced. 39/43 test functions are NOT run in CI. Status: OPEN.
- **Architecture:** 12-step pipeline orchestrator with shared context dict (PIPELINE_ORDER); day-of-week heuristic forecast (`method=dow_heuristic`, never `ml_model`); backtest quality gate (MAPE ≤ 50%, `labor_recommendation_allowed` gates staffing); fixed-ratio staffing rule (1 per 600 sales, `method=rule`, `approval=proposed`); 6 two-hour shift blocks per day; BOM inventory (`method=bom_rule`, no MRP, `suggested_purchase = max(0, total_qty - on_hand)`); waste as measured events (`method=measured` when present, `null` when absent — no ratio estimate); AI output schema gate (7 keys, 3 actions, degraded `method=rule` fallback on invalid); tenant isolation via application-layer query scoping (no DB-level RLS); `X-Api-Key` + optional `X-Tenant-Token` (env-mapped); per-tenant rate limit (5 req/60s, in-process); async job queue via Postgres (`FOR UPDATE SKIP LOCKED`); worker as separate process; `GET /pipeline/run` never executes; manager approval page (mobile HTML, `POST /manager/approve` requires auth); structured logging (`run_id`, `step`, `duration_ms`); MIT license; honest README with Implemented/Experimental/Planned matrix.
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 partial — CI run SUCCESS at HEAD `c49649a0b44e` (run #37066364111, 2026-10-02). 5 consecutive CI runs all SUCCESS. Not independently reproduced.
- **Remaining gaps:** GAP-10-001 (MEDIUM — CI scope smoke-only, 39/43 tests not CI-enforced), GAP-10-002 (MEDIUM — no published releases/tags), GAP-10-003 (LOW — forecast heuristic, 2% growth arbitrary), GAP-10-004 (LOW — staffing fixed-ratio, not optimizer), GAP-10-005 (MEDIUM — no DB-level tenant isolation), GAP-10-006 (LOW — no production observability), GAP-10-007 (LOW — GPT insight depends on external model), GAP-10-008 (MEDIUM — `_run_model_registry` edge case with invented `restaurant_001` default), GAP-10-009 (LOW — stale `execute=1&confirm=yes` reference in `expected_endpoints`), GAP-10-010 (LOW — CI deps minimal, no openai/psycopg2).

### PRJ-11 Büro Assistant ✓ Phase 5 Complete
- **Source:** https://github.com/aminbita162-glitch/buro-assistant — HEAD `893db187185dbed7b4dafd461241eb78acb07f8e`
- **Phase 5 case file:** [cases/11-buro-assistant/engineering-evidence.md](../../cases/11-buro-assistant/engineering-evidence.md)
- **Phase 1 claims:** CLM-11-001 through CLM-11-002 in [claims.csv](claims.csv)
- **Phase 5 evidence:** EVD-11-001 through EVD-11-030 in [cases/11-buro-assistant/engineering-evidence.md §15](../../cases/11-buro-assistant/engineering-evidence.md#15-evidence-matrix)
- **Architecture:** Self-hosted multi-tenant office mail desk; Phase 9 closed (all 9 build phases complete); 856 test functions across 28 test files (static source count); rules-before-model (tenant rule engine, first match wins, zero tokens on hit); redaction before model (5 PII patterns: IBAN/CARD/SSN/PHONE/EMAIL); shadow mode (auto-reply OFF by default); append-only audit with SHA-256 hash chain; human approval queue with tenant scoping; deterministic decision hash; sender auth (SPF/DKIM/DMARC, pass/fail/not_run); attachment quarantine + text extraction (never to model); semantic duplicate (Jaccard ≥ 0.85) + dissatisfied tone (26-term lexicon); air-gap Docker Compose (LOCAL_MODEL=1, no secrets); GDPR privacy data map (Art. 30); STRIDE threat model with mitigations; 50-row capability checklist; Argon2id password hashing; webhook HMAC-SHA256; 14 Alembic migrations; payment not live (fake adapter); external test script written, not run.
- **Evidence maturity:** R1 (source inspected at pinned SHA). R2 not achieved — no CI workflow (0 runs). Test count (856) by static source count (grep) — matches author's report. Not independently executed.
- **Remaining gaps:** GAP-11-001 (MEDIUM — no CI), GAP-11-002 (MEDIUM — no releases/tags), GAP-11-003 (LOW — proprietary license), GAP-11-004 (MEDIUM — external test not run), GAP-11-005 (LOW — regex redaction limitations), GAP-11-006 (LOW — hash chain doesn't detect deletion), GAP-11-007 (LOW — Jaccard word-set limitations), GAP-11-008 (MEDIUM — OpenAI third-party data residency), GAP-11-009 (LOW — monolithic main.py 49KB), GAP-11-010 (LOW — tests not independently executed).

### PRJ-12 Patch-Auri ✓ Phase 6 Complete
- **Source:** https://github.com/aminbita162-glitch/patch-auri — HEAD `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14`
- **Phase 6 case file:** [cases/12-patch-auri/engineering-evidence.md](../../cases/12-patch-auri/engineering-evidence.md)
- **Phase 1 claims:** CLM-12-001 through CLM-12-002 in [claims.csv](claims.csv)
- **Phase 6 evidence:** EVD-12-001 through EVD-12-020 in [cases/12-patch-auri/engineering-evidence.md §10](../../cases/12-patch-auri/engineering-evidence.md#10-evidence-matrix)
- **Architecture:** Non-medical informational NFC + color-guidance prototype; FastAPI monolithic main.py (22,825 chars); 3-level classification (Normal/Monitor/Consult) with per-level non-medical disclaimers; SHA-256 password hashing (unsalted — not a password hashing function); cookie auth (patch_auri_user, httponly, samesite=lax, no Secure flag); SQLite (patches.db) via SQLAlchemy; no NFC library in requirements.txt (NFC simulated via URL params); camera.html referenced but not in git tree (color analysis logic not inspectable); non-medical boundary is disclaimer-based, not code-enforced; 4 files only; no CI, no tests, no releases, no license.
- **Evidence maturity:** R1 (source inspected at pinned SHA via GitHub raw content). R2 not achieved — no CI, no tests.
- **Remaining gaps:** GAP-12-001 (LOW — 4-file repo), GAP-12-002 (MEDIUM — SHA-256 unsalted), GAP-12-003 (MEDIUM — no HTML escaping/XSS), GAP-12-004 (MEDIUM — no rate limiting), GAP-12-005 (LOW — no license), GAP-12-006 (LOW — no RGB range validation), GAP-12-007 (LOW — no Secure cookie flag), GAP-12-008 (LOW — non-medical boundary is disclaimer-based).

### PRJ-13 Azimi Innovation Lab Website ✓ Phase 6 Complete
- **Source:** https://azimi-innovation-lab-website.vercel.app/ — NO SOURCE REPO
- **Phase 6 case file:** [cases/13-azimi-innovation-lab-website/engineering-evidence.md](../../cases/13-azimi-innovation-lab-website/engineering-evidence.md)
- **Phase 1 claims:** CLM-13-001 through CLM-13-002 in [claims.csv](claims.csv)
- **Phase 6 evidence:** EVD-13-001 through EVD-13-015 in [cases/13-azimi-innovation-lab-website/engineering-evidence.md §9](../../cases/13-azimi-innovation-lab-website/engineering-evidence.md#9-evidence-matrix)
- **Architecture:** Vite SPA on Vercel; minimal HTML shell (1,615 chars) + JS bundle (453,536 chars); Persian primary (lang="fa" dir="rtl", Vazirmatn font); identity strings ("Amin Azimi · AI Systems Architect · Azimi Innovation Lab"); project links to AILORA, IBM Challenge, Aegis Agent in bundle; IBM non-endorsement disclaimer present; multilingual greeting (FA/EN/DE/UK/ES); Framer Motion animations; Open Graph + Twitter card meta tags; no source repository — all claims E0.
- **Evidence maturity:** R0 — no source artifact; live URL HTML + JS bundle content only (HTTP 200 on 2026-10-18).
- **Remaining gaps:** GAP-13-001 (HIGH — no source repository), GAP-13-002 (MEDIUM — SPA SEO limitation), GAP-13-003 (MEDIUM — external Google Fonts dependency), GAP-13-004 (LOW — 443 KB bundle), GAP-13-005 (LOW — Framer Motion weight), GAP-13-006 (LOW — no robots.txt/sitemap verified), GAP-13-007 (LOW — website repo unknown per DIRECTIVE.txt).

---

## 4. Open Decisions Requiring Owner Input

These were recorded in Phase 1 (`reviewer-notes.md §11`) and remain unresolved. They affect Phase 3 execution.

| ID | Topic | Status |
|----|-------|--------|
| OD-01 | IBM SHA `e39d3b1df0b9ee` verification (BLOCKED by rate limit) | **OPEN** |
| OD-02 | Aegis rc1 label — README version string only, no release tag | **OPEN** |
| OD-03 | Source code excerpt rights for no-license repos (PRJ-04, 08, 09, 12) | **OPEN** |
| OD-04 | CI failures at HEAD for PRJ-01 and PRJ-02 — known regression? | **OPEN** |
| OD-05 | NimaOS source repo — private/unlisted, or intentionally source-closed? | **OPEN** (Phase 6: searched — 6 unrelated repos found, none owned by portfolio owner; source remains NOT_FOUND) |

**Decisions resolved in Phase 3:**

| ID | Resolution |
|----|-----------|
| OD-01 | SHA `e39d3b1df0b9ee2d711f2f7eff98df7c888e5d139` OBSERVED in commit log. Message: "Align SQLAlchemy enums with PostgreSQL values". CLM-03-001 upgraded to CONDITIONAL. |
| OD-04 (PRJ-01) | CI failure at HEAD `46ca4a939bee` OBSERVED (run `33390090146`). Root cause not inspected. A qualifying CI run `33025990734` at SHA `9571b168` (2026-08-27) is SUCCESS and is referenced in `docs/verification.md`. This is a freeze-commit regression. Disclosed in portfolio case. AILORA has no claim of CI-passing at HEAD; freeze commit is expected to differ from the qualifying run SHA. |
| OD-02 | Aegis rc1 label is source-file-only (`pyproject.toml`, `RELEASES.md`, `system-manifest.yaml`). No published GitHub release confirms it. Tag `v1.0.0-rc2` exists but is not a release — version ambiguity documented as GAP-02-002. Status: OPEN — no release artifact resolves the version claim. |

**Remaining open decisions** (affect later phases):

| ID | Topic | Status |
|----|-------|--------|
| OD-03 | Source code excerpt rights for no-license repos (PRJ-04, 08, 09, 12) | **OPEN** |
| OD-04 (PRJ-02) | CI failures at HEAD for PRJ-02 Aegis — all 3 workflows failing; root cause not inspected | **OPEN** |
| OD-05 | NimaOS source repo — private/unlisted, or intentionally source-closed? | **OPEN** (Phase 6: searched — 6 unrelated repos found, none owned by portfolio owner; source remains NOT_FOUND) |

---

## 5. Phase Completion Status

| Phase | Description | Status | Commit SHA |
|-------|-------------|--------|-----------|
| 0 | Portfolio Baseline and Conflict Map | COMPLETE (read-only) | N/A |
| 1 | Source Traceability Inventory | COMPLETE | `3270b21a350eafa3a152b750faaafbca47756785` |
| 2 | Portfolio Evidence Foundation | COMPLETE | `1315435dXXXXXX` (TBD after Phase 3 commit) |
| 3 | Flagship Deep Engineering Cases — PRJ-03 IBM | COMPLETE | `4810335126309abf64278576c35cc5744a98b263` |
| 3 | Flagship Deep Engineering Cases — PRJ-01 AILORA | COMPLETE | TBD — this phase |
| 3 | Flagship Deep Engineering Cases — PRJ-02 Aegis | COMPLETE | `281f30d9022a13793a78575767c33c87f9b10925` |
| 4 | High-Risk Engineering Cases — PRJ-05 Kian | COMPLETE | `3bbf086613c0d7d037ec6dc7fb0e0857f35dec87` |
| 4 | High-Risk Engineering Cases — PRJ-04 AlinaMatrix | COMPLETE | `f1eed692672b5fd93345d4ad844e530fcd406320` |
| 4 | High-Risk Engineering Cases — PRJ-10 Restaurant | COMPLETE | `12fe7266062e4d904d6cbac3c92e9eb042469a6b` |
| 5 | Domain-Breadth Cases — PRJ-08 Logistics | COMPLETE | (this phase) |
| 5 | Domain-Breadth Cases — PRJ-11 Büro Assistant | COMPLETE | (this phase) |
| 5 | Domain-Breadth Cases — PRJ-09 Personal AI | COMPLETE | `12fe7266062e4d904d6cbac3c92e9eb042469a6b` |
| 6 | Remaining Cases — PRJ-06 Ahmad & Amin | COMPLETE | (this phase) |
| 6 | Remaining Cases — PRJ-07 NimaOS | COMPLETE | (this phase) |
| 6 | Remaining Cases — PRJ-12 Patch-Auri | COMPLETE | (this phase) |
| 6 | Remaining Cases — PRJ-13 Website | COMPLETE | (this phase) |
| 7 | Technical Reviews, Education, Legacy Assets | COMPLETE | `2ab932c093c1aaf8bc5947c80355b06d8402d4ea` |
| 8 | Cross-Project Architecture and Engineering Synthesis | COMPLETE | (this phase) |
| 9 | Portfolio Quality Gate | NOT STARTED — AWAITING OWNER | — |
| 10 | Owner-Approved Publication | NOT STARTED — AWAITING OWNER | — |

---

## 6. Index Version History

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-10-17 | Initial index, created in Phase 2 |
| 1.1 | 2026-10-17 | Phase 3 PRJ-03 update: GAP-03-001 resolved, EVD-03-001..015 added, OD-01 resolved, PRJ-03 evidence maturity updated |
| 1.2 | 2026-10-17 | Phase 3 PRJ-01 update: EVD-01-001..017 added, GAP-01-001..010 recorded, OD-04 (PRJ-01) resolved, PRJ-01 evidence maturity updated to R2 |
| 1.3 | 2026-10-17 | Phase 3 PRJ-02 update: EVD-02-001..025 added, GAP-02-001..006 recorded, PRJ-02 evidence maturity confirmed R1, all three flagship cases complete |
| 1.4 | 2026-10-17 | Phase 4 PRJ-05 update: EVD-05-001..030 added, GAP-05-001 resolved (LIVE mode enforcement is ENFORCED in code), GAP-05-002..013 recorded, PRJ-05 evidence maturity confirmed R1 (R2 partial — CI SUCCESS at HEAD) |
| 1.5 | 2026-10-17 | Phase 4 PRJ-04 update: EVD-04-001..030 added, GAP-04-001 resolved (DeterministicFakeProvider boundary is ENFORCED in code — pure function, no network path, three distinct providers), GAP-04-002..008 recorded, PRJ-04 evidence maturity confirmed R1 (R2 partial — CI SUCCESS at HEAD) |
| 1.6 | 2026-10-17 | Phase 4 PRJ-10 update: EVD-10-001..030 added, GAP-10-001..010 recorded (MEDIUM — CI scope smoke-only; MEDIUM — no releases; MEDIUM — _run_model_registry edge case with invented defaults), CLM-10-001 upgraded REPORTED→OBSERVED (directive-driven development confirmed), PRJ-10 evidence maturity confirmed R1 (R2 partial — CI SUCCESS at HEAD, 5 consecutive runs) |
| 1.7 | 2026-10-18 | Phase 5 PRJ-08 update: EVD-08-001..029 added, GAP-08-001..012 recorded (HIGH — hardcoded JWT secret; HIGH — no license; MEDIUM — 1 test file), CLM-08-001 confirmed PASS (4 CI runs SUCCESS), CLM-08-002 upgraded REPORTED→OBSERVED (heuristic routing confirmed), PRJ-08 evidence maturity confirmed R1 (R2 partial — CI SUCCESS at HEAD, 4 consecutive runs) |
| 1.8 | 2026-10-18 | Phase 5 PRJ-11 update: EVD-11-001..030 added, GAP-11-001..010 recorded (MEDIUM — no CI; MEDIUM — external test not run; MEDIUM — OpenAI data residency), CLM-11-001 upgraded REPORTED→OBSERVED (Phase 9 closed confirmed), CLM-11-002 upgraded OBSERVED/NOT_TESTED→OBSERVED (decision_hash + send_decision inspected), PRJ-11 evidence maturity confirmed R1 (856 tests by static source count; no CI; not independently executed) |
| 1.9 | 2026-10-18 | Phase 5 PRJ-09 update: EVD-09-001..025 added, GAP-09-001..015 recorded (HIGH — no tests; HIGH — no license; HIGH — no CI; MEDIUM — unauthenticated AI; MEDIUM — plaintext auth token), CLM-09-001 upgraded REPORTED→OBSERVED (OpenAI integration confirmed at source level), PRJ-09 evidence maturity confirmed R1 (no CI, no tests — all claims source-level only) |
| 2.0 | 2026-10-18 | Phase 6 PRJ-06 update: EVD-06-001..020 added, GAP-06-001..008 recorded (HIGH — no CI; MEDIUM — no releases; MEDIUM — release seal BLOCKED; MEDIUM — Supabase not live), CLM-06-001 upgraded REPORTED/NOT_TESTED→OBSERVED/CONDITIONAL (Phase 1 seal observed, no CI), CLM-06-002 confirmed OBSERVED/PASS (Supabase offline gate confirmed), PRJ-06 evidence maturity confirmed R1 (source inspected via local clone; 5 unit tests; no CI) |
| 2.1 | 2026-10-18 | Phase 6 PRJ-07 update: EVD-07-001..015 added, GAP-07-001..006 recorded (CRITICAL — no source; HIGH — architecture INFERRED; MEDIUM — mock/live UNKNOWN on 4/7 surfaces), CLM-07-001 upgraded REPORTED/NOT_TESTED→OBSERVED/PASS (translator mock confirmed via HTTP), CLM-07-002 confirmed OBSERVED/BLOCKED (no source repo found via GitHub search), OD-05 updated (6 unrelated repos found, source NOT_FOUND), PRJ-07 evidence maturity R0 (live URL only, no source) |
| 2.2 | 2026-10-18 | Phase 6 PRJ-12 update: EVD-12-001..020 added, GAP-12-001..008 recorded (MEDIUM — SHA-256 unsalted; MEDIUM — no HTML escaping/XSS; MEDIUM — no rate limiting; LOW — no license), CLM-12-001 confirmed OBSERVED/PASS (non-medical disclaimers on all 3 levels), CLM-12-002 upgraded REPORTED/NOT_TESTED→OBSERVED/CONDITIONAL (NFC not in requirements.txt, simulated via URL params; camera.html not in tree), PRJ-12 evidence maturity confirmed R1 (4-file repo fully inspected via raw content; no CI, no tests) |
| 2.3 | 2026-10-18 | Phase 6 PRJ-13 update: EVD-13-001..015 added, GAP-13-001..007 recorded (HIGH — no source; MEDIUM — SPA SEO; MEDIUM — external Google Fonts), CLM-13-001 upgraded REPORTED/NOT_TESTED→OBSERVED/PASS (HTTP 200 confirmed), CLM-13-002 confirmed OBSERVED/BLOCKED (no source repo disclosed), PRJ-13 evidence maturity R0 (live URL + JS bundle only, no source). All 13 cases now complete through Phase 6. |
| 2.4 | 2026-10-18 | Phase 7 update: reviews/ index updated (6 authored evaluation files — 5 text-extractable, 1 image-only), courses/ indexes updated (2 series: senior-agentic-systems 31-page session 1; understanding-ai-generated-code 12 sessions, 122 pages), projects/ legacy assessment created (5 inherited directories — 2 with real implementations, 3 standards/skeleton). All originals preserved unmodified. No instructor answer keys or proprietary content reproduced. Authored opinions distinguished from independent validation. Phase 7 marked COMPLETE in phase status table. |
| 2.5 | 2026-10-18 | Phase 8 update: cross-project architecture and engineering synthesis added (`docs/engineering/cross-project-architecture.md`). Synthesizes 12 cross-cutting themes (multi-agent orchestration, tenant isolation, deterministic vs live model separation, audit/provenance, human approval gates, security/trust boundaries, CI/testing maturity, observability, domain safety, failure handling, data flow, AI evaluation) across all 13 projects. 7 cross-project gaps (XGAP-01..07) recorded. Phase 8 marked COMPLETE in phase status table. All 13 source projects remain read-only. |
