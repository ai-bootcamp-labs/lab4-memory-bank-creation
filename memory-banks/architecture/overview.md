# Architecture Overview — Dev Dana (Personal Task Board)

> Companion to [`PRD-personal-task-board.md`](../../specs/prds/PRD-personal-task-board.md) and [`agents.md`](../../specs/agents.md). Read those for product intent and authoring rules; this document captures **how the system is built and shipped**.

> **Note on stack reconciliation:** The MVP PRD describes a client-only app backed by `localStorage`. The  [`agents.md`](../../specs/agents.md) now mandates a **Node.js backend + PostgreSQL**, which expands the system beyond the original MVP. This overview follows `agents.md` (the authoritative tech-stack source) and treats `localStorage` as a client-side cache / offline buffer rather than the system of record. Any conflicts should be resolved via an ADR before implementation.

## Context

Dev Dana is a Kanban board for a single solo developer ("Dana") who needs to capture, move, and complete tasks faster than Jira/Trello allow. Driving constraints from the PRD:

- Open in **< 1 second** (TTI) and respond to interactions in **< 100 ms**.
- Survive browser refreshes; data must be durable.
- Keyboard-first; WCAG 2.1 AA.
- Frontend bundle **< 200 KB gzipped**.

These constraints shape every decision below.

---

## 1. System Architecture

### Pattern: **Client–Server monolith (3-tier)**
A single-page React app talks to a single Node.js backend service, which persists data in PostgreSQL. Classic 3-tier architecture (presentation / application / data) — **not** microservices, **not** serverless, **not** client-only. One deployable backend, one database, one frontend bundle.

### Top-level component diagram

```
┌──────────── Browser (Dana) ─────────────┐      ┌────────── Server ──────────┐
│                                          │      │                            │
│  React 18 SPA (Vite-built)               │      │  Node.js API Service       │
│  ┌─────────────┐  ┌──────────────────┐   │ HTTPS│  ┌─────────────────────┐   │
│  │  UI Layer   │→ │  State / Store   │←──┼─────►│  │ HTTP Routes / Ctrl  │   │      ┌──────────────┐
│  │ (Components)│  │  (BoardStore)    │   │ JSON │  ├─────────────────────┤   │      │              │
│  └─────────────┘  └────────┬─────────┘   │      │  │ Domain / Services   │◄──┼─────►│ PostgreSQL   │
│         ▲                  │             │      │  ├─────────────────────┤   │ SQL  │  (tasks)     │
│  ┌──────┴────────┐  ┌──────▼─────────┐   │      │  │ Repository / DAL    │   │      │              │
│  │ Keyboard Hub  │  │ API Client     │   │      │  └─────────────────────┘   │      └──────────────┘
│  └───────────────┘  │ + Local Cache  │   │      └────────────────────────────┘
│                     │ (localStorage) │   │
│                     └────────────────┘   │
└──────────────────────────────────────────┘
```

### Key components

#### Frontend (browser)
| Component | Responsibility |
|-----------|----------------|
| **UI Layer** | React components: `Board`, `Column`, `TaskCard`, `TaskEditor`, `EmptyState`. Pure presentation; reads from store, dispatches actions. |
| **BoardStore (state)** | In-memory source of truth for the rendered board. Exposes actions (`createTask`, `moveTask`, `editTask`, `deleteTask`, `reorder`). Implementation TBD via ADR (Zustand or Context + reducer). |
| **API Client** | Typed wrapper over `fetch` for the backend REST endpoints; handles retries, optimistic updates, and error surfacing. |
| **Local Cache (`localStorage`)** | Client-side cache of the last-known board state for instant first paint and offline-tolerant reads. **Not** the system of record — server is authoritative. |
| **Keyboard / Shortcut Hub** | Global key handler mapping `N`, `←`, `→`, `↑`, `↓`, `Enter`, `Delete`/`Backspace`, `Esc` to store actions (FR-8). |
| **DnD Controller** | Drag-and-drop between and within columns (FR-6, FR-7), keyboard-accessible. |

#### Backend (Node.js)
| Component | Responsibility |
|-----------|----------------|
| **HTTP Routes / Controllers** | REST endpoints (e.g., `GET /api/board`, `POST /api/tasks`, `PATCH /api/tasks/:id`, `DELETE /api/tasks/:id`, `POST /api/tasks/:id/move`). Input validation at the boundary. |
| **Domain / Services** | Business rules: title length (1–100), description length (0–500), column membership, ordering invariants (FR-1 through FR-12). |
| **Repository / DAL** | Data access layer over PostgreSQL using a typed query builder or ORM. Encapsulates SQL behind intent-revealing methods like `findBoard()`, `moveTask(id, toColumn, toIndex)`. |

#### Data tier
| Component | Responsibility |
|-----------|----------------|
| **PostgreSQL** | Durable system of record. Stores tasks, columns, ordering, and timestamps. Schema-versioned via migrations. |

### Data model (canonical, server-side)

```ts
type ColumnId = 'todo' | 'in-progress' | 'done';

interface Task {
  id: string;          // UUID v4 (FR-11)
  title: string;       // 1–100 chars (FR-2)
  description: string; // 0–500 chars (FR-2)
  columnId: ColumnId;
  position: number;    // ordering within column (FR-7)
  createdAt: string;   // ISO 8601 (FR-11)
  updatedAt: string;   // ISO 8601
}
```

Indicative PostgreSQL schema:

```sql
CREATE TABLE tasks (
  id           UUID PRIMARY KEY,
  title        VARCHAR(100) NOT NULL,
  description  VARCHAR(500) NOT NULL DEFAULT '',
  column_id    TEXT NOT NULL CHECK (column_id IN ('todo','in-progress','done')),
  position     INTEGER NOT NULL,
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX tasks_column_position_idx ON tasks (column_id, position);
```

### Data flow (write path)
1. User action (click, drag, key) → component handler.
2. Handler calls a store action (e.g., `moveTask(id, toColumn, toIndex)`).
3. Store applies an **optimistic** update and notifies subscribers (UI re-renders immediately to keep the < 100 ms interaction budget).
4. API Client sends the change to the Node.js backend (`PATCH`/`POST`).
5. Backend validates, persists to PostgreSQL, returns the canonical record.
6. Store reconciles the server response; on error, the optimistic change is rolled back and surfaced to the user.
7. Local cache (`localStorage`) is updated to reflect the latest confirmed state.

### Data flow (read path / boot)
1. App mounts → store hydrates **synchronously** from `localStorage` for instant first paint (supports M2 — TTI < 1 s).
2. App mounts → API Client fetches `GET /api/board` in parallel.
3. On response, store reconciles server data over the cached snapshot; UI updates if anything changed.
4. On corrupt cache or fetch failure, fall back gracefully (FR-10): show last-known cache, or empty board with a warning banner — never crash.

---

## 2. Tech Stack

Authoritative source: [`agents.md`](../../specs/agents.md).

| Layer | Choice | Rationale |
|-------|--------|-----------|
| **Frontend framework** | **React 18** | Mandated by `agents.md`. Mature, concurrent rendering. |
| **Frontend build tool** | **Vite** | Mandated. Fast dev server and small production bundles support TTI (M2) and the 200 KB budget (M5). |
| **Language (frontend & backend)** | **TypeScript** (strict mode) | Mandated. Single language across tiers; shared types possible. |
| **Backend runtime** | **Node.js** (LTS) | Mandated by `agents.md`. Same language as frontend, large ecosystem. |
| **Backend framework** | TBD via ADR — likely Express or Fastify | Lightweight HTTP layer; Fastify preferred for performance and built-in schema validation. |
| **Database** | **PostgreSQL** | Mandated by `agents.md`. Relational integrity for ordered task lists; strong indexing and transactions. |
| **DB access** | TBD via ADR — Drizzle, Kysely, or Prisma | Typed query layer; whichever pick keeps the backend slim with explicit migrations. |
| **State management (frontend)** | Lightweight (Zustand or Context + `useReducer`) | Avoids Redux overhead; keeps bundle small. ADR pending. |
| **Drag & drop** | `@dnd-kit/core` (candidate) | Accessible, keyboard-aware, tree-shakeable. ADR pending. |
| **Styling** | CSS Modules / vanilla CSS + tokens | No runtime CSS-in-JS to keep bundle/TTI low; supports WCAG AA via tokens. |
| **IDs** | `crypto.randomUUID()` (browser) / `uuid` package (server) | FR-11. |
| **Unit / integration tests** | **Vitest** + React Testing Library (frontend); **Vitest** or **Jest** (backend) | Fast, native to Vite. |
| **E2E tests** | **Playwright** | Cross-browser matrix from PRD §5. |
| **Linting / formatting** | ESLint + Prettier + `tsc --noEmit` | Enforced in CI; details in [coding-standards.md](../conventions/coding-standards.md). |
| **Package manager** | npm | Lockfile committed. |

### Persistence note
`localStorage` is **not** the system of record under this stack. It remains in the architecture as a **client-side cache** that:
- Provides instant first paint to meet TTI targets.
- Buffers writes when offline (best-effort), flushed to the API when connectivity returns.

The durable record lives in PostgreSQL.

### Bundle budget (frontend)
- **Hard ceiling:** 200 KB gzipped (M5), enforced via the `vite build` report on every release.

---

## 3. Deployment

### Environments

| Environment | Purpose | Frontend | Backend | Database |
|-------------|---------|----------|---------|----------|
| **Local dev** | Day-to-day development | `vite` dev server (`localhost:5173`) | `node`/`tsx` watch mode (`localhost:3000`) | Local PostgreSQL via Docker Compose |
| **Preview / staging** | Per-PR validation, dogfooding | Static host preview build | Ephemeral container per PR | Isolated staging Postgres (or per-PR ephemeral DB) |
| **Production** | Live app for Dana | Static host (CDN-fronted) | Containerized Node.js service (e.g., Azure Container Apps / App Service) | Managed PostgreSQL (e.g., Azure Database for PostgreSQL Flexible Server) |

### Deployment strategy

- **Frontend** → static asset deploy. `vite build` outputs hashed JS/CSS + `index.html` to `dist/`, uploaded to a CDN-fronted static host. Long-lived cache on hashed assets, no-cache on `index.html`. Atomic from each user's perspective on next page load.
- **Backend** → containerized service, **rolling deploy** with health checks. Two replicas minimum for zero-downtime updates. Configuration via environment variables (DB connection string, CORS origin, log level) — no secrets in the image.
- **Database** → managed PostgreSQL with automatic backups and point-in-time restore. Schema changes shipped as **forward-only migrations** run as a pre-deploy step; migrations must be backward-compatible with the previous backend version to allow safe rolling deploys.

### CI/CD pipeline (GitHub Actions, recommended)

On every pull request:
1. **Install** dependencies (`npm ci`) for both frontend and backend workspaces.
2. **Lint** (`eslint`) and **type-check** (`tsc --noEmit`) across the repo.
3. **Unit / integration tests**:
   - Frontend: `vitest run` (covers durability via M4).
   - Backend: `vitest run` against an ephemeral PostgreSQL spun up in the workflow.
4. **Build**:
   - Frontend: `vite build` — fails if bundle > 200 KB gzipped (M5).
   - Backend: build a runnable artifact / Docker image.
5. **E2E tests** (Playwright) against the built frontend + backend wired to a disposable Postgres.
6. **Lighthouse CI** (optional) to assert TTI budget (M2).
7. **Preview deploy** — frontend to preview URL, backend to ephemeral container, DB migrations applied to a per-PR schema.

On merge to `main`:
1. Re-run the full pipeline.
2. **Apply DB migrations** to production Postgres (forward-only, backward-compatible).
3. **Rolling deploy** of the backend container.
4. **Deploy `dist/`** to the production static host.
5. Tag a release; attach the bundle report and migration log as release artifacts.

### Rollback
- **Frontend:** redeploy the previous Git tag's static artifact.
- **Backend:** redeploy the previous container image; rolling deploy back to the prior version.
- **Database:** roll forward only. Because migrations are backward-compatible, the previous backend version can run against the new schema while a fix is prepared. Use managed Postgres point-in-time restore only as a last resort.

### Operational concerns
- **Monitoring:** backend logs and basic health metrics (request rate, error rate, latency p95) — aligned with M3's interaction-latency target.
- **Logging:** structured JSON logs from the backend; `console.warn` only on the frontend (no telemetry transmitted from the browser per PRD §5).
- **Secrets:** DB credentials and any future API keys stored in the host's secret manager (e.g., Azure Key Vault). Never committed.
- **Cost:** dominated by the managed Postgres tier; backend container and static hosting are negligible at single-user scale.

---

## Cross-references
- Product intent & metrics: [PRD-personal-task-board.md](../../specs/prds/PRD-personal-task-board.md)
- Authoritative tech stack & authoring conventions: [agents.md](../../specs/agents.md)
- Persistence decision (under review given the new backend): [ADR-use-localstorage-for-persistence.md](../../specs/adrs/ADR-use-localstorage-for-persistence.md)
- Coding standards: [coding-standards.md](../conventions/coding-standards.md)
- Development workflow: [development-process.md](../workflows/development-process.md)
