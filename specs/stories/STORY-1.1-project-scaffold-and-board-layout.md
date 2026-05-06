# Story 1.1: Project Scaffold & Board Layout

## User Story
As Dev Dana, I want a React + Vite + TypeScript project that renders an empty three-column Kanban board so that I have a working foundation to build task features on.

## Acceptance Criteria
- [ ] `npm create vite@latest` scaffold with React 18 + TypeScript template builds and runs via `npm run dev`.
- [ ] App renders three columns titled exactly **To Do**, **In Progress**, **Done** in left-to-right order.
- [ ] Each column header displays a task count badge showing `0` when empty.
- [ ] Layout fills the viewport at ≥ 1024 px width with equal-width columns and visible column boundaries.
- [ ] `npm run build` completes with bundle size < 200 KB gzipped (PRD M5).

## Technical Notes
- Use CSS Grid or Flexbox; no UI framework needed for MVP.
- Define `Column` and `Board` components; column titles can be a typed enum/const array.
- Add ESLint + Prettier defaults from Vite template.

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
