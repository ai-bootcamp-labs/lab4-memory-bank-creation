# Epic: Drag-and-Drop Interactions

## Description
Enable smooth mouse/trackpad drag-and-drop for moving tasks between columns and reordering tasks within a column. This epic layers interaction polish on top of the core CRUD board so the user can reorganize work visually with sub-100 ms responsiveness.

## Primary Persona
Dev Dana — needs to reprioritize and progress tasks across 2–3 projects without multi-click menus.

## Success Criteria
- User can drag a card from any column to any other column and the change persists (FR-6).
- User can reorder cards within the same column via drag (FR-7).
- Drop targets show clear visual feedback (placeholder/highlight) during drag.
- Interaction latency < 100 ms p95 measured via Chrome DevTools (PRD metric **M3**).
- Smooth performance with up to 200 tasks (no frame drops on drag).
- Maps to PRD metric **M3**: drag/keyboard interaction latency < 100 ms p95.

## Scope/Complexity
M

## Dependencies
- EPIC-1 (Core Board & Task CRUD) must be complete — board, columns, and task model exist.

## User Stories
[Placeholder for generated stories]
