# Phase 1 Source Traceability Notes
**Phase:** 1 — Source Traceability Inventory\
**Observation date (UTC):** 2026-10-17
**Portfolio HEAD at phase start:** `0e8a84cd20322a48da0b722981d1e535394ee6c3`\
**Independence:** Source/documentation observations only; no independent third-party verification claimed.\
**Independence level:** Source/documentation observations only; no independent reproduction claimed.\
**Method:** GitHub public API (unauthenticated, rate-limited) + raw content fetch for key files\

---

## 1. Baseline Recheck

| Item | Phase 0 Baseline | Phase 1 Observed | Drift |
|------|-----------------|------------------|-------|
| Portfolio branch | main | main | NONE |
| Portfolio HEAD | a02bad5eacbd2467523c75f2290f520dd547089f | 0e8a84cd20322a48da0b722981d1e535394ee6c3 | **3 new commits ahead of a02bad5** |
| Working tree | Clean | Clean | NONE |
| cases/ count | 13 | 13 | NONE |
| docs/evidence/ | .gitkeep only | Phase 1 deliverables added | EXPECTED |

**Drift assessment:** The portfolio HEAD has advanced from `a02bad5` (the Phase 0 baseline) to `0e8a84c` via three owner-authored commits:
- `673ebbd` Update DIRECTIVE.txt\
- `215f0af` docs: establish engineering atlas master directive\
- `0e8a84c` Merge origin/main into local main (history reconciliation)

These are owner-authored. No source repositories were changed. The baseline advance is expected and does not invalidate this phase.

---

## 2. Source Repository Access Summary

| PRJ | Source URL | HTTP Access | Source Type |
|-----|-----------|-------------|-------------|
| PRJ-01 | github.com/azimilab2025-ai/ailora | **ACCESSIBLE** | GitHub public |
| PRJ-02 | github.com/aminazimi42-coder/aegis-agent | **ACCESSIBLE** | GitHub public |
| PRJ-03 | github.com/azimilab2025-ai/IBM-Bob-Challenge-2026 | **ACCESSIBLE** | GitHub public |
| PRJ-04 | github.com/aminbita162-glitch/AlinaMatrix-Enterprise | **ACCESSIBLE** | GitHub public |
| PRJ-05 | github.com/aminbita162-glitch/kian-trading-intelligence | **ACCESSIBLE** | GitHub public |
| PRJ-06 | github.com/aminbita162-glitch/ahmad-amin-technology | **ACCESSIBLE** | GitHub public |
| PRJ-07 | nimaos-ai-agent.vercel.app/translator | **NO GITHUB REPO** | Vercel URL only |
| PRJ-08 | github.com/aminbita162-glitch/lieferkette-optimierungsplattform | **ACCESSIBLE** | GitHub public |
| PRJ-09 | github.com/aminbita162-glitch/personal-ai-assistant | **ACCESSIBLE** | GitHub public |
| PRJ-10 | github.com/aminbita162-glitch/restaurant-ai-platform | **ACCESSIBLE** | GitHub public |
| PRJ-11 | github.com/aminbita162-glitch/buro-assistant | **ACCESSIBLE** | GitHub public |
| PRJ-12 | github.com/aminbita162-glitch/patch-auri | **ACCESSIBLE** | GitHub public |
| PRJ-13 | azimi-innovation-lab-website.vercel.app | **NO GITHUB REPO** | Website URL only |

11/13 sources have accessible GitHub repositories. PRJ-07 and PRJ-13 have no known source repository.

**Rate limit note:** GitHub unauthenticated API exhausted during inspection (60 req/hr). IBM commit SHA `e39d3b1df0b9` could not be independently verified. Marked BLOCKED/NOT_TESTED.

---

## 3. Default Branch SHA Register

| PRJ | Canonical Name | Default Branch | HEAD SHA (observed 2026-10-17) |
|-----|---------------|---------------|-------------------------------|
| PRJ-01 | AILORA | main | `46ca4a939bee0396e90abef91905b6d1b95d470c` |
| PRJ-02 | Aegis Agent | main | `69f159771c35b04b250fd282189f99b06de0c76d` |
| PRJ-03 | IBM Supply Chain Challenge | main | `d00c5ebb27948ce9e61cd63add127868f65e36dd` |
| PRJ-04 | AlinaMatrix Enterprise | main | `e7cc7df20164f2806e93bed6f5a665865e72133b` |
| PRJ-05 | Kian Trading Intelligence | main | `54ef6ca4d8fd1b2edb0d8fba921c3451ebc395bb` |
| PRJ-06 | Ahmad & Amin Technology | main | `fbbfc307fd98bdad05ac02c3f2813b39368c5bc6` |
| PRJ-07 | NimaOS AI Agent | N/A | NOT_AVAILABLE — no source repo |
| PRJ-08 | Lieferkette Optimierungsplattform | main | `91c61210e9134302f17a634c2b830cadadf5c4bc` |
| PRJ-09 | Personal AI Assistant | main | `455a3ebb4f84a6ca648d7a8481f864b21680f235` |
| PRJ-10 | Restaurant AI Platform | main | `c49649a0b44e59d17407f11269dd0ec8cfc1d5fa` |
| PRJ-11 | Büro Assistant | main | `893db187185dbed7b4dafd461241eb78acb07f8e` |
| PRJ-12 | Patch-Auri | main | `93ea2196e7dbadb1f4e304dc7a53fad6830f8a14` |
| PRJ-13 | Azimi Innovation Lab Website | N/A | NOT_AVAILABLE — website only |

These SHAs are immutable at time of observation and serve as the canonical inspection baseline for subsequent phases.

---

## 4. CI / Test Maturity Matrix

| PRJ | CI Workflow | Latest Run | Test Files Present | Notes |
|-----|------------|-----------|-------------------|-------|
| PRJ-01 | YES (ci.yml) | **FAIL** | YES | Recent CI failure on HEAD |
| PRJ-02 | YES (ci.yml + release.yml + security.yml) | **ALL FAIL** | YES | Triple failure at HEAD |
| PRJ-03 | NO | N/A | YES | Verification.md cites 69 tests but no automated CI |
| PRJ-04 | YES (ci.yml) | **PASS** | YES | TypeScript vitest structure |
| PRJ-05 | YES (ci.yml) | **PASS** (prior 2 FAIL) | YES | Recently fixed |
| PRJ-06 | NO | N/A | MINIMAL | Phase 1 PASS claimed without CI |
| PRJ-07 | NO | N/A | NONE | No source |
| PRJ-08 | YES (python-tests.yml) | **PASS** | YES (1 file) | Very limited coverage |
| PRJ-09 | NO | N/A | NONE observed | Stale codebase |
| PRJ-10 | YES (ci.yml Smoke) | **PASS** | YES (9+ files) | Best observed test structure |
| PRJ-11 | NO | N/A | YES (tests/) | Rich docs, no automated CI |
| PRJ-12 | NO | N/A | NONE | 6-file repo |
| PRJ-13 | NO | N/A | NONE | Website only |

**Key finding:** Only 5/13 projects have active CI. Of those, 2 (PRJ-01, PRJ-02) have failing CI at HEAD. PRJ-10 (Restaurant) has the most systematic test file structure observed outside the IBM flagship.

---

## 5. License Register

| PRJ | SPDX ID | Notes |
|-----|---------|-------|
| PRJ-01 | NOASSERTION | License file present but not SPDX-recognized |
| PRJ-02 | NOASSERTION | License file present but not SPDX-recognized |
| PRJ-03 | MIT | Confirmed MIT |
| PRJ-04 | NONE | **No license file detected** — IP risk for excerpts |
| PRJ-05 | NOASSERTION | License file present but not SPDX-recognized |
| PRJ-06 | NOASSERTION | License file present but not SPDX-recognized |
| PRJ-07 | UNKNOWN | No source repo |
| PRJ-08 | NONE | **No license file detected** |
| PRJ-09 | NONE | **No license file detected** |
| PRJ-10 | MIT | Confirmed MIT |
| PRJ-11 | NOASSERTION | License file present but not SPDX-recognized |
| PRJ-12 | NONE | **No license file detected** |
| PRJ-13 | UNKNOWN | Website only |

**Advisory:** PRJ-04, PRJ-08, PRJ-09, PRJ-12 have no license file. Source code from these projects is not reproduced in the portfolio; links are used instead.

---

## 6. Release Artifact Register

| PRJ | Release Tags | Latest Tag | Date | Notes |
|-----|-------------|-----------|------|-------|
| PRJ-01 | 3 | `candidate-2026-08-29` | 2026-08-29 | `candidate-2026-08-30` is pre-release; `v0.1.0` earliest |
| PRJ-02 | 0 | NONE | N/A | Portfolio claims rc1 — no matching tag found |
| PRJ-03 | 1 | `v1.0.0-submission-final` | 2026-07-15 | Matches submission date |
| PRJ-04–13 | 0 | NONE | N/A | No releases for any remaining projects |

---

## 7. Priority-Ranked Evidence Gaps

### CRITICAL (must address before Phase 3 flagship cases)

**GAP-01-001** (PRJ-01): AILORA CI failing at HEAD (`46ca4a939bee`). Portfolio does not claim CI passes, but a failing HEAD is a material evidence gap. Must determine whether failure is test infrastructure or functional regression before writing deep engineering case.

**GAP-02-001** (PRJ-02): Aegis Agent ALL THREE workflows (CI, Release, Security) failing at HEAD (`69f159771c35`). This is the most severe CI gap observed. No release tag matching the portfolio's "1.0.0-rc1" claim. Human approval enforcement requires source inspection of `core/tool_tokens.py` and `core/decision_ledger.py` — the core architectural claim cannot be rated above REPORTED without this inspection.

**GAP-03-001** (PRJ-03): IBM challenge commit SHA `e39d3b1df0b9` cited in `verification.md` could not be verified (API rate limit). The "69 tests passing" claim is REPORTED/NOT_TESTED until this SHA is confirmed to exist and the test files are observed. Must verify before making assertions about test coverage.

### HIGH (address in Phase 4 case work)

**GAP-04-001** (PRJ-04): AlinaMatrix Enterprise has no license file. No source code excerpts are used in the portfolio. DeterministicFakeProvider boundary must be inspected in source (extractor.ts) before claiming separation is enforced.

**GAP-05-001** (PRJ-05): Kian LIVE mode enforcement is REPORTED only. `risk_kernel/kernel.py` must be inspected to determine if simulation-only is a code-enforced guard or a documented intent. `test-house/financial-safety` is empty (`.gitkeep`).

**GAP-06-001** (PRJ-06): Ahmad & Amin "Phase 1 PASS" claim has no CI workflow and only minimal test files observed. REPORTED evidence only. Portfolio claim exceeds observable source evidence.

### MEDIUM (address in Phase 5–6 case work)

**GAP-08-001** (PRJ-08): Lieferkette has only one test file (`test_kalender.py`) despite CI passing. Test coverage is almost certainly insufficient for the claimed scope (demand forecasting, inventory, warehouse, route optimization). No license.

**GAP-09-001** (PRJ-09): Personal AI Assistant last pushed April 2026 (5+ months stale at inspection). No CI, no test files, no license, no live URL. MVP claim is the most thinly evidenced case.

**GAP-11-001** (PRJ-11): Büro Assistant claims "Phase 9 closed / 1.0.0" but has no GitHub release tag. `docs/releases/` commercial phase docs exist but have not been inspected. The `decision_hash.py` + `send_decision.py` policy pattern suggests architectural depth not reflected in the portfolio case.

**GAP-10-001** (PRJ-10): Restaurant AI Platform Smoke CI passes but test scope is "smoke" only. `test_fail_closed.py` is a promising negative-case indicator that should be highlighted in Phase 4 deep dive.

### LOW (address in Phase 6 + Phase 7)

**GAP-07-001** (PRJ-07): NimaOS has no source repository. All architectural claims are E0/REPORTED. Live URL accessibility unverified.

**GAP-12-001** (PRJ-12): Patch-Auri has only 6 files. NFC/color implementation is entirely REPORTED. Minimal source evidence for any architectural claim.

**GAP-13-001** (PRJ-13): Website source repository unknown. Only URL is known evidence. Cannot make any source-level assessment.

---

## 8. Source-to-Claim Consistency Findings

### Inconsistencies requiring portfolio attention

| # | Project | Portfolio Claim | Source Evidence | Verdict |
|---|---------|----------------|-----------------|---------|
| 1 | PRJ-02 Aegis | Status "1.0.0-rc1" | Zero GitHub release tags found | **GAP** — version claim unanchored to release artifact |
| 2 | PRJ-02 Aegis | CI/workflows present | All 3 workflows: FAILURE at HEAD | **OBSERVED FAIL** — not a portfolio claim but material evidence gap |
| 3 | PRJ-01 AILORA | CI present | Latest CI run: FAILURE | **OBSERVED FAIL** |
| 4 | PRJ-06 Ahmad | "Phase 1 PASS" | No CI, minimal tests | **REPORTED ONLY** — evidence strength E1 at best |
| 5 | PRJ-03 IBM | "69 tests passing" | Verification.md states it; CI absent; commit SHA BLOCKED | **REPORTED/NOT_TESTED** |

### Claims consistent with source evidence

- AILORA `status-contract.md` correctly documented as qualification gates OBSERVED in tree
- AlinaMatrix DeterministicFakeProvider and no-live-deployment consistent
- Kian simulation-only status consistent with tree (no live exchange code visible)
- Restaurant Phase 1 baseline consistent with observed test file structure
- Büro payment-not-live consistent with no live URL
- Patch-Auri non-medical consistently stated

---

## 9. Architectural Depth Signals (per project)

The following are directory/file patterns observed that indicate architectural sophistication, to guide Phase 3–8 deep dives:

**PRJ-01 AILORA** (359 files): `alembic/` (DB migrations), `docs/governance/enterprise-requirements-traceability.json`, `docs/qualification/` (12+ qualification docs), `docs/qualification/status-contract.md`, `docs/qualification/final-release-manifest.json`, observability tracing (`src/ailora/observability/tracing.py`), `docs/qualification/devsecops-supply-chain.md`, `docs/qualification/real-system-qualification-record.md`.

**PRJ-02 Aegis Agent** (524 files): `core/tool_tokens.py` (tool allowlist evidence), `core/decision_ledger.py`, `core/twin_audit.py`, `core/circuit_breaker.py`, `core/reversible_workflow.py`, `core/security.py`, `agents/` (6 named agents), `charts/aegis-agent/` (Helm chart), `docs/AUDIT_T127.md`. Largest tree by file count — substantial implementation.

**PRJ-03 IBM** (146 files): `backend/tests/` (8 test modules across api/ and unit/), `scripts/evaluate_ai.py`, `backend/app/core/security.py`, `backend/migrations/`. Contained but well-organized structure for a challenge submission.

**PRJ-04 AlinaMatrix** (259 files): `docs/adr/` (3 ADRs), `apps/api/src/` (6 test files), `docs/eval/expert-acceptance-protocol.md`, `docs/phases/` (level-up reports), `docs/case-study.md`. Strongest documentation discipline of the smaller projects.

**PRJ-05 Kian** (285 files): `services/risk_kernel/`, `packages/contracts/risk.py` + `risk_token.py` + `security.py`, `formal/`, `simulation/security_simulators.py`, `test-house/phase-01..07/` evidence files, `apps/web/tests/` (7+ test files including `client-contracts`, `identity`, `remote-commands`).

**PRJ-10 Restaurant** (48 files): Most systematic test file naming of smaller projects — `test_fail_closed.py`, `test_r1..r8`, `test_g2_worker_queue.py`, `test_r7_tenant_token.py`. MIT licensed.

**PRJ-11 Büro** (175 files): `app/agents/decision_hash.py`, `app/policy/send_decision.py`, `migrations/` (3+ migration files), `docs/releases/commercial-phase-1..5`, `docs/TEST_HOUSE.md`. More architectural depth than portfolio case suggests.
