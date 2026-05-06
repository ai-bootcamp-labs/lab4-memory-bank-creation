# PRD: Personal Task Board

## 1. Overview
A lightweight, frontend-only Kanban task board built with React + Vite that lets a solo developer organize work across multiple projects without the overhead of tools like Jira or Trello. All data persists locally via `localStorage`, so the app loads instantly, works offline, and requires zero setup, sign-in, or backend infrastructure. The goal is to provide a fast, keyboard-friendly board that opens in under 1 second and stays out of the developer's way.

## 2. User Personas

### Primary: "Dev Dana" — Solo Full-Stack Developer
- **Role:** Independent developer / freelancer juggling 2–3 concurrent projects (e.g., a client web app, a personal side project, and an open-source contribution).
- **Pain points:**
  - Jira/Asana/Linear feel heavy: 5–10+ clicks and 3–5 seconds of load time just to move a card.
  - Loses ~10 minutes/day context-switching between project trackers.
  - Wants tasks visible at a glance, not buried behind sprints, epics, and workflows.
  - Doesn't want to manage accounts, sync issues, or pay subscriptions for personal task tracking.
- **Goals:** Capture a task in <5 seconds, move it across columns with one keystroke, and trust that data is still there tomorrow.

## 3. Use Cases

1. **Quick capture:** Dana thinks of a task, presses `N`, types the title, presses `Enter`, and the card appears in **To Do**.
2. **Start work:** Dana drags a card from **To Do** to **In Progress** (or selects it and presses `→`).
3. **Finish work:** Dana moves a completed card to **Done** via drag or keyboard shortcut.
4. **Reorder priorities:** Dana drags cards within a column to reflect priority order.
5. **Edit/delete:** Dana clicks a card to edit its title/description, or selects it and presses `Delete` to remove it.
6. **Resume next day:** Dana closes the browser, reopens it the next morning, and the board is exactly as left.
7. **Switch device/browser:** Dana accepts that the board is local-only (out of scope for MVP — see Section 7).

## 4. Functional Requirements

| ID | Requirement |
|----|-------------|
| FR-1 | Board displays exactly three columns: **To Do**, **In Progress**, **Done**. |
| FR-2 | User can create a new task with a title (required, 1–100 chars) and optional description (0–500 chars). |
| FR-3 | New tasks default to the **To Do** column. |
| FR-4 | User can edit a task's title and description inline or via a modal/panel. |
| FR-5 | User can delete a task (with a confirmation step or undo affordance). |
| FR-6 | User can move tasks between columns via drag-and-drop using a mouse/trackpad. |
| FR-7 | User can reorder tasks within a column via drag-and-drop. |
| FR-8 | Keyboard shortcuts: `N` = new task, `←`/`→` = move selected card across columns, `↑`/`↓` = navigate cards within a column, `Enter` = edit, `Delete`/`Backspace` = delete, `Esc` = cancel/close. |
| FR-9 | All board state (tasks, columns, order) is persisted to `localStorage` on every change. |
| FR-10 | On app load, board state is restored from `localStorage`; if missing/corrupt, an empty board is shown without errors. |
| FR-11 | Each task has a stable unique ID (UUID) and a `createdAt` timestamp. |
| FR-12 | Visible task count per column header. |

## 5. Non-Functional Requirements

- **Performance:**
  - Initial load (Time-to-Interactive) < 1 second on a modern laptop over local file/dev server.
  - Drag-drop and keyboard interactions respond in < 100 ms.
  - Smooth handling of up to 200 tasks total without visible lag.
- **Persistence & resilience:**
  - Writes to `localStorage` are debounced (≤ 200 ms) but guaranteed before page unload.
  - Corrupt/invalid stored data must not crash the app; fall back to empty state and log a warning.
- **Accessibility:**
  - All actions reachable via keyboard alone.
  - WCAG 2.1 AA color contrast.
  - ARIA roles for columns (`list`) and cards (`listitem`).
- **Browser support:** Latest 2 versions of Chrome, Edge, Firefox, Safari.
- **Security/Privacy:** No network calls, no analytics, no PII transmitted; all data stays in the user's browser.
- **Tech constraints:** React 18 + Vite + TypeScript; no backend; bundle size < 200 KB gzipped.

## 6. Success Metrics (SMART)

| # | Metric | Target | Measurement |
|---|--------|--------|-------------|
| M1 | Time to create a task (keystrokes from idle to saved) | ≤ 5 seconds / ≤ 4 keystrokes | Manual stopwatch test on 10 trials, by end of MVP sprint |
| M2 | Time-to-Interactive on first load | < 1,000 ms | Lighthouse, measured on a mid-tier laptop, before release |
| M3 | Drag/keyboard interaction latency | < 100 ms p95 | Chrome DevTools Performance panel, before release |
| M4 | Data durability | 100% of changes survive browser refresh in automated test suite | Vitest/Playwright test, run in CI on every PR |
| M5 | Bundle size | < 200 KB gzipped | `vite build` report, checked on every release |
| M6 | Daily active usage by primary user | ≥ 5 days/week for 4 consecutive weeks post-launch | Self-reported by Dana persona / dogfooding log |
| M7 | Tool-switching reduction | Replace Jira/Trello for personal tasks within 2 weeks of launch | Self-reported survey of 1+ pilot user |

## 7. Scope

### In Scope (MVP)
- Three fixed columns: To Do / In Progress / Done.
- Create, read, update, delete (CRUD) tasks with title + description.
- Drag-and-drop between and within columns.
- Keyboard shortcuts as defined in FR-8.
- `localStorage`-based persistence with graceful fallback on corrupt data.
- Single-user, single-board, single-device experience.
- React 18 + Vite + TypeScript implementation.
- Basic responsive layout for desktop (≥ 1024 px width).

### Out of Scope (Explicitly NOT in MVP)
- Backend, server sync, or cloud storage.
- Multi-device sync or cross-browser sync.
- Authentication, user accounts, or sharing/collaboration.
- Multiple boards or per-project boards (single board only).
- Custom columns, WIP limits, swimlanes, or workflow rules.
- Tags, labels, due dates, attachments, comments, or sub-tasks.
- Search, filtering, or full-text indexing.
- Notifications, reminders, or integrations (GitHub, Slack, calendar, etc.).
- Mobile-optimized layout (< 1024 px) and touch drag.
- Analytics, telemetry, or usage tracking.
- Import/export to Jira, Trello, CSV, or Markdown.
- Dark/light theme switcher (single default theme acceptable for MVP).
