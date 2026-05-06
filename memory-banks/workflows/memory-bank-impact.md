# Memory Bank Impact Analysis

## Test Task
Generate the React component and logic for adding a new task card to a Kanban board.

## Prompt Used
```text
Generate the React component and logic for adding a new task card to a Kanban board.
```

---

## Results WITHOUT Memory Banks

### Generated Code
- `AddTaskCard.tsx` — generic React component with its own form state management.
- `AddTaskCard.css` — gradient backgrounds and a thicket of ad-hoc CSS classes.
- `KanbanBoard.tsx` — an unnecessarily complex file that tries to manage the entire board.
- `useKanbanBoard.ts` — naive in-memory state management.
- `package.json` and `package-lock.json` — generated as if the project did not already exist.

### Issues Found
- ❌ **Wrong architecture:** Tried to scaffold a brand-new project (its own `package.json`, etc.) inside an already-existing project.
- ❌ **Ignored design standards:** Invented CSS classes, gradient colors, shadows, and bespoke button styles, ignoring the project's CSS / UI conventions.
- ❌ **Domain rule violations:** `useKanbanBoard.ts` held the data only in memory via a single `useState`, completely ignoring the **Local Persistence** rule.
- ❌ **UX violations:** Added a pop-up modal with `dueDate`, `priority`, and other fields that do not belong to this product. No trace of the **Quick Capture** concept.
- ❌ **No tests:** Did not produce any Vitest / React Testing Library files.

### Estimated Correction Time
Cleaning this up — reducing it to a single inline input, removing the extra form fields, and wiring it to the store — would take roughly **30–40 minutes**.

---

## Results WITH Memory Banks

### Generated Code
- `AddTaskCard.module.css` — clean styling using CSS Modules.
- `AddTaskCard.tsx` — a minimal component that adds tasks **only to the To Do column** and is wired through `useBoardStore`.
- `AddTaskCard.test.tsx` — Vitest + React Testing Library tests, written with the project's coverage targets in mind.

### Improvements
- ✅ **Applied the domain glossary:** Understood the **Quick Capture** flow (press Enter to save instantly) and avoided unnecessary fields like priority and due date.
- ✅ **Correct store wiring:** Called `createTask` from `useBoardStore`, which means the change automatically respects the Local Persistence path.
- ✅ **Followed conventions:** Used CSS Modules (`.module.css`) and respected the **Three Fixed Columns** rule by hard-defaulting new tasks into `To Do`.
- ✅ **Tests included:** Wrote Vitest + RTL tests aligned with the 80% coverage policy, including coverage for keyboard interactions.

### Remaining Issues (if any)
- ⚠️ Almost none. The only thing to verify is that the `useBoardStore` import path matches the actual project folder structure.

### Estimated Correction Time
**2–3 minutes** (only minor import-path adjustments may be required).

---

## Impact Summary

- **Time saved:** Roughly **30 minutes per feature**, on average.
- **Quality improvement:** Project-specific naming, presence of tests, and **100%** elimination of out-of-scope features.
- **Key learning:** Giving the AI an explicit domain glossary plus naming and UI/UX rules constrains its "imagination" and keeps the output strictly inside the product's intended scope.

---

## Refinements Needed

Based on this test, the memory bank should be improved by:

- Extending [`coding-standards.md`](../conventions/coding-standards.md) with more specific guidance on **mocking and testing tooling** — in particular, explicit rules for **mocking the Zustand store** (when `useBoardStore` is the chosen implementation), so tests stay deterministic and consistent across components.
