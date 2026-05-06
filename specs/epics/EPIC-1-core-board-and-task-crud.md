# Epic: Core Board & Task CRUD

## Description
Deliver the foundational Kanban board UI with three fixed columns (To Do / In Progress / Done) and full create/read/update/delete operations on tasks. This epic establishes the data model, component hierarchy, and visual layout so that a user can capture and manage tasks end-to-end via mouse and basic UI controls.

## Primary Persona
Dev Dana — solo full-stack developer who needs to capture tasks in under 5 seconds without leaving the keyboard for long.

## Success Criteria
- Three columns render with correct titles and live task counts (FR-1, FR-12).
- User can create a task with title (1–100 chars) + optional description (0–500 chars), defaulting to **To Do** (FR-2, FR-3).
- User can edit and delete any task, with a confirm/undo affordance on delete (FR-4, FR-5).
- Each task has a stable UUID and `createdAt` timestamp (FR-11).
- Maps to PRD metric **M1**: time to create a task ≤ 5 seconds / ≤ 4 keystrokes (mouse path included as fallback).
- Maps to PRD metric **M5**: bundle remains < 200 KB gzipped after this epic ships.

## Scope/Complexity
M

## Dependencies
- React 18 + Vite + TypeScript project scaffold.
- Agreed task data model (id, title, description, column, order, createdAt).

## User Stories
[Placeholder for generated stories]
