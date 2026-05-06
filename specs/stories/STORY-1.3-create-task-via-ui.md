# Story 1.3: Create Task via UI

## User Story
As Dev Dana, I want to add a new task to the **To Do** column from the UI so that I can capture work as soon as it comes to mind.

## Acceptance Criteria
- [ ] An "Add task" button in the **To Do** column header opens an inline input or compact form.
- [ ] The form requires a `title` (1–100 chars) and accepts an optional `description` (0–500 chars); Save is disabled when title is empty or > 100 chars.
- [ ] Submitting the form creates a task at the top of **To Do** and clears the input within < 200 ms.
- [ ] The column count badge increments by 1 and the new card is visible without a page reload.
- [ ] `Esc` or a Cancel button closes the form without creating a task.

## Technical Notes
- Reuse `addTask` action from Story 1.2.
- Show inline validation message for over-limit titles; trim whitespace on submit.

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
