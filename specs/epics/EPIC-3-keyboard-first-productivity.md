# Epic: Keyboard-First Productivity

## Description
Provide a complete keyboard interaction layer so the user can create, navigate, move, edit, and delete tasks without touching the mouse. This epic delivers the "feels faster than Jira" promise by reducing task capture and movement to a few keystrokes and ensuring full keyboard accessibility.

## Primary Persona
Dev Dana — keyboard-driven developer who values shortcuts and dislikes context-switching to a pointer device.

## Success Criteria
- `N` opens a new-task input; `Enter` saves; `Esc` cancels (FR-8).
- `↑`/`↓` navigate cards within a column; `←`/`→` move the selected card across columns (FR-8).
- `Enter` edits the selected card; `Delete`/`Backspace` removes it (FR-8).
- Visible focus indicator on the selected card at all times.
- 100% of board actions reachable via keyboard alone; ARIA roles set on columns (`list`) and cards (`listitem`) — meets WCAG 2.1 AA.
- Task creation achievable in ≤ 4 keystrokes / ≤ 5 seconds (PRD metric **M1**).
- Maps to PRD metric **M1** (capture speed) and **M3** (interaction latency < 100 ms p95).

## Scope/Complexity
M

## Dependencies
- EPIC-1 (CRUD operations and task model exist).
- EPIC-2 not strictly required, but column-move shortcut behavior should align with drag semantics.

## User Stories
[Placeholder for generated stories]
