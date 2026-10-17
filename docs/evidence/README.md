# Evidence Directory

**Phase 1 — Source Traceability Inventory**  
**Created:** 2025-10-17  
**Portfolio HEAD:** `0e8a84cd20322a48da0b722981d1e535394ee6c3`  
**Status:** Phase 1 deliverables committed. Awaiting owner authorization for Phase 2.

---

## Contents

| File | Description | Phase |
|------|-------------|-------|
| [`source-inventory.csv`](source-inventory.csv) | Per-project source availability, language, license, CI presence, SHA, last-pushed date, and release tags for all 13 cases | 1 |
| [`claims.csv`](claims.csv) | Source-to-claim matrix: each material portfolio claim mapped to source URL, observation class (OBSERVED/REPORTED/NOT_TESTED/BLOCKED), result status, and follow-up action | 1 |
| [`verification-status.csv`](verification-status.csv) | Summary verification status per project: CI health, test evidence, docs observed, deployment URL, portfolio claim consistency, and material gaps | 1 |
| [`reviewer-notes.md`](reviewer-notes.md) | Detailed Phase 1 findings: SHA register, CI matrix, license register, evidence gaps prioritized by severity, architectural depth signals, F01–F30 filter record, open decisions | 1 |

---

## Evidence Model

Observation classes: **OBSERVED** (directly inspected) · **REPORTED** (source author assertion) · **INFERRED** (reasoned interpretation) · **NOT_TESTED** · **BLOCKED** · **FAILED**  
Outcome: **PASS** · **CONDITIONAL** · **FAIL** · **BLOCKED** · **NOT_TESTED** · **NOT_APPLICABLE**  
Evidence strength: E0 (unsubstantiated) → E1 (README/doc) → E2 (inspected code) → E3 (linked execution output) → E4 (independent reproduction) → E5 (longitudinal measurement)

All Phase 1 source observations are analyst read-only inspections of public GitHub metadata and tree contents. No source code was executed. No source repositories were modified.

---

## Project ID Reference

PRJ-01 AILORA · PRJ-02 Aegis Agent · PRJ-03 IBM Supply Chain Challenge · PRJ-04 AlinaMatrix Enterprise  
PRJ-05 Kian Trading Intelligence · PRJ-06 Ahmad & Amin Technology · PRJ-07 NimaOS AI Agent  
PRJ-08 Lieferkette Optimierungsplattform · PRJ-09 Personal AI Assistant · PRJ-10 Restaurant AI Platform  
PRJ-11 Büro Assistant · PRJ-12 Patch-Auri · PRJ-13 Azimi Innovation Lab Website
