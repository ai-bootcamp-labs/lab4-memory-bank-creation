# Coding Standards — Dev Dana

> Authoritative tech-stack source: [`specs/agents.md`](../../specs/agents.md)
> Stack: **React 18 + Vite (frontend)**, **Node.js (backend)**, **TypeScript** everywhere, **PostgreSQL** for persistence.
>
> These standards apply to **all code in the repository** unless an ADR explicitly overrides them. When in doubt, optimize for: readability → testability → performance.

---

## 1. Naming Conventions

Consistent names make the codebase navigable for both humans and AI assistants. Pick the convention by the **kind of thing**, not by the folder it lives in.

### 1.1 Files

| Kind | Convention | Example |
|------|------------|---------|
| React component file | `PascalCase.tsx` | `TaskCard.tsx`, `BoardColumn.tsx` |
| React hook file | `camelCase.ts`, prefix `use` | `useBoardStore.ts`, `useKeyboardShortcuts.ts` |
| Plain TS module (utils, services, types) | `kebab-case.ts` | `api-client.ts`, `task-validators.ts` |
| Test file | `<source>.test.ts(x)` colocated | `TaskCard.test.tsx`, `task-validators.test.ts` |
| E2E test file | `<feature>.spec.ts` under `e2e/` | `e2e/create-task.spec.ts` |
| Backend route file | `kebab-case.routes.ts` | `tasks.routes.ts`, `board.routes.ts` |
| Backend service file | `kebab-case.service.ts` | `task.service.ts` |
| Backend repository file | `kebab-case.repository.ts` | `task.repository.ts` |
| SQL migration | `NNNN_snake_case.sql` (zero-padded) | `0001_create_tasks.sql`, `0002_add_position_index.sql` |
| Type-only file | `kebab-case.types.ts` | `task.types.ts` |
| Index/barrel | `index.ts` (only re-exports, no logic) | `components/index.ts` |

**Rule:** one default export *or* named exports — never both. Prefer **named exports** for everything except React components, where a single default export is acceptable.

### 1.2 Classes

- **PascalCase**, noun phrases, no `I` prefix on interfaces.
- One class per file unless tightly coupled.
- Keep classes small; if a class has > ~5 public methods or > ~150 lines, split it.

```ts
// Good
export class TaskRepository { /* ... */ }
export class ValidationError extends Error { /* ... */ }

// Bad
export class taskRepo { /* ... */ }       // wrong case
export class ITaskRepository { /* ... */ } // no Hungarian-style prefix
export class TaskMgr { /* ... */ }         // unclear abbreviation
```

### 1.3 Functions and Variables

- **`camelCase`** for functions, methods, locals, parameters, and non-component React props.
- Functions are **verbs or verb phrases**: `createTask`, `moveTaskToColumn`, `hydrateBoardFromCache`.
- Booleans use an auxiliary verb prefix: `is`, `has`, `can`, `should`.
- Prefer descriptive names over comments. `taskCountByColumn` beats `m` plus a comment.

```ts
// Good
const isLoading = false;
const hasUnsavedChanges = store.dirty;
function moveTaskToColumn(taskId: string, toColumn: ColumnId, toIndex: number) { /* ... */ }

// Bad
const flag = false;                // what flag?
const data = response.body;        // what data?
function process(x: unknown) {}    // process what?
```

React components are functions named in **PascalCase** because they are constructors of UI:

```tsx
export function TaskCard({ task }: { task: Task }) { /* ... */ }
```

### 1.4 Constants

- **`SCREAMING_SNAKE_CASE`** only for **true compile-time constants**: keys, magic numbers, environment-driven defaults.
- Configuration objects are **`camelCase`** with `as const`.

```ts
// Compile-time constants
export const LOCAL_STORAGE_KEY = 'devdana.board.v1';
export const MAX_TITLE_LENGTH = 100;
export const MAX_DESCRIPTION_LENGTH = 500;
export const PERSIST_DEBOUNCE_MS = 200;

// Configuration object
export const boardConfig = {
  columns: ['todo', 'in-progress', 'done'],
  maxTasksPerColumn: 200,
} as const;
```

### 1.5 Database Tables (PostgreSQL)

- Table names: **`snake_case`**, **plural** nouns. → `tasks`, `audit_events`.
- Column names: **`snake_case`**, singular. → `id`, `column_id`, `created_at`.
- Primary keys: `id` (UUID v4).
- Foreign keys: `<referenced_table_singular>_id`. → `task_id`.
- Timestamps: `created_at`, `updated_at`, `deleted_at` (soft-delete only if needed).
- Indexes: `<table>_<columns>_idx`. → `tasks_column_id_position_idx`.
- Constraints: `<table>_<column>_<kind>`. → `tasks_title_length_chk`.

```sql
CREATE TABLE tasks (
  id           UUID PRIMARY KEY,
  title        VARCHAR(100) NOT NULL,
  description  VARCHAR(500) NOT NULL DEFAULT '',
  column_id    TEXT NOT NULL CHECK (column_id IN ('todo','in-progress','done')),
  position     INTEGER NOT NULL,
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CONSTRAINT tasks_title_length_chk CHECK (char_length(title) BETWEEN 1 AND 100)
);
CREATE INDEX tasks_column_id_position_idx ON tasks (column_id, position);
```

In TypeScript, map snake_case columns to camelCase properties at the repository boundary — never leak `snake_case` into application code.

---

## 2. File Structure

A small monorepo-style layout with two workspaces (`web/` and `server/`) and shared types:

```
repo-root/
├── web/                          # React 18 + Vite frontend
│   ├── src/
│   │   ├── components/           # Presentational + container components (PascalCase.tsx)
│   │   ├── features/             # Feature folders (board/, task-editor/, shortcuts/)
│   │   ├── hooks/                # useXxx.ts
│   │   ├── store/                # BoardStore, reducers, selectors
│   │   ├── api/                  # API client, request/response types
│   │   ├── lib/                  # Pure utilities (no React, no DOM)
│   │   ├── styles/               # CSS modules + tokens
│   │   ├── types/                # *.types.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── e2e/                      # Playwright specs
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   └── package.json
│
├── server/                       # Node.js backend
│   ├── src/
│   │   ├── routes/               # *.routes.ts (HTTP layer only)
│   │   ├── controllers/          # Thin glue from HTTP → service
│   │   ├── services/             # *.service.ts (business rules)
│   │   ├── repositories/         # *.repository.ts (DB access)
│   │   ├── db/
│   │   │   ├── client.ts         # Pool/connection
│   │   │   └── migrations/       # NNNN_*.sql
│   │   ├── middleware/           # Error handler, logger, validation
│   │   ├── lib/                  # Pure utilities
│   │   ├── types/                # Shared backend types
│   │   ├── config.ts             # Env-driven config (validated at boot)
│   │   └── server.ts             # Bootstrap & start
│   └── package.json
│
├── shared/                       # Types/contracts shared across web ↔ server
│   └── src/
│       └── task.types.ts
│
├── memory-banks/
├── specs/
├── .github/workflows/            # CI pipelines
├── package.json                  # Workspace root
└── tsconfig.base.json
```

### Layering rules (must hold)

```
routes  →  controllers  →  services  →  repositories  →  db
```

- A layer may **only** depend on the layer directly beneath it.
- Repositories are the **only** code allowed to write SQL.
- Services contain business rules and are framework-agnostic (no `req`/`res`).
- Controllers/routes deal with HTTP only.
- React components never call `fetch` directly — they go through `web/src/api/`.

### Tests live next to source
`task.service.ts` and `task.service.test.ts` live in the same folder. Only Playwright E2E specs live in a separate `e2e/` tree.

---

## 3. Code Organization

### 3.1 Function length
- **Hard cap: 50 lines** per function (excluding signature, braces, blank lines).
- **Soft target: ~20 lines.** If you cross 20, look for a natural split first.
- A function does **one thing at one level of abstraction**.

```ts
// Bad: mixes validation, persistence, and notification (60+ lines)
export async function handleTaskCreation(input: unknown) { /* ... */ }

// Good: orchestrator at one level, each helper does one thing
export async function createTask(input: CreateTaskInput): Promise<Task> {
  const validated = validateCreateTaskInput(input);
  const task = await taskRepository.insert(validated);
  emitTaskCreated(task);
  return task;
}
```

### 3.2 File length
- **Hard cap: 300 lines** per file (tests excluded).
- Split by responsibility, not by line count: a file that exists only to hold a class plus its 8 helpers should split helpers into their own module.

### 3.3 DRY (don't repeat yourself), but…
- Extract on the **third** repetition, not the first ("Rule of Three"). Premature abstraction is worse than duplication.
- Shared *types* and *constants* should be deduplicated immediately.
- Domain rules (e.g., title length 1–100) live in **exactly one place** — `shared/` — and are imported by both web and server.

```ts
// shared/src/task.types.ts
export const TITLE_MIN = 1;
export const TITLE_MAX = 100;
```

### 3.4 Single Responsibility
Every module, class, and function should have **one reason to change**. If you find yourself writing "and" when describing what something does, split it.

### 3.5 Pure where possible
- Reducers, validators, formatters, selectors → **pure functions**.
- Side effects (DB, network, `localStorage`, `console`) live at module edges.
- Pure functions are trivially testable and AI-assistant-friendly.

### 3.6 Immutability
- Never mutate function arguments.
- Use `readonly` and `as const` aggressively.
- Prefer spreads / `structuredClone` over in-place edits.

### 3.7 Type safety
- `tsconfig` runs in **`strict` mode** (`strict: true`, `noUncheckedIndexedAccess: true`, `noImplicitOverride: true`).
- **Forbidden:** `any`, non-null assertion `!`, `as` casts that widen, `// @ts-ignore`. Use `// @ts-expect-error <reason>` only with a written reason.
- Validate **all external input** (HTTP body, query, env, `localStorage` JSON) with a schema validator (e.g., Zod) at the boundary; trust types only inside.

```ts
import { z } from 'zod';

export const CreateTaskInput = z.object({
  title: z.string().min(1).max(100),
  description: z.string().max(500).default(''),
});
export type CreateTaskInput = z.infer<typeof CreateTaskInput>;
```

### 3.8 Imports
1. Node/standard library
2. Third-party packages
3. `@shared/...` workspace imports
4. Relative imports (`./`, `../`)

Separated by blank lines. No deep relative paths (`../../../`) — use a path alias.

---

## 4. Comments

Code should be self-explanatory. Comments explain **why**, not **what**.

### 4.1 TSDoc / JSDoc on public API
Every **exported** function, class, type, and React component intended for reuse gets a TSDoc block describing intent, parameters, return value, and thrown errors.

```ts
/**
 * Move a task to a new column at the given index.
 *
 * Reorders the destination column to make room and shifts the source column
 * to close the gap. The operation is atomic at the database level.
 *
 * @param taskId   - UUID of the task to move.
 * @param toColumn - Destination column.
 * @param toIndex  - Zero-based position within the destination column.
 * @returns The updated task record.
 * @throws {NotFoundError} when the task does not exist.
 * @throws {ValidationError} when `toIndex` is out of range.
 */
export async function moveTask(
  taskId: string,
  toColumn: ColumnId,
  toIndex: number,
): Promise<Task> { /* ... */ }
```

### 4.2 Inline comments
Use sparingly. Prefer renaming or extracting a function over adding a comment. When you do comment, explain the **non-obvious why**:

```ts
// Good: explains intent the code can't express
// Postgres SERIALIZABLE isolation is required here so concurrent moves
// in the same column don't collapse two tasks onto the same position.
await db.transaction({ isolationLevel: 'serializable' }, async (tx) => { /* ... */ });

// Bad: restates the code
// increment counter
counter += 1;
```

### 4.3 TODO / FIXME / NOTE format
A single, greppable format. **Always** include an owner or issue reference and a short reason.

```
// TODO(@dana, #123): replace optimistic move with CRDT once multi-device sync ships
// FIXME(@dana, #145): swallows error on quota exceeded; add user-facing surface
// NOTE: Postgres < 14 cannot use this syntax; we require 14+ via package.json engines
```

- **`TODO`** — planned improvement, non-blocking.
- **`FIXME`** — known defect, must be tracked.
- **`NOTE`** — durable explanation for future readers.
- **`HACK`** — intentional shortcut; must include why and a removal trigger.

CI fails the build if any `TODO`/`FIXME` lacks an owner or issue link.

### 4.4 No commented-out code
Delete it. Git remembers.

---

## 5. Testing Requirements

### 5.1 Test types and where they live

| Type | Tool | Scope | Location |
|------|------|-------|----------|
| **Unit** | Vitest | Pure functions, reducers, validators, services with DB mocked | Colocated `*.test.ts` |
| **Component** | Vitest + React Testing Library | Single React component, mocked store/API | Colocated `*.test.tsx` |
| **Integration** | Vitest | Backend service + real PostgreSQL (Testcontainers or workflow service) | Colocated `*.test.ts` (tagged) |
| **E2E** | Playwright | Full stack: built frontend + running backend + ephemeral DB | `web/e2e/*.spec.ts` |

### 5.2 Coverage targets
- **Lines / branches: ≥ 80%** overall, enforced in CI.
- **Domain layer (`services/`, `lib/`, reducers, validators): ≥ 95%.** Business rules cannot regress silently.
- Coverage is a **floor, not a goal** — a 100% covered module with no assertions is worthless.

### 5.3 Test naming
Describe behavior, not implementation. Pattern: **`it('<does something> when <condition>')`**.

```ts
describe('moveTask', () => {
  it('places the task at the requested index when the target column has space', async () => { /* ... */ });
  it('throws NotFoundError when the task id does not exist', async () => { /* ... */ });
  it('preserves order of other tasks in both source and destination columns', async () => { /* ... */ });
});
```

Forbidden: `it('works')`, `it('test 1')`, `it('moveTask')`.

### 5.4 Structure: Arrange–Act–Assert

```ts
it('places the task at the requested index when the target column has space', async () => {
  // Arrange
  const task = await seed.task({ columnId: 'todo', position: 0 });

  // Act
  const moved = await taskService.moveTask(task.id, 'in-progress', 0);

  // Assert
  expect(moved.columnId).toBe('in-progress');
  expect(moved.position).toBe(0);
});
```

### 5.5 Determinism
- No real timers, real network, or real clocks in unit/component tests.
- Use fixed UUIDs and ISO timestamps in fixtures.
- Seed PostgreSQL from explicit fixtures; never rely on test ordering.

### 5.6 Required tests for every change
- New function or branch → unit test.
- New API endpoint → integration test (happy path + 1+ error path).
- New user-visible behavior → at least one Playwright scenario.

---

## 6. Error Handling

### 6.1 Custom error hierarchy (backend)

```ts
// server/src/lib/errors.ts
export class AppError extends Error {
  constructor(
    public readonly code: string,           // stable, machine-readable
    message: string,
    public readonly status: number,         // HTTP status
    public readonly details?: unknown,
  ) {
    super(message);
    this.name = new.target.name;
  }
}

export class ValidationError extends AppError {
  constructor(message: string, details?: unknown) { super('VALIDATION_ERROR', message, 400, details); }
}
export class NotFoundError   extends AppError {
  constructor(resource: string)              { super('NOT_FOUND', `${resource} not found`, 404); }
}
export class ConflictError   extends AppError {
  constructor(message: string)               { super('CONFLICT', message, 409); }
}
export class InternalError   extends AppError {
  constructor(message = 'Internal error')    { super('INTERNAL_ERROR', message, 500); }
}
```

**Rules:**
- **Throw typed `AppError` subclasses** — never throw strings or plain `Error` from domain code.
- **Catch only what you can handle.** If you can't handle it, let it propagate.
- **Never swallow errors silently.** A `catch` block must log, rethrow, or transform.
- **Never use exceptions for control flow.**

### 6.2 Standard HTTP error response shape

Every error response from the API has the same JSON shape:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "title must be between 1 and 100 characters",
    "details": [
      { "path": "title", "message": "String must contain at most 100 character(s)" }
    ],
    "requestId": "f9a8b3e2-..."
  }
}
```

Centralized in a single Express/Fastify error-handling middleware so routes never format error bodies themselves:

```ts
export function errorHandler(err: unknown, req: Request, res: Response, _next: NextFunction) {
  const requestId = req.id;
  if (err instanceof AppError) {
    logger.warn({ err, requestId }, 'handled error');
    return res.status(err.status).json({
      error: { code: err.code, message: err.message, details: err.details, requestId },
    });
  }
  logger.error({ err, requestId }, 'unhandled error');
  return res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'Internal error', requestId },
  });
}
```

### 6.3 Logging requirements

- Use **`pino`** (or equivalent) — **structured JSON only**, never `console.log` in `server/`.
- Every log line includes `requestId` and `userId` (if available) for traceability.
- Log levels: `trace` < `debug` < `info` < `warn` < `error` < `fatal`.
  - `info` — request received, request completed, lifecycle events.
  - `warn` — handled errors (4xx), recoverable failures.
  - `error` — unhandled errors, 5xx, dependency failures.
- **Never log secrets, tokens, or PII.** Redact at the logger level.
- Frontend uses `console.warn`/`console.error` only — **no telemetry transmitted** (PRD §5).

```ts
logger.info({ requestId, taskId: task.id }, 'task created');
logger.warn({ requestId, err }, 'validation failed');
logger.error({ requestId, err }, 'database connection lost');
```

### 6.4 Frontend error handling

- All API calls go through the typed API client; it converts non-2xx responses to typed errors using the `error.code` field.
- Components never display raw error messages — they map error codes to user-friendly copy.
- Top-level **React Error Boundary** catches render-time crashes and shows a recoverable empty state (FR-10 spirit).

```tsx
try {
  await api.createTask(input);
} catch (err) {
  if (err instanceof ApiValidationError) {
    setFieldErrors(err.details);
    return;
  }
  notify('Could not create task. Please try again.');
  logger.error('createTask failed', err);
}
```

---

## 7. Quality Criteria

### 7.1 Definition of Done (DoD)

A change is **done** only when **every** item below is true:

- [ ] Implements the acceptance criteria from the linked story / PRD section.
- [ ] Code follows naming, structure, and organization rules in this document.
- [ ] `tsc --noEmit` passes with no errors and no new warnings.
- [ ] `eslint` and `prettier` pass with no overrides.
- [ ] Unit + component + integration tests added or updated; **all tests pass locally**.
- [ ] Coverage thresholds met (≥ 80% overall, ≥ 95% domain).
- [ ] At least one Playwright scenario exists for any new user-visible behavior.
- [ ] No `TODO`/`FIXME` without an owner and issue link.
- [ ] Public APIs (exported functions, components, endpoints) have TSDoc.
- [ ] DB changes ship as forward-only, backward-compatible migrations.
- [ ] Frontend bundle still ≤ 200 KB gzipped (M5).
- [ ] No `any`, no `!`, no `@ts-ignore`, no commented-out code.
- [ ] No secrets, credentials, or PII in source, logs, or test fixtures.
- [ ] Spec docs (`specs/`) and memory bank (`memory-banks/`) updated if behavior, contracts, or conventions changed.
- [ ] Self-reviewed: the author has read their own diff in the PR UI before requesting review.

### 7.2 Code review checklist (reviewer's lens)

**Correctness**
- [ ] Does the code do what the story asks, and only that?
- [ ] Are edge cases handled (empty list, max length, concurrent move, corrupt cache)?
- [ ] Are race conditions and N+1 queries avoided?

**Design**
- [ ] Layering respected (route → controller → service → repository → db)?
- [ ] Single Responsibility upheld?
- [ ] Names accurately describe intent?
- [ ] Is duplication justified (Rule of Three) or should it be extracted?

**Types & validation**
- [ ] All external inputs validated at the boundary?
- [ ] No `any`, `!`, or unsafe casts?
- [ ] Public types live in `shared/` if used by both tiers?

**Tests**
- [ ] Tests describe behavior, not implementation?
- [ ] Failure paths covered, not just the happy path?
- [ ] No flaky timing / network dependencies?

**Error handling**
- [ ] Errors typed as `AppError` subclasses?
- [ ] Logged once, with `requestId`?
- [ ] User-visible messages are friendly and non-leaky?

**Performance & UX**
- [ ] Interaction stays under 100 ms (M3)?
- [ ] Optimistic update + reconciliation correct on failure?
- [ ] Keyboard path works for the new behavior (FR-8)?
- [ ] WCAG AA contrast preserved?

**Security**
- [ ] No SQL string concatenation — only parameterized queries?
- [ ] No secrets in code, env logged, or PII committed?
- [ ] OWASP Top 10 considered (input validation, authz, error leakage)?

**Operability**
- [ ] Migrations forward-only and reversible by roll-forward?
- [ ] Logs sufficient to diagnose without debugging locally?
- [ ] Config read from env, validated at boot, documented in `README`?

A PR is approved only when the reviewer can answer **yes** (or **N/A with reason**) to every applicable item.
