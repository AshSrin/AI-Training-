# AI Automation Inventory Agent — Phase 1

## Goal

Deliver a runnable project scaffold, OpenAPI contract, and a Fastify API returning mock data — so the dashboard (Phase 2) can be built against real endpoints immediately.

---

## 1. What Phase 1 Produces

| Deliverable | Description |
|---|---|
| TypeScript project | Node.js project with tsconfig, eslint, vitest |
| Sample monorepo fixture | 3 mock apps with Playwright specs, page objects, components |
| OpenAPI spec | `openapi.yaml` — single source of truth for all endpoints |
| Fastify API | All routes returning mock data on port 4000 |
| MockStore | In-memory data store implementing `IStore` interface |
| CLI entry point | `automation-agent serve` starts the API server |

---

## 2. Project Structure (Phase 1 Scope)

```
src/
├── models/              TypeScript interfaces (zero dependencies)
│   ├── application.ts
│   ├── test.ts
│   ├── feature.ts
│   ├── component.ts
│   ├── relationship.ts
│   ├── duplicate.ts
│   ├── coverage.ts
│   ├── health.ts
│   ├── gap.ts
│   └── index.ts
│
├── store/               Data access layer
│   ├── IStore.ts         Interface definition
│   └── mock-store.ts     In-memory implementation with fixture data
│
├── api/                 Fastify HTTP layer
│   ├── server.ts
│   └── routes/
│       ├── inventory.route.ts
│       ├── coverage.route.ts
│       ├── duplicates.route.ts
│       ├── gaps.route.ts
│       ├── health.route.ts
│       ├── graph.route.ts
│       ├── scan.route.ts
│       └── trends.route.ts
│
├── cli/
│   ├── index.ts
│   └── serve.command.ts
│
└── config.ts

fixtures/
└── sample-monorepo/
    ├── apps/
    │   ├── checkout/
    │   ├── catalog/
    │   └── account/
    ├── tests/
    ├── pages/
    └── components/

openapi.yaml
```

**Not in scope for Phase 1:** `engines/`, `orchestrator/`, `ai/`, `dashboard/`, `sqlite-store.ts`

---

## 3. Models

Plain TypeScript interfaces. No logic.

| Model | Fields |
|-------|--------|
| Application | id, name, path |
| TestRecord | id, title, filePath, describeBlock, applicationId, featureId, status |
| Feature | id, name, applicationId, inferredBy |
| Component | id, name, filePath, applicationId |
| Relationship | sourceId, targetId, edgeType |
| Duplicate | id, testIdA, testIdB, type, confidenceScore, recommendation |
| CoverageReport | applicationId, featureId, coveragePct, amiScore, depth, recordedAt |
| HealthRecord | testId, passRate, avgDuration, flakinessScore, lastRun |
| GapSuggestion | id, applicationId, featureId, priority, suggestedTest, reason |
| ScanJob | id, status, startedAt, completedAt |

---

## 4. Store Interface

```
IStore
├── getApplications(filters?)        → Application[]
├── upsertApplications(apps)         → void
├── getTests(filters?, pagination?)  → { data: TestRecord[], total: number }
├── upsertTests(tests)               → void
├── getFeatures(appId?)              → Feature[]
├── upsertFeatures(features)         → void
├── getRelationships(nodeId?)        → Relationship[]
├── upsertRelationships(rels)        → void
├── getDuplicates()                  → Duplicate[]
├── upsertDuplicates(dups)           → void
├── getCoverage(appId?)              → CoverageReport[]
├── upsertCoverage(reports)          → void
├── getHealth(testId?)               → HealthRecord[]
├── upsertHealth(records)            → void
├── getGaps()                        → GapSuggestion[]
├── upsertGaps(gaps)                 → void
├── createScan()                     → ScanJob
├── getScan(id)                      → ScanJob
└── updateScan(id, status)           → void
```

**Phase 1 implementation:** `MockStore` — backed by `Map` objects, populated with hardcoded data matching the sample monorepo fixture (3 apps, ~15 features, ~40 tests, duplicate clusters, coverage gaps).

---

## 5. API Endpoints

All routes are thin handlers: parse request → call one store method → return JSON.

| Route | Method | Store Call |
|-------|--------|-----------|
| /api/v1/apps | GET | getApplications() |
| /api/v1/features | GET | getFeatures(appId) |
| /api/v1/inventory | GET | getTests(filters, pagination) |
| /api/v1/coverage | GET | getCoverage(appId) |
| /api/v1/duplicates | GET | getDuplicates() |
| /api/v1/gaps | GET | getGaps() |
| /api/v1/health | GET | getHealth() |
| /api/v1/graph | GET | getRelationships() |
| /api/v1/trends | GET | getCoverage() grouped by date |
| /api/v1/scan | POST | createScan() — returns job as immediately complete |
| /api/v1/scan/:id | GET | getScan(id) |

All GET routes support: filtering by `appId`, `featureId`, `status`; pagination via `offset`/`limit`; CORS enabled for `localhost:3000`.

Server creation: `createServer(store: IStore) → FastifyInstance`

---

## 6. Sample Monorepo Fixture

Realistic test data for development. Located at `fixtures/sample-monorepo/`.

| Content | Purpose |
|---------|---------|
| 3 apps (checkout, catalog, account) | Multi-app discovery |
| ~12 Playwright spec files | Test extraction targets |
| ~6 page objects | Dependency graph sources |
| ~8 shared components | Coverage denominator |
| 2-3 intentional duplicate test pairs | Duplicate detection validation |
| 1 app with low test density | Gap detection validation |

---

## 7. CLI

Single command for Phase 1:

```
automation-agent serve          # Starts Fastify on port 4000
```

Built with `commander`. Entry point: `src/cli/index.ts`.

---

## 8. Technology Stack (Phase 1)

| Concern | Technology |
|---------|-----------|
| Language | TypeScript on Node.js |
| API | Fastify |
| CLI | commander |
| Tests | vitest |
| Lint | eslint |
| API spec | OpenAPI 3.1 |

---

## 9. Testing (Phase 1)

| What | How |
|------|-----|
| Models | TypeScript compiler (type checking) |
| MockStore | Unit tests — verify CRUD on in-memory data |
| API routes | Integration tests — `fastify.inject()` with MockStore, assert status + response shape |

---

## 10. Verification Checklist

- [ ] `npm run build` succeeds
- [ ] `openapi.yaml` passes `swagger-cli validate`
- [ ] `automation-agent serve` starts Fastify on port 4000
- [ ] All GET endpoints return correctly shaped JSON
- [ ] Filters (`?appId=`, `?status=`) narrow results
- [ ] Pagination (`?offset=0&limit=10`) works
- [ ] `POST /scan` returns a completed scan job
- [ ] Mock data is internally consistent (coverage numbers match test counts)
- [ ] All unit/integration tests pass

---

## 11. What Comes Next

| Phase | Scope |
|-------|-------|
| Phase 2 | React dashboard consuming this API |
| Phase 3 | SQLite persistence replacing MockStore |
| Phase 4 | Real analysis engines (AST parsing, graph, coverage) |
| Phase 5 | Runtime validation + AI layer |
| Phase 6 | CI integration + hardening |

---

# END
