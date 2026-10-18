# Evidence Directory

**Phases 1–2 — Source Traceability Inventory and Evidence Foundation**
**Phase 1 created:** 2026-10-17 · **Phase 2 created:** 2026-10-17
**Phase 1 portfolio HEAD:** `0e8a84cd20322a48da0b722981d1e535394ee6c3`
**Phase 2 portfolio HEAD:** `3270b21a350eafa3a152b750faaafbca47756785`
**Status:** Phase 2 deliverables committed. Awaiting owner authorization for Phase 3.

**Start here → [`central-index.md`](central-index.md)** — navigable index linking all evidence files, project status, and claim-to-source map.

---

## Contents

| File | Description | Phase |
|------|-------------|-------|
| [`central-index.md`](central-index.md) | **Entry point.** Navigable index: all 13 projects, evidence maturity, per-project claim links, open decisions, phase completion status | 2 |
| [`schema.md`](schema.md) | Normative evidence schema: unique ID scheme (PRJ/CLM/EVD/GAP/ADR), required fields, artifact kinds, observation class and status taxonomy, evidence strength (E0–E5), reproducibility levels (R0–R5), independence levels | 2 |
| [`provenance-rules.md`](provenance-rules.md) | Normative provenance rules: source SHA pinning (P-01–P-16), observation timestamps, provenance layers, independence disclosure, missing evidence handling, IP sensitivity | 2 |
| [`source-inventory.csv`](source-inventory.csv) | Per-project source availability, language, license, CI presence, SHA, last-pushed date, and release tags for all 13 cases | 1 |
| [`claims.csv`](claims.csv) | Source-to-claim matrix: 33 claims (CLM-01-001 through CLM-13-002), observation class, result status, source SHA, follow-up actions | 1 |
| [`verification-status.csv`](verification-status.csv) | Summary verification status per project: CI health, test evidence, docs observed, deployment URL, portfolio claim consistency, and material gaps | 1 |
| [`reviewer-notes.md`](reviewer-notes.md) | Detailed Phase 1 findings: SHA register, CI matrix, license register, 10 priority-ranked evidence gaps (3 critical), architectural depth signals, F01–F30 filter record, 5 open decisions | 1 |

---

## Evidence Model

Observation classes: **OBSERVED** (directly inspected) · **REPORTED** (source author assertion) · **INFERRED** (reasoned interpretation) · **NOT_TESTED** · **BLOCKED** · **FAILED**
Outcome: **PASS** · **CONDITIONAL** · **FAIL** · **BLOCKED** · **NOT_TESTED** · **NOT_APPLICABLE**
Evidence strength: E0 (unsubstantiated) → E1 (README/doc) → E2 (inspected code) → E3 (linked execution output) → E4 (independent reproduction) → E5 (longitudinal measurement)
Reproducibility: R0 (statement only) → R1 (source inspected) → R2 (CI linked) → R3 (independent reproduction) → R4 (negative cases reproduced) → R5 (longitudinal measurement)

Full definitions: see [schema.md](schema.md). Source SHA and timestamp rules: see [provenance-rules.md](provenance-rules.md).

Phase 1 and Phase 2 observations are internal read-only inspections of public GitHub metadata and repository trees. No source code was executed and no source repositories were modified. No independent third-party validation is claimed.

---

## Project ID Reference

PRJ-01 AILORA · PRJ-02 Aegis Agent · PRJ-03 IBM Supply Chain Challenge · PRJ-04 AlinaMatrix Enterprise
PRJ-05 Kian Trading Intelligence · PRJ-06 Ahmad & Amin Technology · PRJ-07 NimaOS AI Agent
PRJ-08 Lieferkette Optimierungsplattform · PRJ-09 Personal AI Assistant · PRJ-10 Restaurant AI Platform
PRJ-11 Büro Assistant · PRJ-12 Patch-Auri · PRJ-13 Azimi Innovation Lab Website
