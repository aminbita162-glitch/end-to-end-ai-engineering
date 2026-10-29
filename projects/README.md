# Inherited Engineering Assets — Legacy Assessment

**Phase:** 7 — Technical Reviews, Education, Legacy Assets
**Last updated:** 2026-10-18
**Observation date:** 2026-10-18
**Context:** Inherited engineering assets predating the evidence-first portfolio methodology; no independent validation claimed.

## Purpose

This document assesses the `projects/` directory — inherited engineering assets that predate the evidence-first portfolio methodology applied across the 13 cases. The assessment evaluates provenance, duplication, and relevance against the current portfolio structure.

## Directory Inventory

| Directory | Files | Status | Content |
|-----------|-------|--------|---------|
| `01-ai-product-engineering/` | ~40 files | Inherited — has a redirect to `cases/04-lieferkette-optimierungsplattform/` | Supply chain optimization sub-project with FastAPI backend, Docker, tests, architecture docs, evidence screenshots |
| `02-ai-assistant-project/` | ~80+ files | Inherited — has a redirect to `cases/05-personal-ai-assistant/` | Full-stack AI assistant platform with FastAPI backend, PostgreSQL, Redis, Docker Compose, API docs, evidence screenshots |
| `03-intelligent-automation/` | ~10 files (mostly .gitkeep) | Inherited — redirect to `cases/` | Skeleton with placeholder directories (assets, docs, evidence) — no project content |
| `04-data-apis-decision-systems/` | 1 file (README.md) | Inherited — redirect to `cases/` | Standards/framework document only — no project implementation |
| `05-deployment-testing-reliability/` | 1 file (README.md) | Inherited — redirect to `cases/` | Standards/framework document only — no project implementation |

## Provenance and Duplication Assessment

### 01-ai-product-engineering/

**Provenance:** Predates the evidence-first portfolio. Contains a `ai-powered-supply-chain-optimization/` sub-project with its own README, FastAPI app (`app/main.py`, `app/api/inventory.py`, `app/models/inventory.py`, etc.), Docker setup, test suite, architecture documentation, and evidence screenshots (GitHub repository, VS Code implementation, Swagger API, GitHub Actions CI).

**Duplication:** The top-level README contains a redirect note pointing to `cases/04-lieferkette-optimierungsplattform/` — but the sub-project's content is a *different* project (an inventory management API, not the Lieferkette logistics platform). The supply chain optimization sub-project appears to be an earlier standalone project that predates the 13-case structure. Its relationship to PRJ-03 (IBM Supply Chain Challenge) or PRJ-08 (Lieferkette) is unclear — it may be an earlier iteration of the IBM challenge project or a separate educational exercise.

**Relevance:** The sub-project demonstrates real engineering work (FastAPI, SQLAlchemy, Pydantic schemas, Docker, tests, architecture docs). However, it is not one of the 13 canonical cases and its provenance predates the evidence methodology. The redirect at the top-level README points to a case that does not match its content.

**Recommendation:** Preserve as-is. Do not modify. The content is inherited and predates the current methodology. It could be integrated or archived later.

### 02-ai-assistant-project/

**Provenance:** Predates the evidence-first portfolio. Contains a full-stack AI assistant platform with a substantial FastAPI backend (auth, conversations, messages, OpenAI integration, PostgreSQL/Redis, Docker Compose), API documentation, security model, and evidence screenshots (Render deployment, Docker Swagger, VS Code implementation).

**Duplication:** The top-level README contains a redirect note pointing to `cases/05-personal-ai-assistant/` — but the content is a *different* project. This is a full-stack assistant platform with JWT auth, conversation management, and Docker Compose, while PRJ-09 (Personal AI Assistant) is a simpler Flask MVP with psycopg2 raw SQL. The redirect is incorrect; this project has more architectural depth than PRJ-09 but is not one of the 13 canonical cases.

**Relevance:** The sub-project demonstrates real engineering work (layered architecture, async SQLAlchemy, OpenAI SDK integration, Docker Compose, health checks, security model). It predates the evidence methodology and its relationship to the 13 cases is not established.

**Recommendation:** Preserve as-is. Do not modify. The redirect is cosmetically present but the content is independent inherited work.

### 03-intelligent-automation/

**Provenance:** Skeleton directory with placeholder `.gitkeep` files in subdirectories (assets/demo, assets/diagrams, assets/images, docs/architecture, docs/decisions, docs/operations, docs/product, docs/security, docs/testing, evidence/logs, evidence/screenshots, evidence/test-results). No project implementation.

**Duplication:** None — no content to duplicate. The README is a standards/framework document with a redirect to `cases/`.

**Relevance:** Structural placeholder only. No engineering evidence.

**Recommendation:** Preserve as-is. No content to modify.

### 04-data-apis-decision-systems/

**Provenance:** Standards/framework document only. No project implementation. README covers data engineering standards, API engineering standards, decision-system models, and reliability principles.

**Duplication:** None — no project content. The README is a methodology document, not a case study.

**Relevance:** Methodology reference material. Not a case study.

**Recommendation:** Preserve as-is. No content to modify.

### 05-deployment-testing-reliability/

**Provenance:** Standards/framework document only. No project implementation. README covers testing strategy, deployment engineering, CI/CD, observability, reliability principles, security validation, performance, and recovery — a comprehensive standards reference.

**Duplication:** None — no project content. The README is a methodology document, not a case study.

**Relevance:** Methodology reference material. Not a case study.

**Recommendation:** Preserve as-is. No content to modify.

## Overall Assessment

The `projects/` directory contains 5 inherited directories:
- 2 contain real engineering implementations that predate the evidence-first methodology (01, 02).
- 3 contain only standards/framework documents or empty placeholder structures (03, 04, 05).

**No source code was copied or modified in this assessment.** All content is preserved as-is. The inherited assets predate the current portfolio structure and are not part of the 13 canonical cases. Their provenance, duplication status, and relevance are recorded here.

**Key finding:** The redirect notes in `01-ai-product-engineering/README.md` and `02-ai-assistant-project/README.md` point to cases that do not match the inherited content. These redirects were added during an earlier portfolio restructuring but the inherited sub-projects are distinct from the 13 canonical cases. This is a cosmetic discrepancy, not a content error — the inherited content is preserved and the redirect is a navigation aid, not a claim of equivalence.

**Decision required:** none. These inherited assets are preserved as they are.
