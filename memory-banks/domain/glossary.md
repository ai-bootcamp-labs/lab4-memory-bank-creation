# Domain Glossary: Personal Task Board (Dev Dana)

> Source specs: [`PRD-personal-task-board.md`](../../specs/prds/PRD-personal-task-board.md), [`EPIC-1`](../../specs/epics/EPIC-1-core-board-and-task-crud.md), [`EPIC-2`](../../specs/epics/EPIC-2-drag-and-drop-interactions.md), [`EPIC-3`](../../specs/epics/EPIC-3-keyboard-first-productivity.md), [`EPIC-4`](../../specs/epics/EPIC-4-local-persistence-and-resilience.md).
>
> This glossary defines vocabulary used in code, UI copy, specs, and conversations about the project. Use these exact terms to avoid ambiguity.

## Board

**Definition:** The single, application-wide Kanban surface that contains all of the user's tasks across exactly three fixed columns. There is one board per user — never multiple boards or per-project boards in the MVP.
**Context:** "The board" is the entire app from the user's perspective. When code or specs refer to *board state*, they mean the complete set of tasks plus their column assignment and ordering. The board is **not** a noun that gets created, listed, or chosen — it simply exists.
**Example:** "On load, the board is restored from `localStorage`." → restore the entire app's task data, not "open a board".

---

## Column

**Definition:** One of exactly three fixed lanes on the board: **To Do**, **In Progress**, **Done**. Each column holds an ordered list of tasks. Column identity is represented in code by the type `ColumnId = 'todo' | 'in-progress' | 'done'`.
**Context:** Columns are **not** user-configurable — they cannot be renamed, reordered, added, or removed. Each column header shows a live task count (FR-12). Tasks move between columns via drag-and-drop (FR-6) or `←`/`→` keyboard shortcuts (FR-8).
**Example:** "Move the selected card to the next column" means shift its `columnId` from `todo` → `in-progress` → `done` (or the reverse), never to a custom column.

---

## Task (a.k.a. Card)

**Definition:** A single unit of work tracked on the board, consisting of a required `title` (1–100 chars), an optional `description` (0–500 chars), a stable UUID `id`, a `createdAt` timestamp, a `columnId`, and a `position` within its column. **"Task" and "card" are the same entity** — "task" is the data model term; "card" is the UI representation.
**Context:** Tasks default to the **To Do** column on creation (FR-3). They are the only first-class entity in the domain — the MVP has no tags, due dates, sub-tasks, comments, or attachments. Use "task" in backend, data, and spec language; use "card" only in UI/UX language ("the card glows when selected").
**Example:** "Delete the task" (in code) and "Delete the card" (in UI copy) refer to the same operation.

---

## Dev Dana

**Definition:** The single primary user persona — a solo full-stack developer / freelancer juggling 2–3 concurrent projects who needs frictionless personal task tracking outside of heavy tools like Jira or Trello.
**Context:** Every product decision is evaluated against Dana's needs: capture in < 5 seconds, keyboard-first interaction, no sign-in, instant load, and local durability. When a feature request can't be tied back to Dana's workflow, it likely belongs in "Out of Scope" (PRD §7). Dana is also the **dogfooding user** for success metrics M6 and M7.
**Example:** Reject a feature with "this is great, but Dana doesn't need multi-board support — the PRD scopes us to a single board." Don't reject as "not asked for"; reject as "out of persona scope."

---

## Quick Capture

**Definition:** The keyboard-driven flow for creating a task with the fewest possible keystrokes: press `N` → type title → press `Enter`. The new task appears in **To Do** instantly.
**Context:** Quick Capture is the headline interaction of the product — it directly satisfies success metric **M1** (≤ 5 seconds / ≤ 4 keystrokes). It is the reason the keyboard layer exists (EPIC-3). Any change that adds steps, modal dialogs, or required fields to task creation breaks Quick Capture and must be rejected or routed through an ADR.
**Example:** Adding a "select column" step to creation would violate Quick Capture, because tasks **always** default to **To Do** (FR-3).

---

## Selected Card / Selection

**Definition:** The single task currently focused for keyboard interaction, indicated by a visible focus ring and tracked in app state as `selectedTaskId`. Only one card may be selected at any time.
**Context:** Selection is the anchor for every keyboard shortcut except `N` (new): `↑`/`↓` move selection within a column, `←`/`→` move the selected task across columns, `Enter` edits, `Delete`/`Backspace` removes (FR-8). Selection is purely a UI concept — it is **not** persisted to `localStorage` and resets on reload. Do not confuse "selected" (focus) with "highlighted during drag" (drop preview).
**Example:** "Move the task right" in spec language means: take the task identified by `selectedTaskId` and move it to the next column.

---

## Local Persistence

**Definition:** The mechanism by which all board state (tasks, columns, ordering, schema version) is written to the browser's `localStorage` so it survives page refreshes, browser restarts, and crashes — without any backend, account, or network call.
**Context:** Local Persistence is the entire durability story for the MVP (EPIC-4, FR-9, FR-10, metric **M4**). Writes are debounced ≤ 200 ms but **must** flush on `beforeunload`. On read, missing or corrupt data falls back to an empty board with a `console.warn` — never a crash. Because the data lives only in one browser on one device, "Local Persistence" implies **no sync, no backup, and no cross-device access** — those are explicitly out of scope.
**Example:** "All changes are persisted" means written to `localStorage`, not to a server. A spec saying "data is saved" without qualification refers to local persistence.

---

## Undo Affordance

**Definition:** A short-lived UI mechanism (e.g., a toast with an "Undo" action) that lets the user reverse a destructive operation — primarily task deletion (FR-5) — without a separate confirmation dialog.
**Context:** The PRD permits *either* a confirm step *or* an undo affordance for delete; the project's chosen pattern is undo, because it preserves Quick-Capture-style speed while protecting against accidental data loss. Undo is **transient** — once the affordance disappears (e.g., a few seconds later), the deletion is final and the task is gone from `localStorage`.
**Example:** Pressing `Delete` on a selected card removes it immediately and shows "Task deleted — Undo". Clicking Undo within the window restores the task with the same `id` and original `position`.

---

## Key Business Rules

### Three Fixed Columns

**Rule:** The board displays exactly three columns — **To Do**, **In Progress**, **Done** — in that left-to-right order. Columns cannot be added, removed, renamed, reordered, hidden, or made user-configurable. Newly created tasks always start in **To Do**.
**Rationale:** Dev Dana's pain point is the ceremony of heavy task trackers. Fixing the workflow at three columns eliminates configuration overhead, keeps the data model tiny (`ColumnId` is a closed union), and makes keyboard movement (`←`/`→`) deterministic. Every "let users customize columns" request is explicitly out of scope (PRD §7) and must be deferred to a post-MVP RFC.
**Example:** A request to "add a Backlog column for triage" is rejected at intake — not because Backlog is a bad idea, but because it violates the Three Fixed Columns rule. The user is asked to use **To Do** for triage, or to file an RFC to revisit the rule.

---

### Local-Only, Single-Device Operation

**Rule:** All data lives in the user's browser `localStorage`. The application performs **no network calls**, has **no accounts or authentication**, performs **no analytics or telemetry**, and provides **no cross-device or cross-browser synchronization**. The board in Browser A on Laptop A is a completely separate dataset from the board in Browser B on the same machine, or any board on a phone.
**Rationale:** Eliminating the backend is what enables the < 1 second TTI (M2), zero-cost hosting, full offline operation, and absolute privacy ("no PII transmitted" — PRD §5). It is also the reason every feature is achievable by a solo developer in a small project. Adding sync, accounts, or telemetry would invalidate the architecture, the privacy posture, and the success metrics simultaneously.
**Example:** Dana opens the app on a second laptop and sees an empty board. This is **correct behavior**, not a bug. A request to "sync between my laptop and phone" is rejected and routed to a post-MVP backend RFC; it does not become a story under the current epics.
