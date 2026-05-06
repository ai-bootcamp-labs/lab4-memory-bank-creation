# Story 1.7: Empty States & Visual Polish

## User Story
As Dev Dana, I want clear empty-state messaging and consistent visual styling so that the board feels finished and guides me when columns are empty.

## Acceptance Criteria
- [ ] Each empty column shows a muted placeholder message (e.g., "No tasks yet — press **N** or click **+ Add task**").
- [ ] Cards have consistent padding, border-radius, hover state, and focus ring meeting WCAG 2.1 AA contrast (≥ 4.5:1 for text).
- [ ] Column headers, counts, and add buttons share a consistent typography scale.
- [ ] Layout remains usable down to 1024 px width with no horizontal scroll or overlapping elements.
- [ ] Lighthouse Accessibility score ≥ 95 on the populated board.

## Technical Notes
- Define design tokens (colors, spacing, radii) in a single CSS variables file.
- Verify contrast with the Chrome DevTools color picker.

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
