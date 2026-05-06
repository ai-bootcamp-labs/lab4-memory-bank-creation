# Story 1.4: Render Task Cards in Columns

## User Story
As Dev Dana, I want each task to render as a card in its assigned column so that I can see all my work at a glance.

## Acceptance Criteria
- [ ] Each task renders as a `Card` component showing the title (always visible) and description (truncated to ~2 lines if present).
- [ ] Cards within a column are ordered by the `order` field ascending.
- [ ] Each column header shows an accurate live count matching the number of cards in that column.
- [ ] Cards have ARIA `role="listitem"` and columns have `role="list"` for accessibility.
- [ ] Rendering 200 cards across the three columns produces no console warnings and remains visually responsive on scroll.

## Technical Notes
- Use stable React `key={task.id}`.
- Memoize column lists if profiler shows unnecessary re-renders.
- Title truncation via CSS `text-overflow: ellipsis`; description via `-webkit-line-clamp: 2`.

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
