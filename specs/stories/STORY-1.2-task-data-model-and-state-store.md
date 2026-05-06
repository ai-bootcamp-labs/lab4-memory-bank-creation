# Story 1.2: Task Data Model & State Store

## User Story
As Dev Dana, I want a typed task data model and central state store so that all CRUD features share a consistent shape and a single source of truth.

## Acceptance Criteria
- [ ] `Task` type defined with fields: `id: string` (UUID v4), `title: string`, `description: string`, `column: 'todo' | 'inProgress' | 'done'`, `order: number`, `createdAt: string` (ISO 8601).
- [ ] Central store (React Context + reducer, or Zustand) exposes `tasks` and actions: `addTask`, `updateTask`, `deleteTask`, `moveTask`, `reorderTask`.
- [ ] `addTask` generates a UUID and `createdAt` automatically and places the new task at the top of the **To Do** column.
- [ ] Store actions are pure and synchronously update state (no side effects in this story).
- [ ] Unit tests cover all five actions with at least one happy-path case each.

## Technical Notes
- Prefer `crypto.randomUUID()` over a library to keep bundle small.
- Keep persistence concerns out of this story (handled in EPIC-4).
- Export selectors like `getTasksByColumn(state, column)` sorted by `order`.

## Estimation
1 day

<!-- INVEST Validation Checklist:
[x] Independent
[x] Negotiable
[x] Valuable
[x] Estimable
[x] Small
[x] Testable
-->
