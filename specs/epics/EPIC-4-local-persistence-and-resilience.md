# Epic: Local Persistence & Resilience

## Description
Persist all board state to `localStorage` so the user's tasks survive refreshes, browser restarts, and crashes — and gracefully recover when stored data is missing or corrupt. This epic delivers the "trust that data is still there tomorrow" guarantee from the PRD without introducing any backend.

## Primary Persona
Dev Dana — relies on the board across days/weeks and cannot tolerate silent data loss.

## Success Criteria
- Every state change (create, edit, delete, move, reorder) is persisted to `localStorage` (FR-9).
- Writes are debounced ≤ 200 ms but flushed before `beforeunload` so no change is lost on tab close.
- On load, board state is restored from `localStorage`; missing or corrupt data falls back to an empty board with a console warning, no crash (FR-10).
- A schema version field is stored alongside data to allow future migrations.
- Automated test (Vitest/Playwright) verifies that 100% of changes survive a simulated refresh — PRD metric **M4**.
- Maps to PRD metric **M4** (data durability 100%) and supports **M2** (TTI < 1,000 ms by keeping payloads small).

## Scope/Complexity
S

## Dependencies
- EPIC-1 (task data model exists and components dispatch state changes).
- Can be developed in parallel with EPIC-2 and EPIC-3 once the model is stable.

## User Stories
[Placeholder for generated stories]
