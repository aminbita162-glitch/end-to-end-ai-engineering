# PRJ-04 — AlinaMatrix Enterprise: Engineering Evidence

**Project ID:** PRJ-04
**Canonical name:** AlinaMatrix Enterprise
**Source:** https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise
**Source HEAD (Phase 4 inspection):** `e7cc7df20164f2806e93bed6f5a665865e72133b` (2026-10-17T03:38:24Z)
**Default branch:** `main`
**License:** NONE — no license file detected. GitHub API reports `license: null`. Copying source code into the portfolio requires explicit owner authorization. No source code is copied into this case file — only attributed descriptions, file paths, and SHA-pinned permalinks.
**Software version:** `1.0.0` (OBSERVED in `package.json`)
**Phase 4 inspection date (UTC):** 2026-10-17
**Independence:** Source/documentation observations only; no independent third-party verification claimed.
**Independence level:** Source/documentation observations only; no independent reproduction claimed.
**Evidence maturity:** R1 — source and configuration inspected at immutable revision. R2 partially achieved — CI run SUCCESS at HEAD `e7cc7df20164` observed via GitHub Actions API (run #43, 2026-10-17T03:38:27Z). Not independently reproduced.
**Reproducibility ladder:** R1 (source inspected at pinned SHA). R2 partial — CI run artifact linked and observed at HEAD. R3 not reached — no independent clean-environment reproduction. R5 not reached — no longitudinal operational measurement.

---

## 1. Context and Scope

AlinaMatrix Enterprise is a multi-tenant AI document-engineering and evidence-control platform. It turns complex source evidence into auditable professional artifacts through a controlled pipeline of four agents, human review, four-eyes approval, Merkle provenance, and a release gate. The beachhead is M03 (Architecture Decision Records) only — modules M01–M02 and M04–M30 are explicitly deferred by ADR-0001.

**Status (OBSERVED in README.md at HEAD):**

> **Enterprise Candidate — Active Development**
> No production claim is made. Beachhead M03 is not yet proven.
> Candidate stays until real deployment and real use.
> A small test count is not proof of real-world capacity.

**Non-goals (OBSERVED across README, ADRs, and source comments):**

- Not a production system. Not deployed to production. No production claim of any kind.
- Not a chatbot, search engine, or general-purpose LLM wrapper.
- No live LLM call has been made. All agent calls use `DeterministicFakeProvider` only (ADR-0003, R07 directive).
- Beachhead M03 only. Other document types (M01–M02, M04–M30) are deferred.
- No regional disaster recovery (no DR site, no cross-region replication).
- "AI writes your documents" must not appear (ADR-0003 explicit prohibition).

---

## 2. System Architecture (RECONSTRUCTED FROM SOURCE)

Architecture reconstructed by reading `README.md`, `package.json`, `apps/api/src/`, `packages/domain/src/`, `packages/db/src/`, `packages/db/migrations/`, `packages/contracts/src/`, `packages/renderer/src/`, `.github/workflows/ci.yml`, `docker-compose.yml`, `render.yaml`, and `docs/adr/` at HEAD SHA `e7cc7df20164`. Not a verified runtime topology.

### 2a. Monorepo Decomposition

```
AlinaMatrix Enterprise (pnpm workspaces, TypeScript strict, Node 22)
  │
  ├── apps/api/          — Node HTTP API server (no Express; raw Node HTTP)
  │     src/             — auth, extractor, ingest, review, router, storage, logger, openapi
  │     6 test files     — auth.test.ts, ingest.test.ts, logger.test.ts, review.test.ts,
  │                        router.test.ts, storage.test.ts
  │
  ├── apps/web/          — Next.js App Router (web frontend)
  │     app/             — claims, projects, sources, review pages
  │
  ├── packages/contracts/ — Zod schemas (shared types; no runtime dependency)
  │     src/             — agents, auth, budget, claims, consensus, export, live-model,
  │                        m03, pinned-model, provenance, release, render, review,
  │                        revocation, sources, template, trace
  │
  ├── packages/domain/   — Pure TypeScript domain logic (no runtime deps)
  │     src/             — agents, architect, budget, claims, consensus, export,
  │                        generate, guard, live-model, negative-evidence,
  │                        numeric-reconciliation, pinned-model, provenance, release,
  │                        review, revocation, template, trace, workflow
  │     24 test files    — unit + 1 golden fixtures test
  │
  ├── packages/db/       — PostgreSQL data access layer
  │     src/             — agents, claims, client, consensus, export-audit, m03,
  │                        migrate, provenance, release, render, review, revocation,
  │                        sources, tenant-context, trace
  │     migrations/      — 14 SQL migrations (001–014, forward-only)
  │     seed/            — fixture.sql + injection-fixture.ts
  │     20 test files    — including 4 integration tests (skipped without live DB)
  │
  └── packages/renderer/ — HTML/PDF artifact rendering
        src/             — beachhead, html, manifest, pdf, replay, watermark
        6 test files
```

### 2b. Four-Agent Pipeline

The system has exactly four agents, each with a frozen capability set (OBSERVED in `packages/domain/src/agents.ts`):

| Agent | Capabilities | Role |
|-------|-------------|------|
| **AlinaArchitect** | `READ_SOURCE`, `READ_EVIDENCE`, `WRITE_DRAFT` | Reads source evidence, writes the M03 draft |
| **AlinaGuard** | `READ_EVIDENCE`, `RUN_VALIDATION` | Validates the draft against schema, quote-lock, and guard rules |
| **AlinaDocEngine** | `COMPILE_ARTIFACT` | Compiles the approved draft into the final artifact (HTML + PDF) |
| **AlinaOptimizer** | `READ_USAGE`, `WRITE_CACHE_METADATA` | Reads usage metrics and writes cache metadata; no content authority |

**Release/export operations are blocked for ALL agents unconditionally** (OBSERVED in `agents.ts`):

```typescript
const RELEASE_OPERATIONS: ReadonlySet<string> = new Set(["RELEASE", "EXPORT"]);

export function assertAgentCapability(agent: AgentName, capability: AgentCapability | string): void {
  if (RELEASE_OPERATIONS.has(capability.toUpperCase())) {
    throw new CapabilityDeniedError(agent, capability);  // unconditionally
  }
  // ... then per-agent capability check
}
```

**Engineering significance:** No agent can release or export. Release is a separate human-gated process (§5). This is a structural separation between agent capability and release authority — the agents propose, a human disposes.

### 2c. Workflow State Machine

The pipeline has 17 states with enforced legal transitions (OBSERVED in `packages/domain/src/workflow.ts`):

```
INGESTED → CLASSIFIED → EXTRACTED → EVIDENCE_READY → ARCHITECTED → GENERATED → VALIDATING → NEEDS_REVIEW → APPROVED → BUILDING → BUILT → RELEASED

Error paths: FAILED_RETRYABLE, FAILED_TERMINAL, CANCELLED, EXPIRED, QUARANTINED
```

`assertLegalTransition(from, to)` throws `IllegalTransitionError` on any non-allowed path. Terminal states (`FAILED_TERMINAL`, `CANCELLED`, `EXPIRED`, `QUARANTINED`) have no outbound transitions — re-entry requires explicit human intervention outside the map.

---

## 3. DeterministicFakeProvider and Provider Separation (KEY CLAIM)

### 3a. DeterministicFakeProvider — Deterministic, No Live LLM

**OBSERVED in `packages/domain/src/agents.ts`:**

`DeterministicFakeProvider` is a pure function. Given the same `(promptSha256, schemaSha256, inputHash)` the output is always identical — no randomness, no temperature variation, no network call. Token counts are derived deterministically from the input hashes so `usage_event` rows contain reproducible values.

The ADR-0003 directive is explicit (OBSERVED):

> No live LLM may be called by the system. Until a later directive revision explicitly permits it, all agent calls use `DeterministicFakeProvider`.

### 3b. PinnedModelProvider — Deterministic with Pinned Config

**OBSERVED in `packages/domain/src/pinned-model.ts`:**

`PinnedModelProvider` sits beside `DeterministicFakeProvider`. It pins four fields (`modelId`, `temperature`, `schemaVersion`, `budgetCeilingMinorUnits`) and produces a deterministic output whose hash differs from `DeterministicFakeProvider` for the same input (the seed includes the provider name + model id, so the hash differs). Budget is checked before the model is called — over budget returns `BudgetBreachResult`, model is NOT called.

### 3c. LiveModelProvider — Real LLM, Env-Gated, Budget-Enforced

**OBSERVED in `packages/domain/src/live-model.ts`:**

`LiveModelProvider` makes a real LLM API call with the same four pinned fields. It enforces:

1. **Budget check before model call** — if `spentMinorUnits > budgetCeilingMinorUnits`, the model is NOT called and a `BudgetBreachResult` is returned.
2. **API key check** — if the key (env var named in `config.apiKeyEnvVar`) is missing or empty, the call is blocked with a `LiveCallBlockedResult` and the exact blocker reason. No response is invented.
3. **Unknown key dropping** — the model's raw response is cleaned: only known keys (`_provider`, `_modelId`, `_temperature`, `_schemaVersion`, `result`, `reasoning`) are kept. Unknown keys are dropped before validation.
4. **Schema validation** — the cleaned output is validated against `LiveModelOutputSchema`. If validation fails, the call is blocked (schema mismatch).
5. **Semantic variance recording** — `recordLiveSemanticVariance` compares `DeterministicFakeProvider`'s output hash against the live provider's output hash for the same input and returns a `SemanticVarianceRecord` (sameHash: false — different provider).

**Provider separation is ENFORCED in code, not just documented.** The three providers are distinct classes with distinct invariants. `DeterministicFakeProvider` has no network path. `LiveModelProvider` requires a valid API key and budget. `PinnedModelProvider` is deterministic with a different seed.

### 3d. ADR-0003 — Determinism and Honesty Policy

**OBSERVED in `docs/adr/ADR-0003-determinism-and-honesty.md`:**

| Aspect | Decision | Status |
|--------|----------|--------|
| Determinism | "1000% deterministic" is a quality slogan, not a provable property. Versioned inputs, pinned generation, constrained generation, reproducible build, immutable audit, human approval. | Accepted |
| Honesty | Status is exactly "Enterprise Candidate — Active Development". No other status permitted. Test not executed = not passed. No invented citations/DOI/scores. | Accepted |
| DeterministicFakeProvider | Same prompt+schema+input hash → same JSON output. Every fake call writes a usage row. Idempotency tested in Phase 5. | Accepted |
| Temperature=0 | A sampling setting, not factual correctness. Must never appear as a correctness claim. | Accepted |

**Residual risk (from ADR):** "Replay test (Phase 8) is the only machine-verifiable proof of the determinism invariant. Until Phase 8 is complete, the invariant is stated policy, not proven fact."

---

## 4. Multi-Tenant Isolation and RLS

### 4a. PostgreSQL RLS with Fail-Closed

**OBSERVED in `packages/db/migrations/001_identity_tenancy_rls.sql`:**

All tenant-scoped tables have RLS enabled AND forced:
- `ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;`
- `ALTER TABLE tenants FORCE ROW LEVEL SECURITY;` (applied to 7 tables: tenants, roles, users, memberships, projects, audit_events, sessions)

**Fail-closed design:** `current_tenant_id()` returns `NULL` when `app.current_tenant_id` is not set. RLS `USING` clauses compare against `NULL` → no rows returned. This is the "fail closed" behaviour required by the directive.

### 4b. Server-Derived Tenant Context

**OBSERVED in `packages/db/src/tenant-context.ts`:**

`setTenantContext(client, tenantId)` validates the tenant_id as a UUID before issuing `SELECT set_config('app.current_tenant_id', $1, true)`. Non-UUID strings, empty strings, SQL injection attempts (`'; DROP TABLE tenants; --`), and path-like strings (`../other-tenant`) are all rejected before reaching the database.

The tenant_id is server-derived from the session/membership — never from client input (ADR-0002 R08 directive).

### 4c. RLS Integration Tests

**OBSERVED in `packages/db/src/rls.integration.test.ts`:**

Live RLS integration tests run against a PostgreSQL 16 instance with `DATABASE_URL` and `SUPERUSER_URL`. Tests include:
- Tenant A context sees only Tenant A in `tenants` table
- Tenant A cannot read Tenant B's projects
- No tenant context (empty `app.current_tenant_id`) → no rows (fail-closed)
- Fixture IDs matching `packages/db/seed/fixture.sql`

**CI includes live PostgreSQL:** `.github/workflows/ci.yml` provisions a `postgres:16` service container and runs migrations + seed before tests. The CI run at HEAD is SUCCESS (run #43).

**Test count note:** README claims "101 integration tests are skipped without a live PostgreSQL instance." In CI (where PostgreSQL is provisioned), these tests run. The 4 integration test files are: `agents.integration.test.ts`, `claims.integration.test.ts`, `rls.integration.test.ts`, `sources.integration.test.ts`. The 101 count is REPORTED in the README — NOT independently re-counted.

---

## 5. Human Review, Four-Eyes, and Release Gate

### 5a. Four-Eyes Approval

**OBSERVED in `packages/domain/src/review.ts`:**

`assertFourEyes(authorId, approverId)` throws `FourEyesError` when `authorId === approverId` — the author of the draft cannot be the sole release approver. A second independent approver is required.

`canCompleteReview(claims)` returns `false` when any claim is rejected — a rejected claim blocks the review from completing.

**OBSERVED in `apps/api/src/review.ts`:**

The HTTP handler `POST /review-tasks/:taskId/approvals` calls `assertFourEyes` before recording an "approved" task approval. The four-eyes rule is enforced at the API boundary, not just in domain logic.

### 5b. Release Gate

**OBSERVED in `packages/domain/src/release.ts`:**

`checkReleaseGate(input)` returns `{ passed, reasons }`. The gate blocks when ANY of these conditions is true:

1. `validationsPassed === false`
2. `criticalFindings > 0`
3. `fourEyesSatisfied === false`
4. `manifestPresent === false`
5. `checksumPresent === false`
6. `sourceVersionIds` is empty
7. `versionPinsPresent === false`
8. `sourceJobState` is `FAILED_RETRYABLE`, `FAILED_TERMINAL`, or `CANCELLED`

Release labels follow a legal transition map: `DRAFT → INTERNAL_REVIEW → APPROVED → PUBLISHED → ARCHIVED → REVOKED`. `assertLegalLabelTransition` throws `IllegalLabelError` on any non-allowed path.

### 5c. Expert Acceptance Protocol

**OBSERVED in `docs/eval/expert-acceptance-protocol.md`:**

A 9-section evaluation protocol with 0–5 scoring. All scores are blank (`__`) — no score is pre-filled or assumed. The protocol states: "Any score of 0 on a critical item forces an overall FAIL." The reviewer name, role, date, and verdict fields are all blank. **No expert review has been performed.**

---

## 6. Provenance, Audit, and Revocation

### 6a. Merkle Provenance Ledger

**OBSERVED in `packages/domain/src/provenance.ts`:**

- `computeLeafHash(leaf)` — deterministic SHA-256 over canonical (sorted-keys) JSON of `(sourceVersionId, contentHash, manifestSha256, htmlSha256)`.
- `computeMerkleRoot(leafHashes)` — folds sorted leaf hashes into a Merkle root (SHA-256, sorted leaves, left-right concatenation). Empty set → SHA-256 of empty string. Odd layers → duplicate last node.
- `buildProvenanceLedgerEntry` — assembles a ledger entry with leaf hash, Merkle root, and 1-based sequence.
- `verifyProvenanceLedger(leaves, expectedRoot)` — re-folds and confirms the root matches (tamper detection).

**Determinism guarantee:** No `Date.now()` or `Math.random()`. `JSON.stringify` with sorted keys produces canonical form. Insertion order does not change the root (leaves sorted before folding).

### 6b. Append-Only Audit

**OBSERVED in `packages/db/migrations/001_identity_tenancy_rls.sql`:**

`audit_events` table has RLS enabled and forced. The migration comment states: "Append-only. Update and delete are denied by RLS policy."

**Migration 009** (`009_export_audit.sql`) adds export audit events. **Migration 011** (`011_revocation_events.sql`) adds revocation events with append-only enforcement.

### 6c. Revocation — Event, Not Mutation

**OBSERVED in `packages/domain/src/revocation.ts`:**

`revokeRelease(input)` builds a revocation event record. It is a pure function — it does not mutate published bytes. The `priorLabel` must be `PUBLISHED` or `ARCHIVED`. The `reason` must be non-empty.

The published HTML in `release_labels` is protected by a DB trigger (migration 008) that rejects any `UPDATE` of the `published_html` column. The revocation event is append-only (migration 011 rejects `UPDATE` and `DELETE`).

---

## 7. Budget Enforcement

**OBSERVED in `packages/domain/src/budget.ts`:**

`checkBudget({ spentMinorUnits, budgetMinorUnits })` uses `bigint` comparison (exact; `number` loses precision past 2^53). Over budget → `passed: false`; the caller must not call the provider.

`BudgetBreaker` wraps `DeterministicFakeProvider` and refuses to call it when over budget, returning a `BudgetBreachResult` with the exact overage amount.

Both `PinnedModelProvider` and `LiveModelProvider` check the budget before calling the model. This is a three-layer budget guard: domain `checkBudget`, provider-level budget check, and `BudgetBreaker` wrapper.

---

## 8. Guard Validation

**OBSERVED in `packages/domain/src/guard.ts`:**

`runGuard(input)` checks `AlinaGuard` capability (`RUN_VALIDATION`) and validates the M03 draft against:

1. **Schema validation** — `M03ContentModelSchema.safeParse(draftJson)`. Failures → `schemaErrors++` per issue.
2. **Quote-lock re-verification** — for `witnessed` claims, `assertQuoteLock` verifies the quote hash matches the fragment text.
3. **Unsupported blocks** — claims with `supportStatus` not "supported" or "witnessed".
4. **Contradictions** — claims flagged as contradictions.
5. **Negative evidence** — claims missing negative evidence notes when required.
6. **Terminology drift** — terms in the draft's terminology section not in the project's registered terms.

`guardResultToTransition`:
- All rules pass → `APPROVED`
- Any soft block → `NEEDS_REVIEW`
- Schema or quote-lock hard fail → `NEEDS_REVIEW` (also records an error finding)

---

## 9. Negative Evidence Recording

**OBSERVED in `packages/domain/src/negative-evidence.ts`:**

`recordNegativeEvidence(input)` creates a structured record of evidence that was expected but not found in the source. The `note` must be non-empty — silence is a violation. The `expectedLabel`, `sourceVersionId`, and `projectId` must all be non-empty.

`NegativeEvidenceKind` types: `fragment_absent`, `claim_absent`, `number_absent`, `term_absent`, `section_absent`.

This complements the guard's `assertNegativeEvidenceNote` (which requires a note when `support_status` is "supported" or `claim_type` is "inferred"). The guard says the note must exist; the recorder produces it.

---

## 10. Export with Watermark and Permission

**OBSERVED in `packages/domain/src/export.ts`:**

`assertExportPermission(input)` throws `ExportPermissionError` when `allowed === false`.

`buildExportBundle(input)` verifies the watermark is intact (present as four `<meta>` tags in the HTML `<head>`) before returning. Throws `WatermarkIntactError` when the watermark is missing. The check requires each field's value to appear in its own named meta tag — not merely as a bare substring.

Watermark fields: `x-alinamatrix-tenant`, `x-alinamatrix-artifact-version`, `x-alinamatrix-build-id`, `x-alinamatrix-build-time`.

---

## 11. Architecture Decision Records

### ADR-04-001 — Beachhead: Module M03 Only

| Field | Value |
|-------|-------|
| **Source** | [`docs/adr/ADR-0001-beachhead-m03.md`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/docs/adr/ADR-0001-beachhead-m03.md) |
| **Status** | Accepted |
| **Date** | 2026-10-01 |
| **Decision** | Beachhead is M03 (Architecture Decision Record) only. M01–M02 and M04–M30 are deferred. |
| **Rationale** | An ADR is the simplest, most self-contained document type that exercises the full pipeline. Shipping all modules in parallel would produce an untestable, unauditable system. |
| **Alternatives rejected** | Shipping all 30 modules in parallel. |
| **Residual risk** | "M03 beachhead is not yet proven. No production claim is made." |

### ADR-04-002 — Technology Stack

| Field | Value |
|-------|-------|
| **Source** | [`docs/adr/ADR-0002-stack.md`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/docs/adr/ADR-0002-stack.md) |
| **Status** | Accepted |
| **Date** | 2026-10-01 |
| **Decision** | TypeScript strict + Node 22; pnpm workspaces; Next.js App Router; Node HTTP API (no Express); PostgreSQL 16 with RLS; Redis 7; zod at all boundaries; argon2id; UUID server-side; UTC storage; SQL forward-only migrations; Vitest; GitHub Actions; ESLint + Prettier. |
| **Alternatives rejected** | Prisma (hides SQL; RLS harder to reason about); Drizzle (immature at decision time); Bun (not LTS-stable); fashionable framework replacements. |
| **Residual risk** | PostgreSQL 16 RLS not exercised until Phase 2. Redis 7 not installed until Phase 5. |

### ADR-04-003 — Determinism and Honesty Policy

| Field | Value |
|-------|-------|
| **Source** | [`docs/adr/ADR-0003-determinism-and-honesty.md`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/docs/adr/ADR-0003-determinism-and-honesty.md) |
| **Status** | Accepted |
| **Date** | 2026-10-01 |
| **Decision** | Determinism = versioned inputs + pinned generation + constrained generation + reproducible build + immutable audit + human approval. Honesty = exact status line, no suppressed failures, no invented metrics. `DeterministicFakeProvider` only — no live LLM. |
| **Key invariant** | "Temperature=0 is a sampling setting. It does not guarantee factual correctness. That claim must never appear in documentation, UI, or commit messages." |
| **Residual risk** | "Replay test (Phase 8) is the only machine-verifiable proof of the determinism invariant. Until Phase 8 is complete, the invariant is stated policy, not proven fact." |

---

## 12. Threat Model

**OBSERVED in `docs/security/threat-model-v0.md` (Version 0.1, 2026-10-01):**

12 threats catalogued (T01–T12). All marked "Open" at Phase 1 baseline. The threat model is honest: "Phase 1 delivers no authentication, no database, no agent runtime, and no file handling."

| ID | Threat | Phase | Mitigation | Test Status |
|----|--------|------|------------|-------------|
| T01 | Authentication bypass | Phase 2 | argon2id; httpOnly cookie; server-side session | Unit test (mock) — PASSED |
| T02 | Authorization bypass (horizontal) | Phase 2 | PostgreSQL RLS; server-derived tenant context | Unit + integration test |
| T03 | Tenant breach via missing RLS | Phase 2 | Unset `app.current_tenant_id` → no rows (fail closed) | Integration test (CI) |
| T04 | Path traversal in object storage | Phase 2 | Object storage rejects `..` and absolute paths | NOT TESTED |
| T05 | Prompt injection via source document | Phase 6 | Injected text must not change the plan | Unit test — PASSED |
| T06 | Source immutability violation | Phase 3 | SHA-256 at upload; second write rejected | NOT TESTED |
| T07 | Audit record mutation | Phase 2 | DB trigger denies UPDATE/DELETE on audit_events | NOT TESTED |
| T08 | Self-approval (four-eyes) | Phase 7 | Author cannot be sole release approver | Unit test — PASSED |
| T09 | XSS in rendered HTML | Phase 8 | Allowlist of known content model keys; unknown dropped | Unit test — PASSED |
| T10 | SQL injection | Phase 2+ | Parameterized queries only; no string interpolation | Unit test — PASSED |
| T11 | Malicious MIME / oversized upload | Phase 3 | MIME sniff + declared type check; size cap | Unit test — PASSED |
| T12 | Secret leakage via logs | Phase 2 | Logger redacts secret-shaped strings | NOT TESTED |

**Limitations:** Threat model is v0.1 and scoped to Phase 1 skeleton. The release scorecard (OBSERVED in `docs/release/scorecard.md`) states: "Security coverage is unit-level only (mock doubles), not live penetration testing." 6 of 12 threats have unit-level test evidence; the rest are planned but their implementation status at HEAD is NOT TESTED.

---

## 13. CI and Test Evidence

### CI at HEAD

**OBSERVED via GitHub Actions API (2026-10-17T23:42:00Z):**

| Property | Value |
|----------|-------|
| Workflow | `CI` (`.github/workflows/ci.yml`) |
| Run | #43 |
| Head SHA | `e7cc7df20164f2806e93bed6f5a665865e72133b` |
| Conclusion | `success` |
| Created | 2026-10-17T03:38:27Z |

**CI scope (OBSERVED in `ci.yml`):**

1. PostgreSQL 16 service container (health-checked with `pg_isready`)
2. `pnpm install --frozen-lockfile`
3. `psql $SUPERUSER_URL -f scripts/ci-db-init.sql` (role initialization)
4. `node scripts/ci-migrate.mjs` (apply 14 migrations + seed)
5. `pnpm typecheck` (TypeScript strict across all packages)
6. `pnpm test` (Vitest across all packages; includes integration tests with live PostgreSQL)
7. `pnpm lint` (ESLint + Prettier, `--max-warnings 0`)

### Test File Inventory

| Package | Test files | Notes |
|---------|-----------|-------|
| `apps/api` | 6 | auth, ingest, logger, review, router, storage |
| `packages/db` | 20 | Includes 4 integration tests (agents, claims, rls, sources) |
| `packages/domain` | 24 | Includes golden fixtures, negative-evidence, timetravel, quote-lock-release |
| `packages/renderer` | 6 | beachhead, html, manifest, pdf, replay, watermark |
| **Total** | **56** | |

**Test count claims (REPORTED in README.md, not independently re-counted):**

| Claim | Source | Status |
|-------|--------|--------|
| "938 unit tests pass across 5 packages" | README.md | REPORTED — not independently verified |
| "apps/api: 90, packages/domain: 439, packages/db: 323, packages/renderer: 86" | README.md | REPORTED — per-package breakdown not verified |
| "101 integration tests are skipped without a live PostgreSQL instance" | README.md | REPORTED — integration tests run in CI (PostgreSQL provisioned); count not verified |
| "LiveModelProvider has 2 skipped tests (no API key in this env)" | README.md | REPORTED — not verified |

### Releases and Tags

| Property | Value |
|----------|-------|
| Published releases | 0 |
| Tags | 0 |
| Version in `package.json` | `1.0.0` |

No release artifact confirms the `1.0.0` version. The version string is in `package.json` only — not anchored to a published release. This is a version-claim gap (GAP-04-002).

---

## 14. Deployment Surface

### Render Blueprint (Candidate, Not Production)

**OBSERVED in `render.yaml`:**

| Service | Type | Plan | Notes |
|---------|------|------|-------|
| `alinamatrix-db` | PostgreSQL | starter | Render-managed PostgreSQL 16; migrations 001–014 applied by preDeployCommand |
| `alinamatrix-api` | web | starter | Node HTTP API; `node dist/server.js`; health check `/health`; autoDeploy: true |
| `alinamatrix-web` | web | starter | Next.js; `pnpm start`; health check `/`; autoDeploy: true |

The `render.yaml` explicitly states: "Render candidate — not production." Integration tests remain skipped without a live database. Secrets come from the host environment — no secret is committed.

### Docker Compose (Development Only)

**OBSERVED in `docker-compose.yml`:**

PostgreSQL 16 + Redis 7 for local development. Comment: "Phase 2: postgres and redis only. No application services in this file." This is a development dependency file, not a production deployment.

---

## 15. Case Study — Honest Blocker

**OBSERVED in `docs/case-study.md`:**

The case study page is a truthful blocker, not a fabricated success story:

> "No real customer. No organisation has deployed or used AlinaMatrix Enterprise in a production environment. The system has not been deployed to Render or any other host. No live PostgreSQL instance is running outside of local development."

This is an example of intellectual honesty in the source — the absence of a case study is itself documented as evidence.

---

## 16. Evidence Matrix

| Evidence ID | Claim | Source path | Observation class | Result status | Evidence strength | Notes |
|-------------|-------|-------------|-------------------|---------------|-------------------|-------|
| EVD-04-001 | DeterministicFakeProvider is deterministic — same inputs → same output | `packages/domain/src/agents.ts` | OBSERVED | PASS | E2 | Pure function; SHA-256 seed from pinned hashes; no randomness/network |
| EVD-04-002 | No live LLM is called — DeterministicFakeProvider only | `docs/adr/ADR-0003-determinism-and-honesty.md` + `packages/domain/src/agents.ts` | OBSERVED | PASS | E2 | ADR-0003 R07 directive enforced; DeterministicFakeProvider has no network path |
| EVD-04-003 | Three providers: DeterministicFakeProvider, PinnedModelProvider, LiveModelProvider | `packages/domain/src/{agents,pinned-model,live-model}.ts` | OBSERVED | PASS | E2 | Distinct classes with distinct invariants; LiveModelProvider env-gated + budget-enforced |
| EVD-04-004 | Four agents with frozen capabilities; release/export blocked for all | `packages/domain/src/agents.ts` | OBSERVED | PASS | E2 | `RELEASE_OPERATIONS` set checked unconditionally before per-agent check |
| EVD-04-005 | Workflow state machine enforces legal transitions (17 states) | `packages/domain/src/workflow.ts` | OBSERVED | PASS | E2 | `assertLegalTransition` throws `IllegalTransitionError`; terminal states have no outbound |
| EVD-04-006 | PostgreSQL RLS enabled and forced on 7 tenant-scoped tables | `packages/db/migrations/001_identity_tenancy_rls.sql` | OBSERVED | PASS | E2 | `FORCE ROW LEVEL SECURITY` on tenants, roles, users, memberships, projects, audit_events, sessions |
| EVD-04-007 | RLS fails closed — unset tenant_id returns no rows | `packages/db/migrations/001_identity_tenancy_rls.sql` | OBSERVED | PASS | E2 | `current_tenant_id()` returns NULL when not set; NULL = uuid → no rows |
| EVD-04-008 | Tenant context is server-derived; UUID-validated before DB | `packages/db/src/tenant-context.ts` | OBSERVED | PASS | E2 | UUID regex validation; SQL injection and path traversal rejected before `set_config` |
| EVD-04-009 | Four-eyes approval enforced — author cannot be sole approver | `packages/domain/src/review.ts` + `apps/api/src/review.ts` | OBSERVED | PASS | E2 | `assertFourEyes` throws `FourEyesError` when `authorId === approverId`; enforced at API boundary |
| EVD-04-010 | Release gate blocks on 8 conditions | `packages/domain/src/release.ts` | OBSERVED | PASS | E2 | Validations, critical findings, four-eyes, manifest, checksum, source versions, version pins, source job state |
| EVD-04-011 | Merkle provenance ledger with SHA-256 and tamper detection | `packages/domain/src/provenance.ts` | OBSERVED | PASS | E2 | `computeMerkleRoot` sorts leaves; `verifyProvenanceLedger` re-folds to detect tampering |
| EVD-04-012 | Audit events are append-only — UPDATE/DELETE denied by RLS | `packages/db/migrations/001_identity_tenancy_rls.sql` | OBSERVED | PASS | E2 | RLS on audit_events; migration comment confirms append-only policy |
| EVD-04-013 | Revocation writes an event; does not mutate published bytes | `packages/domain/src/revocation.ts` | OBSERVED | PASS | E2 | Pure function; `priorLabel` must be PUBLISHED/ARCHIVED; DB trigger protects published_html |
| EVD-04-014 | Budget enforcement with bigint precision (past 2^53) | `packages/domain/src/budget.ts` | OBSERVED | PASS | E2 | `bigint` comparison; `BudgetBreaker` refuses to call provider when over budget |
| EVD-04-015 | Guard validates M03 draft against schema, quote-lock, and 4 soft rules | `packages/domain/src/guard.ts` | OBSERVED | PASS | E2 | `runGuard` checks 6 rule types; `guardResultToTransition` routes to NEEDS_REVIEW on any block |
| EVD-04-016 | Negative evidence recording — absence is documented, not silent | `packages/domain/src/negative-evidence.ts` | OBSERVED | PASS | E2 | `note` must be non-empty; "silence is a violation" |
| EVD-04-017 | Export requires permission + watermark intact (4 meta tags) | `packages/domain/src/export.ts` | OBSERVED | PASS | E2 | `assertExportPermission` + `isWatermarkIntact` (named meta tag check, not bare substring) |
| EVD-04-018 | CI passes at HEAD (run #43, 2026-10-17) | `.github/workflows/ci.yml` | OBSERVED | PASS | E3 | GitHub Actions API: `conclusion: success`; CI includes PostgreSQL 16, migrations, typecheck, test, lint |
| EVD-04-019 | 56 test files across 4 packages | tree at SHA `e7cc7df20164` | OBSERVED | PASS | E2 | apps/api: 6, packages/db: 20, packages/domain: 24, packages/renderer: 6 |
| EVD-04-020 | "938 unit tests pass" claim | `README.md` | REPORTED | NOT_TESTED | E1 | Not independently re-counted; CI runs tests but individual count not verified |
| EVD-04-021 | 14 SQL migrations (forward-only) | `packages/db/migrations/001–014` | OBSERVED | PASS | E2 | 001 identity/tenancy/RLS through 014 consensus decision events |
| EVD-04-022 | Threat model with 12 threats (T01–T12) | `docs/security/threat-model-v0.md` | OBSERVED | PASS | E2 | All "Open" at Phase 1; mitigations planned per phase; 6 have unit test evidence |
| EVD-04-023 | Expert acceptance protocol — all scores blank | `docs/eval/expert-acceptance-protocol.md` | OBSERVED | PASS | E2 | No score pre-filled; reviewer fields blank; no expert review performed |
| EVD-04-024 | Case study is a truthful blocker — no real customer | `docs/case-study.md` | OBSERVED | PASS | E2 | "No real customer. No organisation has deployed or used AlinaMatrix in production." |
| EVD-04-025 | Status is "Enterprise Candidate — Active Development" | `README.md` + `ADR-0003` | OBSERVED | PASS | E2 | ADR-0003: "No other status is permitted until a separate revision of this directive." |
| EVD-04-026 | No license file — GitHub API reports `license: null` | repo root | OBSERVED | BLOCKED | E2 | No LICENSE file in tree. Copying source requires explicit owner authorization. |
| EVD-04-027 | No published releases or tags | GitHub API releases + tags | OBSERVED | PASS | E2 | 0 releases, 0 tags. Version 1.0.0 in `package.json` only — not anchored to a release. |
| EVD-04-028 | Render Blueprint is a candidate, not production | `render.yaml` | OBSERVED | PASS | E2 | Explicit comment: "Render candidate — not production." |
| EVD-04-029 | Docker Compose is development-only (postgres + redis) | `docker-compose.yml` | OBSERVED | PASS | E2 | Comment: "Phase 2: postgres and redis only. No application services." |
| EVD-04-030 | Determinism invariant is stated policy, not proven fact | `docs/adr/ADR-0003-determinism-and-honesty.md` | OBSERVED | CONDITIONAL | E2 | ADR residual risk: "Replay test (Phase 8) is the only machine-verifiable proof." Phase 8 status NOT TESTED. |

---

## 17. Key Source File Reference

| File | Role | Permalink |
|------|------|-----------|
| [`packages/domain/src/agents.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/agents.ts) | Four-agent capabilities, DeterministicFakeProvider, cache key, release/export block |
| [`packages/domain/src/live-model.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/live-model.ts) | LiveModelProvider — env-gated, budget-enforced, schema-validated |
| [`packages/domain/src/pinned-model.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/pinned-model.ts) | PinnedModelProvider — deterministic with pinned config, budget-enforced |
| [`packages/domain/src/workflow.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/workflow.ts) | 17-state workflow state machine with legal transition enforcement |
| [`packages/domain/src/review.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/review.ts) | Four-eyes approval, claim-level review, canCompleteReview |
| [`packages/domain/src/release.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/release.ts) | Release gate (8 conditions), label lifecycle |
| [`packages/domain/src/guard.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/guard.ts) | M03 draft validation (schema, quote-lock, 4 soft rules) |
| [`packages/domain/src/provenance.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/provenance.ts) | Merkle provenance ledger (SHA-256, sorted leaves, tamper detection) |
| [`packages/domain/src/revocation.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/revocation.ts) | Revocation event — pure function, no published-byte mutation |
| [`packages/domain/src/budget.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/budget.ts) | Budget enforcement with bigint precision |
| [`packages/domain/src/export.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/export.ts) | Export permission + watermark intact verification (named meta tags) |
| [`packages/domain/src/negative-evidence.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/domain/src/negative-evidence.ts) | Negative evidence recording — absence documented, not silent |
| [`packages/db/src/tenant-context.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/db/src/tenant-context.ts) | Server-derived tenant context; UUID-validated; fail-closed |
| [`packages/db/migrations/001_identity_tenancy_rls.sql`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/packages/db/migrations/001_identity_tenancy_rls.sql) | Identity, tenancy, RLS with FORCE, append-only audit |
| [`apps/api/src/extractor.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/apps/api/src/extractor.ts) | Text extractor (PDF/DOCX/txt/md); fails with ExtractionError, never invents |
| [`apps/api/src/auth.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/apps/api/src/auth.ts) | argon2id auth; httpOnly cookie; server-side session |
| [`apps/api/src/review.ts`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/apps/api/src/review.ts) | HTTP handlers for Phase 7 human review; four-eyes at API boundary |
| [`docs/adr/ADR-0003-determinism-and-honesty.md`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/docs/adr/ADR-0003-determinism-and-honesty.md) | Determinism policy, honesty policy, DeterministicFakeProvider directive |
| [`docs/security/threat-model-v0.md`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/docs/security/threat-model-v0.md) | 12-threat model (T01–T12); all open at Phase 1 |
| [`docs/release/scorecard.md`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/docs/release/scorecard.md) | Release scorecard: FAIL, 5 blockers (2 closed, 3 open) |
| [`.github/workflows/ci.yml`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/.github/workflows/ci.yml) | CI: PostgreSQL 16, migrations, typecheck, test, lint |
| [`render.yaml`](https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise/blob/e7cc7df20164f2806e93bed6f5a665865e72133b/render.yaml) | Render Blueprint — candidate, not production |

---

## 18. Open Gaps

| Gap ID | Severity | Description |
|--------|----------|-------------|
| GAP-04-001 | HIGH | No license file. Source code cannot be copied into the portfolio without explicit owner authorization. All evidence in this file uses links and attributed descriptions only — no source code is reproduced. |
| GAP-04-002 | MEDIUM | Version 1.0.0 in `package.json` is not anchored to a published release. 0 releases, 0 tags. The version claim is a source file assertion, not a release artifact. |
| GAP-04-003 | MEDIUM | "938 unit tests pass" and per-package breakdown (90/439/323/86) are REPORTED in README.md, not independently re-counted. CI runs the tests but the individual count was not verified. |
| GAP-04-004 | MEDIUM | Determinism invariant is stated policy (ADR-0003), not proven fact. ADR residual risk: "Replay test (Phase 8) is the only machine-verifiable proof." Phase 8 replay test status at HEAD is NOT TESTED. |
| GAP-04-005 | LOW | Threat model is v0.1 and scoped to Phase 1 skeleton. 6 of 12 threats have unit test evidence; the rest are planned but implementation status at HEAD is NOT TESTED. |
| GAP-04-006 | LOW | Security coverage is unit-level (mock doubles) — not live penetration testing. Release scorecard (OBSERVED) states this explicitly. |
| GAP-04-007 | LOW | No regional disaster recovery. Backup drill (R5) passed against local Docker PostgreSQL 16, but no DR site, no cross-region replication, no PITR. |
| GAP-04-008 | INFO | Expert acceptance protocol has all blank scores — no expert review has been performed. This is honest, not a gap in honesty. |

---

## 19. Reproducibility Recipe (R1 → R3 path)

This is a documented path to reach R3 independently. It has NOT been executed (not authorized per DIRECTIVE.txt F03/G/J — source repositories are read-only).

```bash
git clone https://github.com/aminbita162-glitch/AlinaMatrix-Enterprise.git
cd AlinaMatrix-Enterprise
git checkout e7cc7df20164f2806e93bed6f5a665865e72133b  # HEAD SHA
# Requires: Node.js 22+, pnpm 9+, PostgreSQL 16 (for integration tests)

# Install
pnpm install --frozen-lockfile

# Start PostgreSQL (for integration tests)
docker compose up -d postgres  # or use a local PostgreSQL 16

# Initialize roles + apply migrations + seed
psql "$SUPERUSER_URL" -f scripts/ci-db-init.sql
node scripts/ci-migrate.mjs

# Static checks
pnpm typecheck   # TypeScript strict across all packages
pnpm lint        # ESLint + Prettier (--max-warnings 0)

# Tests (unit + integration with live PostgreSQL)
pnpm test        # Vitest across all packages

# Expected (REPORTED in README.md): 938 unit + 101 integration pass
# CI at HEAD e7cc7df20164 is SUCCESS (run #43)
```

Preconditions: Node.js 22+; pnpm 9+; PostgreSQL 16 for integration tests; no API key needed (DeterministicFakeProvider only; LiveModelProvider tests are skipped without a key).

**CI reproduction note:** CI at HEAD `e7cc7df20164` is SUCCESS (run #43, 2026-10-17T03:38:27Z). CI provisions PostgreSQL 16, applies migrations, seeds, and runs typecheck + test + lint. Reproduction at HEAD should produce a passing run.
