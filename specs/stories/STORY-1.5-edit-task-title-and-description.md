# Story 1.5: Edit Task Title & Description

## User Story
As Dev Dana, I want to edit a task's title and description so that I can refine work items as my understanding of them changes.

## Acceptance Criteria
- [ ] Clicking a card (or an explicit edit affordance) opens an editor (inline or modal) pre-filled with the current title and description.
- [ ] Saving updates the task's `title` and/or `description` in the store and immediately reflects on the card.
- [ ] Same validation as create: title 1–100 chars required, description 0–500 chars; Save disabled on invalid input.
- [ ] `Esc` or Cancel closes the editor without saving changes.
- [ ] The task's `id`, `column`, `order`, and `createdAt` are unchanged after edit.

## Technical Notes
- Reuse `updateTask` action from Story 1.2.
- Consider one shared `TaskForm` component for create + edit to reduce duplication.

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
