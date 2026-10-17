# Lieferkette Optimierungsplattform

**Status:** `Actively evolving · German logistics decision-support platform`

A German-language logistics decision-support platform providing demand forecasting, inventory management, warehouse operations, and route optimization heuristics.

## What It Is

An actively evolving decision-support platform for German logistics operations. Covers demand forecast, inventory, warehouse management, and route heuristic modules. Live API and dashboard available.

## What It Is Not

- Not the IBM TechXchange 2026 Challenge project (that is Supply Chain #1, a separate system).
- Not a completed enterprise product.
- Not production-ready in the sense of a hardened, certified deployment.

## Problem

German logistics operations require integrated decision support across demand forecasting, inventory control, warehouse routing, and transport optimization. Traditional tools treat these in isolation.

## Scope

End-to-end logistics decision-support covering:
- Demand forecasting
- Inventory optimization
- Warehouse management
- Route heuristics

Actively evolving. Not a completed enterprise product.

## Architecture

FastAPI backend providing REST API endpoints. Dashboard UI for operational visualization. Interactive API documentation. Health check endpoint.

## Engineering Decisions

- Actively evolving codebase: scope and features expand as logistics use cases are validated.
- German-language domain focus: naming, business rules, and documentation oriented toward German logistics context.
- Heuristic-based routing: optimization through practical heuristics rather than exact solvers.

## Stack

| Layer | Technology |
|-------|-----------|
| API Framework | FastAPI |
| Deployment | Render |
| Documentation | OpenAPI / Swagger |

Full stack documented in source repository.

## Testing

Evidence available in the source repository. Refer to repository for current test status given the actively evolving nature of the platform.

## Deployment Model

Hosted on Render. Persistent live API, dashboard, docs, and health endpoints.

## Live Links

| Resource | URL |
|----------|-----|
| Live API | https://lieferkette-optimierungsplattform.onrender.com |
| Dashboard | https://lieferkette-optimierungsplattform.onrender.com/dashboard/ |
| Docs | https://lieferkette-optimierungsplattform.onrender.com/docs |
| Health | https://lieferkette-optimierungsplattform.onrender.com/health |
| Demo | https://youtu.be/hUALYUOO_sI |

## Technical Evidence

Refer to the source repository. Portfolio engineering evidence for this project is in:
[`projects/01-ai-product-engineering/ai-powered-supply-chain-optimization/`](../../projects/01-ai-product-engineering/ai-powered-supply-chain-optimization/)

That folder contains architecture documentation, API specifications, testing evidence, deployment artifacts, and screenshots accumulated during development.

## Measurable Results

No metrics claimed beyond what the source repository and portfolio evidence folder document.

## Source Repository

https://github.com/aminbita162-glitch/lieferkette-optimierungsplattform
