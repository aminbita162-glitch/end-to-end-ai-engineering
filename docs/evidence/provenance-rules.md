# Provenance Rules

**Phase:** 2 — Portfolio Evidence Foundation\
**Created:** 2026-10-17\
**Portfolio HEAD at phase start:** `3270b21a350eafa3a152b750faaafbca47756785`\
**Status:** Normative provenance rules for all portfolio evidence records.

These rules govern how source SHAs, observation timestamps, independence levels, and derived artifacts are recorded. They apply to all evidence added in Phase 2 and later, and serve as the retroactive standard against which Phase 1 records should be read.

---

## 1. Source SHA Pinning

### Rule P-01: Full SHA Required

Every source observation must record a full 40-character commit SHA, not a branch name, tag, or abbreviated ref. Branch names are mutable; a SHA is an immutable content address.

- ✅ `source_commit_sha: 46ca4a939bee0396e90abef91905b6d1b95d470c`
- ❌ `source_commit_sha: main`
- ❌ `source_commit_sha: candidate-2026-08-29`
- ❌ `source_commit_sha: 46ca4a9` (abbreviated)

### Rule P-02: Tag SHA Distinction

A release tag name (e.g., `v1.0.0-submission-final`) is not the same as the commit SHA it points to. Record both the tag name and the commit SHA when available.

### Rule P-03: BLOCKED When SHA Unavailable

If a full SHA cannot be obtained (e.g., rate limit, private repo, website-only artifact), record `source_commit_sha: BLOCKED` or `NOT_AVAILABLE` with a `limitations` entry explaining why. Do not substitute a branch name or invent a SHA.

### Rule P-04: Immutable Permalinks

Where a service provides immutable permalinks (e.g., GitHub `blob/<SHA>/path/to/file`), use these in `source_line_range_or_permalink` instead of branch-relative URLs. Branch-relative URLs become stale as code changes.

### Rule P-05: Git Blob SHA ≠ File Hash

A Git blob SHA is an SHA-1 content hash over a specific internal format. It is not equivalent to a SHA-256 digest of the raw file bytes. Never present a Git blob SHA as a SHA-256 file digest. Hash only actual bytes available for inspection.

---

## 2. Observation Timestamps

### Rule P-06: UTC Required

All `observed_at_utc` timestamps must be in ISO 8601 format with explicit UTC suffix: `YYYY-MM-DDTHH:MM:SSZ`. Date-only values (e.g., `2026-10-17`) are permitted only when the time is genuinely unknown; record `T00:00:00Z` with a note in `limitations` explaining the precision.

### Rule P-07: Observation ≠ Source Event

The `observed_at_utc` timestamp records **when this portfolio program inspected** the source artifact — not when the artifact was created, committed, or published. Never conflate these.

| Timestamp field | Records |
|-----------------|---------|
| `observed_at_utc` | When the source artifact was inspected |
| `last_pushed_at` (source-inventory.csv) | When the source repo last received a push (from GitHub API) |
| `latest_release_date` (source-inventory.csv) | When the release artifact was published |
| Git commit `AuthorDate` | When the source author created the commit |

### Rule P-08: Volatile Artifacts

For changing artifacts (live websites, health endpoints), capture retrieval time, URL, and include an explicit volatility warning in `limitations`. Never imply that a live URL response at time T proves the service is continuously available.

---

## 3. Evidence Provenance Layers

All evidence in this portfolio must be categorized into one of four provenance layers. These layers are mutually exclusive and must not be conflated.

| Layer | Label | Description |
|-------|-------|-------------|
| 1 | `ORIGINAL_SOURCE` | Canonical repository at an immutable SHA permalink. This is the authoritative artifact. |
| 2 | `ORIGINAL_EXECUTION` | An actual CI/test/benchmark run with a linked timestamp and run ID. |
| 3 | `PORTFOLIO_OBSERVATION` | Read-only inspection of Layer 1 or Layer 2 artifacts. |
| 4 | `INDEPENDENT_REPRODUCTION` | A third party re-ran the test/build in a separate, clean environment. |
| 5 | `OPERATIONAL_EVIDENCE` | Approved measured telemetry over a disclosed, representative time window. |

**No evidence laundering:** A copied log, generated chart, authored evaluation, or repeated assertion does not become independent proof by being stored in a different folder. Every derived record must link back to its original Layer 1 or Layer 2 artifact and declare which layer it belongs to.

**Current portfolio state:** All Phase 1 and Phase 2 records are Layer 3 (PORTFOLIO_OBSERVATION). Layer 4 and Layer 5 evidence are absent and would require separate authorization.

---

## 4. Observation Independence Disclosure

### Rule P-09: Label Every Record

Every evidence record must state its `independence_level` using one of the values defined in [schema.md §7](schema.md#7-independence-levels):

- `SELF_REVIEW`
- `INTERNAL_PORTFOLIO_REVIEW`
- `SEPARATE_ENGINEER_REVIEW`
- `INDEPENDENT_THIRD_PARTY_REVIEW`

### Rule P-10: No Independence Inflation

Internal portfolio observations are based on available source and documentation; no independent third-party validation or clean-room reproduction is claimed.

### Rule P-11: Reviewer Conflicts

If the observer has access to context that might bias interpretation (e.g., prior instructions about a project, access to unpublished information), this must be noted in the `limitations` field.

---

## 5. Handling Missing or Inaccessible Evidence

### Rule P-12: Explicit UNKNOWN

Use the literal string `UNKNOWN` (not blank) for any required field whose value is genuinely not obtainable. A blank field in a CSV may be misread as implying verification. `UNKNOWN` is an explicit statement of what is not known.

### Rule P-13: BLOCKED vs NOT_TESTED

| Status | Use when |
|--------|----------|
| `BLOCKED` | The check could not be performed due to rate limits, permissions, private repo, or other external constraint |
| `NOT_TESTED` | The check was within scope but was not performed in this phase |

Do not use `BLOCKED` to mean "we chose not to do this". Reserve it for genuine external barriers.

### Rule P-14: No Invented Hashes

Never fabricate a SHA or hash. If the artifact hash is not available, record `raw_artifact_hash_if_available: NOT_AVAILABLE`. A fictional hash is worse than an absent one.

---

## 6. Intellectual Property and Sensitivity

### Rule P-15: Link, Do Not Copy

When source code must be referenced, prefer a SHA-pinned permalink over reproducing the code in the portfolio. Reproduction requires a compatible license. Projects with `license_spdx: NONE` (PRJ-04, PRJ-08, PRJ-09, PRJ-12) must not have source code copied into the portfolio without explicit owner authorization.

### Rule P-16: Sensitivity Classification

Set `sensitivity` in every record:

| Value | Meaning |
|-------|---------|
| `NONE` | Public information with no sensitivity concerns |
| `LOW` | Potentially sensitive; review before broad distribution |
| `MEDIUM` | Contains personal data, private API patterns, or internal configuration |
| `HIGH` | Contains credentials, PII, proprietary algorithms, or confidential customer data |

Records with `sensitivity: HIGH` must not be committed to the public portfolio. They must be redacted and linked to a metadata record only.

---

## 7. Portfolio Observation Record Format

When a Phase 3+ case file records a portfolio observation, include this minimal block:

```
**Observation record**
- Provenance layer: PORTFOLIO_OBSERVATION
- Source: <source_url>/blob/<full_sha>/<path>
- Observed at: <YYYY-MM-DDTHH:MM:SSZ>
- Method: <how>
- Independence: Source/documentation observations only; no independent reproduction claimed
- Limitations: <what this does NOT prove>
```

This block must appear for every substantive factual claim in a case engineering document. It ensures a skeptical reader can follow the claim-to-source chain without navigating multiple files.

---

## 8. Provenance Rules Version History

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-10-17 | Initial rules, created in Phase 2 |
