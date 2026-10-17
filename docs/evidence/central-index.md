# Central Evidence Index

**Phase:** 4 — High-Risk Engineering Cases (PRJ-10 Restaurant AI Platform — complete)
**Created:** 2026-10-17
**Phase 1 portfolio HEAD:** `0e8a84cd20322a48da0b722981d1e535394ee6c3`
**Phase 2 portfolio HEAD:** `3270b21a350eafa3a152b750faaafbca47756785`
**Phase 3 (PRJ-03) portfolio HEAD:** `4810335126309abf64278576c35cc5744a98b263`
**Phase 3 (PRJ-01) portfolio HEAD:** `d3ba70b3a46fdabf2751525f4d835a6738a435d9`
**Phase 3 (PRJ-02) portfolio HEAD:** `281f30d9022a13793a78575767c33c87f9b10925`
**Phase 4 (PRJ-05) portfolio HEAD:** `3bbf086613c0d7d037ec6dc7fb0e0857f35dec87`
**Phase 4 (PRJ-04) portfolio HEAD:** `f1eed692672b5fd93345d4ad844e530fcd406320`
**Independence:** SECOND_AGENT_REVIEW (Hermes) — not independent third-party review
**Status:** Authoritative navigation index for all portfolio evidence.

This file is the single entry point for a reviewer navigating from a portfolio claim to its evidence. It summarises the 13 projects, links all evidence files, and shows the current observation maturity for each project. Deep-dive case evidence is added in Phases 3–8.

---

## How to Navigate

**5-minute reviewer:** Read the [Project Status Summary](#2-project-status-summary) table. Check the `evidence_maturity` column. Follow a specific project row to its source SHA and Phase 1 gaps.

**15-minute reviewer:** Follow links in the [Per-Project Evidence Map](#3-per-project-evidence-map) for flagship cases (PRJ-01, PRJ-02, PRJ-03). Read the linked `reviewer-notes.md` sections for CI findings and critical gaps.

**60-minute reviewer:** Start with [schema.md](schema.md) to understand ID conventions, then [provenance-rules.md](provenance-rules.md) for how observations were made. Cross-reference `claims.csv` claim IDs against `reviewer-notes.md` gap IDs. Phase 3+ case files will add `EVD-` and `ADR-` records.

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
| PRJ-06 | Ahmad & Amin Technology | PUBLIC | `fbbfc307fd98` | R1 | NO CI | MINIMAL | NOASSERTION | GAP-06-001 (HIGH) | PHASE 6 |
| PRJ-07 | NimaOS AI Agent | **NO SOURCE** | NOT_AVAILABLE | R0 | NO CI | NONE | UNKNOWN | GAP-07-001 (LOW) | PHASE 6 |
| PRJ-08 | Lieferkette Optimierungsplattform | PUBLIC | `91c61210e913` | R1 | PASS | MINIMAL (1 file) | **NONE** | GAP-08-001 (MEDIUM) | PHASE 5 |
| PRJ-09 | Personal AI Assistant | PUBLIC | `455a3ebb4f84` | R1 | NO CI | NONE | **NONE** | GAP-09-001 (MEDIUM) | PHASE 5 |
| PRJ-10 | Restaurant AI Platform | PUBLIC | `c49649a0b44e` | R1 | PASS (smoke) | OBSERVED | MIT | GAP-10-001 (MEDIUM — CI scope smoke-only) | **COMPLETE** |
| PRJ-11 | Büro Assistant | PUBLIC | `893db187185d` | R1 | NO CI | OBSERVED | NOASSERTION | GAP-11-001 (MEDIUM) | PHASE 5 |
| PRJ-12 | Patch-Auri | PUBLIC | `93ea2196e7db` | R1 | NO CI | NONE | **NONE** | GAP-12-001 (LOW) | PHASE 6 |
| PRJ-13 | Azimi Innovation Lab Website | **WEBSITE ONLY** | NOT_AVAILABLE | R0 | N/A | N/A | UNKNOWN | GAP-13-001 (LOW) | PHASE 6 |

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
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 partial — CI run SUCCESS at HEAD `e7cc7df20164` observed via GitHub Actions API (run #43). Not independently reproduced by this reviewer.
- **Remaining gaps:** GAP-04-002 (version 1.0.0 not anchored to release), GAP-04-003 (938 test count REPORTED not independently re-counted), GAP-04-004 (determinism invariant is stated policy not proven fact — Phase 8 replay test NOT TESTED), GAP-04-005 (threat model v0.1, 6/12 threats with unit evidence), GAP-04-006 (security unit-level only, no penetration testing), GAP-04-007 (no regional disaster recovery), GAP-04-008 (expert acceptance protocol all blank — no expert review performed).

### PRJ-05 Kian Trading Intelligence ✓ Phase 4 Complete
- **Source:** https://github.com/aminbita162-glitch/kian-trading-intelligence — HEAD `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb`
- **Phase 4 case file:** [cases/05-kian-trading-intelligence/engineering-evidence.md](../../cases/05-kian-trading-intelligence/engineering-evidence.md)
- **Phase 1 claims:** CLM-05-001 through CLM-05-002 in [claims.csv](claims.csv)
- **Phase 4 evidence:** EVD-05-001 through EVD-05-030 in [cases/05-kian-trading-intelligence/engineering-evidence.md §16](../../cases/05-kian-trading-intelligence/engineering-evidence.md#16-evidence-matrix)
- **GAP-05-001 RESOLVED:** LIVE mode enforcement is ENFORCED in code — `OperatingModeConfig` singleton defaults to `SIMULATION` with `live_authorized=False`; `set_mode(LIVE)` without `live_authorized` raises `PermissionError`; `is_live` is True only when both `mode==LIVE` and `live_authorized==True`. Status: RESOLVED (was HIGH).
- **Architecture:** Independent deterministic Risk Kernel (AD-004, 750 lines, 9 sequential safety checks, thread-safe); execution engine enforces "no order reaches adapter without risk authorization"; `OperatingModeConfig` simulation-only guard; signed risk tokens (AD-036, HMAC-SHA256, 30s TTL, nonce replay protection); trust-domain separation (kernel key ≠ vault key); graduated autonomy L0–L5 with independent kill plane (AD-038); `E9OwnerReviewPackage.live_trading_authorized` invariant `False`; formal TLA+ models for order/session/reservation/SAFE_HALT; simulation code outside deployable tree (E3).
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 partially — CI run SUCCESS at HEAD `54ef6ca4d8fd` (OBSERVED via GitHub Actions API). Not independently reproduced by this reviewer.
- **Remaining gaps:** GAP-05-002 (`test-house/financial-safety/` empty), GAP-05-003 (CI recently fixed, 2 prior failures), GAP-05-004 (2003 test count REPORTED, not independently re-counted), GAP-05-005 (no coverage threshold in CI), GAP-05-006 (no published releases/tags), GAP-05-007 (no formal STRIDE threat model), GAP-05-008 (emergency stop deactivation is caller responsibility), GAP-05-009 (kill plane in-process only), GAP-05-010 (HMAC symmetric only), GAP-05-011 (license PROPRIETARY — no source copied), GAP-05-012 (`formal_models` PLANNED), GAP-05-013 (strategy_validation_gate/agent_runtime_mesh/decision_ledger PLANNED).

### PRJ-06 Ahmad & Amin Technology
- **Source:** https://github.com/aminbita162-glitch/ahmad-amin-technology — HEAD `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6`
- **Phase 1 claims:** CLM-06-001 through CLM-06-002 in [claims.csv](claims.csv)
- **High gap:** GAP-06-001 — "Phase 1 PASS" claim lacks CI or verifiable test artifacts ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Phase 6 priority:** MEDIUM — private two-person app; privacy and deployment evidence needed

### PRJ-07 NimaOS AI Agent
- **Source:** https://nimaos-ai-agent.vercel.app/translator — NO GITHUB REPO
- **Phase 1 claims:** CLM-07-001 through CLM-07-002 in [claims.csv](claims.csv)
- **Low gap:** GAP-07-001 — no source repository; all claims E0/REPORTED ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Phase 6 priority:** LOW — source-closed; only live URL evidence possible

### PRJ-08 Lieferkette Optimierungsplattform
- **Source:** https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform — HEAD `91c61210e9134302f17a634c2b830cadadf5c4bc`
- **Phase 1 claims:** CLM-08-001 through CLM-08-002 in [claims.csv](claims.csv)
- **Medium gap:** GAP-08-001 — very limited test coverage (1 file); no license ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Phase 5 priority:** MEDIUM — heuristic routing claim requires source inspection

### PRJ-09 Personal AI Assistant
- **Source:** https://github.com/aminbita162-glitch/personal-ai-assistant — HEAD `455a3ebb4f84a6ca648d7a8481f864b21680f235`
- **Phase 1 claims:** CLM-09-001 in [claims.csv](claims.csv)
- **Medium gap:** GAP-09-001 — stale (April 2026); no CI, no tests, no license ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Phase 5 priority:** MEDIUM — MVP status; connector permissions and scope require inspection

### PRJ-10 Restaurant AI Platform ✓ Phase 4 Complete
- **Source:** https://github.com/aminbita162-glitch/restaurant-ai-platform — HEAD `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa`
- **Phase 4 case file:** [cases/10-restaurant-ai-platform/engineering-evidence.md](../../cases/10-restaurant-ai-platform/engineering-evidence.md)
- **Phase 1 claims:** CLM-10-001 through CLM-10-002 in [claims.csv](claims.csv)
- **Phase 4 evidence:** EVD-10-001 through EVD-10-030 in [cases/10-restaurant-ai-platform/engineering-evidence.md §19](../../cases/10-restaurant-ai-platform/engineering-evidence.md#19-evidence-matrix)
- **GAP-10-001 (MEDIUM):** CI scope is "smoke" — only `test_fail_closed.py` (4 of 43 tests) is CI-enforced. 39/43 test functions are NOT run in CI. Status: OPEN.
- **Architecture:** 12-step pipeline orchestrator with shared context dict (PIPELINE_ORDER); day-of-week heuristic forecast (`method=dow_heuristic`, never `ml_model`); backtest quality gate (MAPE ≤ 50%, `labor_recommendation_allowed` gates staffing); fixed-ratio staffing rule (1 per 600 sales, `method=rule`, `approval=proposed`); 6 two-hour shift blocks per day; BOM inventory (`method=bom_rule`, no MRP, `suggested_purchase = max(0, total_qty - on_hand)`); waste as measured events (`method=measured` when present, `null` when absent — no ratio estimate); AI output schema gate (7 keys, 3 actions, degraded `method=rule` fallback on invalid); tenant isolation via application-layer query scoping (no DB-level RLS); `X-Api-Key` + optional `X-Tenant-Token` (env-mapped); per-tenant rate limit (5 req/60s, in-process); async job queue via Postgres (`FOR UPDATE SKIP LOCKED`); worker as separate process; `GET /pipeline/run` never executes; manager approval page (mobile HTML, `POST /manager/approve` requires auth); structured logging (`run_id`, `step`, `duration_ms`); MIT license; honest README with Implemented/Experimental/Planned matrix.
- **Evidence maturity:** R1 (source inspected at immutable SHA). R2 partial — CI run SUCCESS at HEAD `c49649a0b44e` (run #37066364111, 2026-10-02). 5 consecutive CI runs all SUCCESS. Not independently reproduced by this reviewer.
- **Remaining gaps:** GAP-10-001 (MEDIUM — CI scope smoke-only, 39/43 tests not CI-enforced), GAP-10-002 (MEDIUM — no published releases/tags), GAP-10-003 (LOW — forecast heuristic, 2% growth arbitrary), GAP-10-004 (LOW — staffing fixed-ratio, not optimizer), GAP-10-005 (MEDIUM — no DB-level tenant isolation), GAP-10-006 (LOW — no production observability), GAP-10-007 (LOW — GPT insight depends on external model), GAP-10-008 (MEDIUM — `_run_model_registry` edge case with invented `restaurant_001` default), GAP-10-009 (LOW — stale `execute=1&confirm=yes` reference in `expected_endpoints`), GAP-10-010 (LOW — CI deps minimal, no openai/psycopg2).

### PRJ-11 Büro Assistant
- **Source:** https://github.com/aminbita162-glitch/buro-assistant — HEAD `893db187185dbed7b4dafd461241eb78acb07f8e`
- **Phase 1 claims:** CLM-11-001 through CLM-11-002 in [claims.csv](claims.csv)
- **Medium gap:** GAP-11-001 — no release tag for "1.0.0"; `decision_hash.py` and `send_decision.py` not yet inspected ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `app/agents/decision_hash.py` (OBSERVED), `app/policy/send_decision.py` (OBSERVED), `docs/releases/commercial-phase-1..5` (OBSERVED)
- **Phase 5 priority:** MEDIUM — governance depth understated in portfolio case

### PRJ-12 Patch-Auri
- **Source:** https://github.com/aminbita162-glitch/patch-auri — HEAD `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14`
- **Phase 1 claims:** CLM-12-001 through CLM-12-002 in [claims.csv](claims.csv)
- **Low gap:** GAP-12-001 — 6-file repo; NFC/color implementation entirely REPORTED ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Phase 6 priority:** LOW — full repo can be inspected in a single pass

### PRJ-13 Azimi Innovation Lab Website
- **Source:** https://azimi-innovation-lab-website.vercel.app/ — NO SOURCE REPO
- **Phase 1 claims:** CLM-13-001 through CLM-13-002 in [claims.csv](claims.csv)
- **Low gap:** GAP-13-001 — no source repository; website-only ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Phase 6 priority:** LOW — URL accessibility and published claims are the only evidence possible

---

## 4. Open Decisions Requiring Owner Input

These were recorded in Phase 1 (`reviewer-notes.md §11`) and remain unresolved. They affect Phase 3 execution.

| ID | Topic | Status |
|----|-------|--------|
| OD-01 | IBM SHA `e39d3b1df0b9ee` verification (BLOCKED by rate limit) | **OPEN** |
| OD-02 | Aegis rc1 label — README version string only, no release tag | **OPEN** |
| OD-03 | Source code excerpt rights for no-license repos (PRJ-04, 08, 09, 12) | **OPEN** |
| OD-04 | CI failures at HEAD for PRJ-01 and PRJ-02 — known regression? | **OPEN** |
| OD-05 | NimaOS source repo — private/unlisted, or intentionally source-closed? | **OPEN** |

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
| OD-05 | NimaOS source repo — private/unlisted, or intentionally source-closed? | **OPEN** |

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
| 4 | High-Risk Engineering Cases — PRJ-10 Restaurant | COMPLETE | (this phase) |
| 5 | Domain-Breadth Cases | NOT STARTED — AWAITING OWNER | — |
| 6 | Remaining Cases and Website Reference | NOT STARTED — AWAITING OWNER | — |
| 7 | Technical Reviews, Education, Legacy Assets | NOT STARTED — AWAITING OWNER | — |
| 8 | Cross-Project Architecture and Engineering Synthesis | NOT STARTED — AWAITING OWNER | — |
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
