# Central Evidence Index

**Phase:** 2 — Portfolio Evidence Foundation  
**Created:** 2026-10-17  
**Phase 1 portfolio HEAD:** `0e8a84cd20322a48da0b722981d1e535394ee6c3`  
**Phase 2 portfolio HEAD:** `3270b21a350eafa3a152b750faaafbca47756785`  
**Independence:** SECOND_AGENT_REVIEW (IBM Bob) — not independent third-party review  
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

Phase 3+ will add per-project case evidence files at `cases/<case>/engineering-evidence.md` and reference `EVD-` IDs back to this index.

---

## 2. Project Status Summary

Columns: **evidence_maturity** = highest reproducibility level achieved (R0–R5, per [schema.md §6](schema.md#6-reproducibility-levels)). **ci_status** = latest observed CI run status at Phase 1 HEAD SHA. **test_evidence** = source-level test file presence (OBSERVED / NOT_OBSERVED). **phase_1_gaps** = count of named gaps in reviewer-notes.md. **deep_dive_phase** = earliest phase authorized for deep engineering case work.

| ID | Project | Source available | Phase 1 HEAD SHA | evidence_maturity | ci_status | test_evidence | license | phase_1_gaps | deep_dive_phase |
|----|---------|-----------------|-----------------|-------------------|-----------|---------------|---------|-------------|-----------------|
| PRJ-01 | AILORA | PUBLIC | `46ca4a939bee` | R1 | **FAIL** | OBSERVED | NOASSERTION | GAP-01-001 (CRITICAL) | PHASE 3 |
| PRJ-02 | Aegis Agent | PUBLIC | `69f159771c35` | R1 | **ALL FAIL** | OBSERVED | NOASSERTION | GAP-02-001 (CRITICAL) | PHASE 3 |
| PRJ-03 | IBM Supply Chain | PUBLIC | `d00c5ebb2794` | R1 | NO CI | OBSERVED | MIT | GAP-03-001 (CRITICAL) | PHASE 3 |
| PRJ-04 | AlinaMatrix Enterprise | PUBLIC | `e7cc7df20164` | R1 | PASS | OBSERVED | **NONE** | GAP-04-001 (HIGH) | PHASE 4 |
| PRJ-05 | Kian Trading Intelligence | PUBLIC | `54ef6ca4d8fd` | R1 | PASS (recently fixed) | OBSERVED | NOASSERTION | GAP-05-001 (HIGH) | PHASE 4 |
| PRJ-06 | Ahmad & Amin Technology | PUBLIC | `fbbfc307fd98` | R1 | NO CI | MINIMAL | NOASSERTION | GAP-06-001 (HIGH) | PHASE 6 |
| PRJ-07 | NimaOS AI Agent | **NO SOURCE** | NOT_AVAILABLE | R0 | NO CI | NONE | UNKNOWN | GAP-07-001 (LOW) | PHASE 6 |
| PRJ-08 | Lieferkette Optimierungsplattform | PUBLIC | `91c61210e913` | R1 | PASS | MINIMAL (1 file) | **NONE** | GAP-08-001 (MEDIUM) | PHASE 5 |
| PRJ-09 | Personal AI Assistant | PUBLIC | `455a3ebb4f84` | R1 | NO CI | NONE | **NONE** | GAP-09-001 (MEDIUM) | PHASE 5 |
| PRJ-10 | Restaurant AI Platform | PUBLIC | `c49649a0b44e` | R1 | PASS (smoke) | OBSERVED | MIT | GAP-10-001 (MEDIUM) | PHASE 4 |
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

### PRJ-01 AILORA
- **Source:** https://github.com/azimilab2025-ai/ailora — HEAD `46ca4a939bee0396e90abef91905b6d1b95d470c`
- **Phase 1 claims:** CLM-01-001 through CLM-01-005 in [claims.csv](claims.csv)
- **Critical gap:** GAP-01-001 — CI failing at HEAD ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `docs/qualification/status-contract.md` (OBSERVED), release tag `candidate-2026-08-29` (OBSERVED), `docs/qualification/final-release-manifest.json` (OBSERVED)
- **Phase 3 priority:** FLAGSHIP — deepest architecture and decision case work authorized in Phase 3

### PRJ-02 Aegis Agent
- **Source:** https://github.com/aminazimi42-coder/aegis-agent — HEAD `69f159771c35b04b250fd282189f99b06de0c76d`
- **Phase 1 claims:** CLM-02-001 through CLM-02-004 in [claims.csv](claims.csv)
- **Critical gap:** GAP-02-001 — all three CI workflows failing at HEAD; no release tag for rc1 claim ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `core/tool_tokens.py`, `core/decision_ledger.py`, `core/circuit_breaker.py`, `core/reversible_workflow.py` (all OBSERVED in tree — not yet inspected)
- **Phase 3 priority:** FLAGSHIP — human approval enforcement architecture requires source inspection

### PRJ-03 IBM Supply Chain Challenge
- **Source:** https://github.com/azimilab2025-ai/IBM-Bob-Challenge-2026 — HEAD `d00c5ebb27948ce9e61cd63add127868f65e36dd`
- **Phase 1 claims:** CLM-03-001 through CLM-03-004 in [claims.csv](claims.csv)
- **Critical gap:** GAP-03-001 — IBM commit SHA `e39d3b1df0b9ee` BLOCKED (rate limit); 69-test claim REPORTED/NOT_TESTED ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `docs/verification.md` (OBSERVED), `backend/tests/` (8+ test files, OBSERVED), `scripts/evaluate_ai.py` (OBSERVED), release tag `v1.0.0-submission-final`
- **Phase 3 priority:** FLAGSHIP — strongest release provenance; verification.md and test structure warrant deep inspection

### PRJ-04 AlinaMatrix Enterprise
- **Source:** https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise — HEAD `e7cc7df20164f2806e93bed6f5a665865e72133b`
- **Phase 1 claims:** CLM-04-001 through CLM-04-003 in [claims.csv](claims.csv)
- **High gap:** GAP-04-001 — no license; DeterministicFakeProvider boundary requires source inspection ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `docs/adr/` (3 ADRs OBSERVED), `docs/eval/expert-acceptance-protocol.md` (OBSERVED)
- **Phase 4 priority:** HIGH — AlinaMatrix fake/live provider separation is a key claim

### PRJ-05 Kian Trading Intelligence
- **Source:** https://github.com/aminbita162-glitch/kian-trading-intelligence — HEAD `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb`
- **Phase 1 claims:** CLM-05-001 through CLM-05-002 in [claims.csv](claims.csv)
- **High gap:** GAP-05-001 — LIVE mode enforcement requires `risk_kernel/kernel.py` inspection; `test-house/financial-safety` is empty ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `services/risk_kernel/kernel.py` (OBSERVED in tree), `packages/contracts/risk.py` (OBSERVED), `test-house/phase-01..07/` evidence files
- **Phase 4 priority:** HIGH — simulation-only enforcement is a critical safety claim

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

### PRJ-10 Restaurant AI Platform
- **Source:** https://github.com/aminbita162-glitch/restaurant-ai-platform — HEAD `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa`
- **Phase 1 claims:** CLM-10-001 through CLM-10-002 in [claims.csv](claims.csv)
- **Medium gap:** GAP-10-001 — Smoke CI only; `test_fail_closed.py` not yet inspected ([reviewer-notes.md §7](reviewer-notes.md#7-priority-ranked-evidence-gaps))
- **Architectural signals:** `backend/tests/test_fail_closed.py` (OBSERVED — negative case indicator), `test_r7_tenant_token.py` (OBSERVED)
- **Phase 4 priority:** HIGH — tenant isolation and fail-closed behavior are key claims

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

These decisions are not blocking Phase 2 (schema and index work). They are blocking Phase 3 deep-dive work on PRJ-01, PRJ-02, and PRJ-03.

---

## 5. Phase Completion Status

| Phase | Description | Status | Commit SHA |
|-------|-------------|--------|-----------|
| 0 | Portfolio Baseline and Conflict Map | COMPLETE (read-only) | N/A |
| 1 | Source Traceability Inventory | COMPLETE | `3270b21a350eafa3a152b750faaafbca47756785` |
| 2 | Portfolio Evidence Foundation | IN PROGRESS | TBD (this phase) |
| 3 | Flagship Deep Engineering Cases | NOT STARTED — AWAITING OWNER | — |
| 4 | High-Risk Engineering Cases | NOT STARTED — AWAITING OWNER | — |
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
