# Story 1.6: Delete Task with Undo

## User Story
As Dev Dana, I want to delete a task with a quick undo option so that I can clean up the board confidently without fearing accidental loss.

## Acceptance Criteria
- [ ] Each card exposes a delete affordance (icon button or menu item) reachable by mouse.
- [ ] Triggering delete removes the card from the column immediately and shows a toast/snackbar with an "Undo" action visible for at least 5 seconds.
- [ ] Clicking "Undo" restores the task to its original column and `order` position.
- [ ] After the undo window expires, the task is permanently removed from the store.
- [ ] The column count badge updates correctly on both delete and undo.

## Technical Notes
- Implement undo by keeping the deleted task in a transient buffer; commit the deletion when the toast dismisses.
- Reuse `deleteTask` and `addTask` (or a dedicated `restoreTask`) actions.

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
