# Evidence Schema

**Phase:** 2 — Portfolio Evidence Foundation\
**Created:** 2026-10-17\
**Portfolio HEAD at phase start:** `3270b21a350eafa3a152b750faaafbca47756785`\
**Status:** Normative schema for all Phase 2+ evidence records.

This document defines the unique ID scheme, required fields, status taxonomy, evidence strength scale, and reproducibility levels used across all evidence files in `docs/evidence/`. Every record added in Phase 2 and later must conform to this schema. Phase 1 records are grandfathered and noted where they diverge.

---

## 1. Unique ID Scheme

### Project IDs

| ID | Canonical Name |
|----|---------------|
| PRJ-01 | AILORA |
| PRJ-02 | Aegis Agent |
| PRJ-03 | IBM Supply Chain Challenge |
| PRJ-04 | AlinaMatrix Enterprise |
| PRJ-05 | Kian Trading Intelligence |
| PRJ-06 | Ahmad & Amin Technology |
| PRJ-07 | NimaOS AI Agent |
| PRJ-08 | Lieferkette Optimierungsplattform |
| PRJ-09 | Personal AI Assistant |
| PRJ-10 | Restaurant AI Platform |
| PRJ-11 | Büro Assistant |
| PRJ-12 | Patch-Auri |
| PRJ-13 | Azimi Innovation Lab Website |

IDs PRJ-01 through PRJ-13 are fixed. They must not be recycled, renumbered, or assigned to different projects.

### Claim IDs

Format: `CLM-<PRJ_NUM>-<NNN>` (three-digit zero-padded sequential integer per project).\
Example: `CLM-03-001` = first claim for IBM Supply Chain Challenge.

Claims are never deleted. If a claim is superseded, add a `superseded_by` reference and record the supersession date. The old ID remains pointing to the replacement.

### Evidence IDs

Format: `EVD-<PRJ_NUM>-<NNN>`\
Example: `EVD-01-001` = first evidence record for AILORA.

Phase 1 used `claims.csv` to record both claims and evidence in one file. Phase 2+ maintains the same claim IDs in `claims.csv` and introduces separate `EVD-` IDs in Phase 3+ case evidence files when deeper source inspection is performed.

### Gap IDs

Format: `GAP-<PRJ_NUM>-<NNN>`\
Example: `GAP-01-001` = first unresolved evidence gap for AILORA.

Gaps from Phase 1 are defined in `reviewer-notes.md` Section 7 using this scheme. They are not recycled.

### Architecture Decision Record IDs

Format: `ADR-<PRJ_NUM>-<NNN>`\
Example: `ADR-03-001` = first ADR extracted for IBM Supply Chain Challenge.

ADRs are created in Phase 3+ deep-dive case files. The ID prefix distinguishes portfolio-level ADR records from any source-project ADRs (which may have their own naming).

---

## 2. Required Evidence Fields

The following fields are required for every claim or evidence record. Use the exact field names in CSV headers and Markdown tables.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `evidence_id` | string | Yes | EVD- or CLM- ID (unique, never recycled) |
| `project_id` | string | Yes | PRJ-NN identifier |
| `claim_id` | string | Yes | CLM-NN-NNN (or NONE for pure evidence records not tied to a specific claim) |
| `source_url` | URL | Yes | Canonical URL of source repository or artifact |
| `source_commit_sha` | string | Yes | Full 40-character SHA of inspected commit, or NOT_AVAILABLE / BLOCKED |
| `source_path` | string | Yes | File path within repository, or N/A |
| `source_line_range_or_permalink` | string | Recommended | Specific line range or permalink to the observed artifact |
| `artifact_kind` | enum | Yes | See §3 |
| `observed_at_utc` | ISO 8601 | Yes | Timestamp of observation (YYYY-MM-DDTHH:MM:SSZ), or UNKNOWN |
| `observation_class` | enum | Yes | See §4 |
| `result_status` | enum | Yes | See §4 |
| `method` | string | Yes | How the observation was made (e.g., "GitHub API unauthenticated", "raw file fetch") |
| `tool_version_if_used` | string | Conditional | Tool name and version if a specific tool was used |
| `source_run_url_if_applicable` | URL | Conditional | Link to CI run, release artifact, or other execution record |
| `raw_artifact_hash_if_available` | string | Conditional | SHA-256 of the artifact bytes, only for actual available bytes |
| `limitations` | string | Yes | What is NOT proven or cannot be determined from this record |
| `confidence` | enum | Yes | HIGH / MEDIUM / LOW / UNKNOWN |
| `independence_level` | enum | Yes | See §5 |
| `sensitivity` | enum | Yes | NONE / LOW / MEDIUM / HIGH (for redaction decisions) |
| `reviewer` | string | Yes | Who or what performed the observation (human, tool, or method) |
| `follow_up` | string | Yes | Next action required, or NONE |

**Never leave required fields blank or omit them.** Use `UNKNOWN` or `NOT_AVAILABLE` when a value genuinely cannot be obtained; do not leave blanks that imply success.

---

## 3. Artifact Kind Values

| Value | Meaning |
|-------|---------|
| `SOURCE_CODE` | Inspected source file in repository |
| `CONFIGURATION` | Configuration file (CI workflow, Dockerfile, etc.) |
| `TEST_FILE` | Test source file |
| `CI_RUN` | Linked CI execution record |
| `RELEASE_ARTIFACT` | Published release tag, package, or asset |
| `DOCUMENTATION` | README, qualification doc, ADR, verification doc |
| `DEPLOYMENT_URL` | Live service endpoint |
| `WEBSITE` | Public website (no source repo) |
| `METADATA` | Repository metadata (language, license, star count) |
| `DERIVED` | A record derived from another artifact (must link to original) |

---

## 4. Observation Class and Result Status Taxonomy

### Observation Class (how the claim was made)

| Value | Meaning |
|-------|---------|
| `OBSERVED` | Analyst directly inspected the source artifact at the stated revision |
| `REPORTED` | Assertion made by the source author in documentation or README |
| `INFERRED` | Technical interpretation from observed evidence; not direct proof |
| `NOT_TESTED` | A test or execution check that was not performed |
| `BLOCKED` | Access, environment, rate limits, rights, or permissions prevent verification |
| `FAILED` | A scoped check was executed and failed |

These classes are mutually exclusive per observation. Do not merge them.

### Result Status (outcome of the observation)

| Value | Meaning |
|-------|---------|
| `PASS` | Claim is directly supported by the observed artifact |
| `CONDITIONAL` | Claim is partially supported; qualifications apply |
| `FAIL` | Claim contradicted by observation or check |
| `BLOCKED` | Verification prevented; outcome unknown |
| `NOT_TESTED` | No execution evidence; source-level only |
| `NOT_APPLICABLE` | Claim/check does not apply to this artifact |

**Critical rule:** A source file being present (`OBSERVED`) does not imply that tests pass (`NOT_TESTED`). These must be recorded as separate records with separate statuses. Never collapse `OBSERVED` into `PASS` for an execution claim.

---

## 5. Evidence Strength Scale

| Level | Description | Example |
|-------|-------------|---------|
| E0 | Unsubstantiated claim; no source artifact | Marketing text only |
| E1 | Source document or README statement | README says "69 tests pass" |
| E2 | Directly inspected code, configuration, or release artifact | Test files observed in tree |
| E3 | Linked execution output with SHA, environment, command, and result | CI run URL with passing status |
| E4 | Independent clean-environment reproduction | Third party ran tests in isolated env |
| E5 | Approved longitudinal operational measurement with provenance | Uptime telemetry over 30 days |

Evidence strength is not a quality score and does not imply `PASS`. A E2 observation of test files does not prove the tests pass (still `NOT_TESTED` for execution). Upgrades from E1/E2 to E3/E4/E5 require qualifying external evidence; this portfolio program does not itself authorize the execution needed for E3–E5 in source projects.

---

## 6. Reproducibility Levels

| Level | Description |
|-------|-------------|
| R0 | Statement only; no source artifact |
| R1 | Source and configuration inspected at immutable revision |
| R2 | Test/CI run artifact linked and associated with that revision |
| R3 | Build/test reproduction independently executed in isolated environment |
| R4 | Critical negative/failure cases independently reproduced |
| R5 | Operational behavior measured over a disclosed representative window |

The current Phase 1 and Phase 2 evidence is R1 for source-accessible projects and R0 for PRJ-07 and PRJ-13. R2 applies only where a CI run URL is linked. R3–R5 are not authorized under this portfolio program.

---

## 7. Independence Levels

| Value | Meaning |
|-------|---------|
| `SELF_REVIEW` | Author of the work reviewed their own output |
| `INTERNAL_PORTFOLIO_REVIEW` | Internal source/documentation inspection; not independently reproduced |
| `SEPARATE_ENGINEER_REVIEW` | A human engineer distinct from the author reviewed the work |
| `INDEPENDENT_THIRD_PARTY_REVIEW` | Reviewer with no conflicts and separate environment |

Internal portfolio observations are based on available source and documentation; no independent third-party validation or clean-room reproduction is claimed.

---

## 8. Schema Versioning

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-10-17 | Initial schema, created in Phase 2 |

Schema changes require a new version entry and must not invalidate existing record IDs.
