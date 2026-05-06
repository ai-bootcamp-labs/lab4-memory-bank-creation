# Development Process — Dev Dana (Personal Task Board)

> Source specs: [`PRD-personal-task-board.md`](../../specs/prds/PRD-personal-task-board.md), [`agents.md`](../../specs/agents.md), epics under [`specs/epics/`](../../specs/epics/), stories under [`specs/stories/`](../../specs/stories/).
>
> Tailored for a **solo developer** ("Dana") shipping a **frontend-only**, **localStorage-backed** static site (React 18 + Vite + TypeScript). No backend, no shared environments, no accounts. Optimized for fast iteration with safety nets that prevent silent data loss and keep the bundle under 200 KB gzipped.

---

## 1. Development Process: Idea to Production

The pipeline below is intentionally lightweight — the goal is to keep ceremony low while preserving traceability from idea to deployed code. Every change, even a one-line fix, follows the same path.

```
Idea  →  Issue  →  Story / Spec  →  Branch  →  Implement  →  Test  →  PR  →  Self-review  →  Merge  →  Deploy  →  Verify
```

### 1.1 Idea & Issue creation
- Capture **every** change idea as a GitHub Issue — even your own. The board is the system of record for *tasks Dana works on today*; GitHub Issues are the system of record for *changes to the product itself*.
- Issue title format: `[type] short imperative summary`
  - Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `perf`, `test`.
  - Example: `[feat] keyboard shortcut to move card across columns`
- Issue body must include:
  - **Why** (link to the PRD section, epic, or story it serves).
  - **Acceptance criteria** as a checklist.
  - **Out of scope** notes when the boundary is non-obvious.
- Apply labels: `epic:1-crud`, `epic:2-dnd`, `epic:3-keyboard`, `epic:4-persistence`, plus `size:S|M|L`.

### 1.2 Task breakdown (story-level)
- If the issue maps to a new behavior, write or update a **story** under [`specs/stories/`](../../specs/stories/) using [`STORY-template`](../../specs/templates/story-template.md). Naming: `STORY-{epic}.{number}-{slug}.md` per [`agents.md`](../../specs/agents.md).
- A story is "ready to implement" only when:
  - It has acceptance criteria mapped to PRD FR-IDs (e.g., FR-6, FR-8).
  - It has at least one explicit success metric link (M1–M7) where applicable.
  - It can be completed in **≤ 1 day of focused work**. If not, split it.
- Decisions that change architecture or contracts get an **ADR** under [`specs/adrs/`](../../specs/adrs/) before code is written.

### 1.3 Implement
- Branch off `main` (see §2).
- Follow the rules in [`coding-standards.md`](../conventions/coding-standards.md) — types, naming, file structure, layering.
- Domain rules (e.g., title length, column IDs) live in `shared/`; never duplicate.
- Update or add tests **in the same commit** as the code they cover.
- Update specs/memory-bank docs in the same PR if behavior or vocabulary changes.

### 1.4 Test (locally, before PR)
- Run `npm run typecheck`, `npm run lint`, `npm test`, `npm run e2e`, `npm run build`.
- Manually exercise the feature with **keyboard only** (FR-8) and verify focus indicators (WCAG AA).
- Confirm bundle size in the `vite build` report is still ≤ 200 KB gzipped (M5).

### 1.5 Pull Request → Self-review → Merge
- Open a PR against `main` (see §3).
- Read your own diff in the PR UI before requesting review (or before self-merging on solo work).
- CI must be green; checklist in §3.2 must be satisfied.

### 1.6 Deploy
- Merge to `main` triggers the production deploy workflow (see §5).
- Deploy is fully automated; nothing is shipped by hand.

### 1.7 Verify in production
- After deploy, open the production URL with **a browser profile that already contains real data** (your daily-driver profile) and confirm:
  - The app loads under 1 s (M2).
  - Existing tasks survive — the schema migration (if any) ran cleanly.
  - The new behavior works end-to-end.
- If anything is wrong, **roll back first, diagnose second** (§5.4).
- Close the originating issue with a link to the merged PR and the deployed commit SHA.

---

## 2. Branching Strategy: GitHub Flow

We use **GitHub Flow** — `main` is always deployable, and short-lived branches are merged via PR. No long-lived `develop`, `release`, or `hotfix` branches.

### 2.1 Branches

| Branch | Purpose | Lifetime |
|--------|---------|----------|
| `main` | Always-deployable production code. Every commit on `main` is automatically deployed. | Permanent |
| `feature/*` | New behavior or enhancement. | Hours to a few days |
| `bugfix/*` | Fix for a defect that exists on `main`. | Hours to a day |
| `chore/*` | Tooling, dependency bumps, refactors, doc updates with no behavior change. | Short |
| `experiment/*` | Throwaway spikes. **Never** merged to `main`. | Short, then deleted |

### 2.2 Naming
`<type>/<issue-number>-<short-slug>`

```
feature/42-keyboard-move-card
bugfix/57-undo-restores-position
chore/61-bump-vite-to-7
```

### 2.3 Rules
- **`main` is sacred.** Never push directly. Branch protection requires a PR with all CI checks green.
- Branch from latest `main`. **Rebase**, don't merge `main` back in, to keep history linear.
- Squash-merge into `main` so each PR is a single, reverable commit.
- Delete the branch on merge (auto-delete in repo settings).
- One issue → one branch → one PR. Don't bundle unrelated changes.

### 2.4 Commit messages (Conventional Commits)
Each commit on a feature branch uses Conventional Commits:

```
<type>(<scope>): <short summary>

<optional body explaining the why>

Refs: #42
```

Types mirror issue types (`feat`, `fix`, `chore`, `docs`, `refactor`, `perf`, `test`).
Example: `feat(keyboard): move selected card with arrow keys (#42)`.

The squash-merge commit on `main` becomes the canonical history entry — keep it descriptive.

---

## 3. Pull Request Process & Code Review

Even on a solo project, the PR is the place where **you become your own reviewer** and CI becomes your second pair of eyes. Treat it that way.

### 3.1 Pre-PR checklist (run locally before opening)

- [ ] Branch is rebased on the latest `main`.
- [ ] `npm run typecheck` passes (no `any`, no `@ts-ignore` without reason).
- [ ] `npm run lint` and `npm run format:check` pass.
- [ ] `npm test -- --coverage` passes; coverage thresholds met (≥ 80% overall, ≥ 95% domain — see §4).
- [ ] `npm run e2e` passes locally for any user-visible change.
- [ ] `npm run build` succeeds; bundle ≤ 200 KB gzipped (M5).
- [ ] Manually verified the change with keyboard alone (FR-8) and with a screen reader for newly added interactive elements.
- [ ] No `TODO`/`FIXME` without an owner and issue link.
- [ ] No `console.log` left behind; only `console.warn` / `console.error` for the documented FR-10 fallback path.
- [ ] Specs (`specs/`) and memory bank (`memory-banks/`) updated when behavior, contracts, or conventions changed.

### 3.2 PR description template

```markdown
## What
One-paragraph summary of the change.

## Why
Closes #<issue>. Links to PRD section / epic / story.

## How
- Bullet list of the key implementation choices.
- Mention any deviations from the spec and why.

## Acceptance criteria
- [ ] AC1 from the linked story
- [ ] AC2 ...

## Verification
- Tests added/updated: <files>
- Manual checks: keyboard path, focus ring, refresh-survival.
- Bundle size: <NN.N KB gzipped> (delta: ±X KB)

## Risk & rollback
Describe blast radius and how to roll back.

## Screenshots / GIFs (UI changes only)
```

### 3.3 Required CI checks (branch protection)
A PR cannot merge unless **all** of the following are green:

1. Typecheck (`tsc --noEmit`).
2. Lint (`eslint`) and format (`prettier --check`).
3. Unit + component tests with coverage gates.
4. Build with bundle-size budget check.
5. Playwright E2E suite.
6. (Optional) Lighthouse CI asserting TTI < 1,000 ms (M2).

### 3.4 Review criteria (self-review on solo work)

Read the PR diff in GitHub's UI — **not** in your editor. The change of context surfaces issues you missed. Verify:

**Correctness**
- [ ] Implements the linked story's ACs and *only* those.
- [ ] Edge cases handled: empty board, 200-task limit, corrupt `localStorage`, rapid keystrokes during animation.

**Domain & contracts**
- [ ] Three Fixed Columns rule respected (no new column types snuck in).
- [ ] Local-Only rule respected (no `fetch`, no analytics, no telemetry).
- [ ] Quick Capture path still ≤ 4 keystrokes / ≤ 5 s.

**Quality**
- [ ] Layering respected (UI → store → persistence adapter).
- [ ] No `any` / `!` / unsafe casts.
- [ ] Public exports have TSDoc.
- [ ] Tests describe behavior, cover failure paths, and aren't flaky.

**Performance & UX**
- [ ] Interaction stays under 100 ms p95 (M3).
- [ ] Keyboard path works; visible focus ring; ARIA `list` / `listitem` set.
- [ ] WCAG AA contrast preserved.

**Security**
- [ ] No PII, secrets, or third-party scripts introduced.
- [ ] All external input (typed `localStorage` payload, parsed JSON) validated at the boundary.

### 3.5 Merge policy
- **Squash and merge** only. No merge commits, no rebase-merge.
- Solo merges are allowed only after the **CI is green** and the **self-review checklist** is fully ticked.
- For non-trivial changes (any architecture, persistence, or UX-shifting work), wait at least overnight before merging — sleeping on it catches more bugs than re-reading does.

---

## 4. Testing Strategy

The test pyramid for this project is **wide on unit, focused on E2E for interaction-critical flows** (drag-and-drop, keyboard navigation, persistence/refresh).

### 4.1 What to test where

| Layer | Tool | What goes here |
|-------|------|-----------------|
| **Unit** | Vitest | Pure logic: reducers, selectors, validators (`title 1–100`, `description 0–500`), the persistence adapter (debounce, schema-version handling, corrupt-payload fallback), keyboard-shortcut dispatcher mapping. |
| **Component** | Vitest + React Testing Library | Rendering of `Board`, `Column`, `TaskCard`, `TaskEditor`, `EmptyState`; focus/selection behavior; ARIA roles; undo affordance UI. |
| **E2E** | Playwright | Full user journeys in a real browser: Quick Capture (`N` → type → `Enter`), drag a card across columns, reorder within a column, edit/delete with undo, refresh-survives-data, corrupt-`localStorage` recovery. |

### 4.2 Drag-and-drop coverage (E2E is non-negotiable here)

Drag-and-drop is the easiest place to silently break the product, and unit tests cannot exercise it meaningfully. Required Playwright scenarios:

1. **Cross-column move (mouse):** drag from `To Do` → `In Progress` → `Done`; assert column counts, card order, and that the change persists across `page.reload()`.
2. **Within-column reorder:** drag a card up and down within a column; assert order in the DOM **and** in `localStorage`.
3. **Drag with keyboard:** select a card, press `→`, assert it moved to the next column and remained selected.
4. **Cancelled drag:** press `Esc` mid-drag; assert no state change.
5. **Drag at scale:** seed 200 tasks, drag one across columns; assert no frame drops via `page.evaluate(performance.now())` deltas (supports M3 < 100 ms p95).

E2E tests run against the production build (`vite preview` or the deployed preview URL), not the dev server.

### 4.3 Coverage targets (CI-enforced)

| Scope | Lines | Branches |
|-------|-------|----------|
| **Overall** | **≥ 80%** | **≥ 80%** |
| Domain (`store/`, `lib/`, validators, reducers, persistence adapter) | **≥ 95%** | **≥ 95%** |
| UI components | ≥ 70% (lower because RTL covers the meaningful behavior) | ≥ 70% |

Coverage is a **floor, not a goal** — a high-coverage module with no assertions is worthless. The PR review explicitly checks that tests assert observable behavior, not implementation details.

### 4.4 Test naming convention

```
describe('<unit under test>', () => {
  it('<does something> when <condition>', () => { /* ... */ });
});
```

Examples:
- `it('persists the new task to localStorage when a task is created')`
- `it('falls back to an empty board when localStorage contains corrupt JSON')` (FR-10)
- `it('moves the selected card to the next column when ArrowRight is pressed')` (FR-8)

Forbidden: `it('works')`, `it('test 1')`, `it('moveTask')`.

### 4.5 Determinism rules
- No real timers. Use Vitest fake timers for debounce tests (200 ms persist debounce).
- No real network. There **is** no network in this app — any test that hits one is a bug.
- Fixed UUIDs and ISO timestamps in fixtures.
- Each test starts from a known `localStorage` state (cleared in `beforeEach`).

### 4.6 What requires which tests

| Change | Unit | Component | E2E |
|--------|------|-----------|-----|
| New pure helper / reducer branch | ✅ | — | — |
| New React component | ✅ (logic) | ✅ | If user-visible |
| New keyboard shortcut | ✅ (dispatcher) | ✅ (focus) | ✅ |
| Drag-and-drop change | ✅ (reorder logic) | — | ✅ **required** |
| Persistence-format change | ✅ | — | ✅ (refresh-survival) |
| Pure refactor with no behavior change | Existing tests must still pass | — | — |

---

## 5. Deployment Process

The whole product is a static bundle (HTML + hashed JS/CSS) — there is no server to deploy. Deploy is a file upload, fully automated by CI.

### 5.1 Target host
**GitHub Pages** (default) via the official `actions/deploy-pages` action. **Vercel** is an acceptable alternative — pick one, record it in an ADR, and stick with it.

Either way:
- HTTPS is mandatory.
- `index.html` served with `Cache-Control: no-cache`.
- Hashed JS/CSS assets served with `Cache-Control: public, max-age=31536000, immutable`.
- A custom domain (optional) configured via DNS + the host's domain settings.

### 5.2 Environments

| Environment | Trigger | URL pattern | Purpose |
|-------------|---------|-------------|---------|
| **Local dev** | `npm run dev` | `http://localhost:5173` | Day-to-day work. |
| **PR preview** | Every PR | `https://<host>/preview/<pr-number>/` (or Vercel/Netlify per-PR URL) | Reviewer dogfooding. Each preview has its own URL → its own `localStorage` namespace, so previews can't corrupt your real data. |
| **Production** | Push to `main` | `https://<your-pages-domain>/` | Live app for Dana. |

There is no shared staging tier because there is no shared data — every browser is its own isolated environment.

### 5.3 GitHub Actions pipeline (recommended layout)

Two workflows under `.github/workflows/`:

**`ci.yml` — runs on every push and PR**
1. Checkout, cache `~/.npm`, `npm ci`.
2. `npm run typecheck`
3. `npm run lint && npm run format:check`
4. `npm test -- --coverage` (uploads coverage report; fails if thresholds unmet).
5. `npm run build` (uploads bundle report; fails if > 200 KB gzipped).
6. `npm run e2e` (Playwright in headless Chromium, Firefox, WebKit).
7. (Optional) Lighthouse CI checks TTI budget.
8. On PR: deploy `dist/` to a preview URL.

**`deploy.yml` — runs only on push to `main`**
1. Re-run the full `ci.yml` job set (no shortcuts on green-on-PR — reproducibility matters).
2. **Build once more** with production env (`npm run build`).
3. Upload `dist/` as the Pages artifact (or push to Vercel).
4. **Deploy** via `actions/deploy-pages@v4` (or Vercel deploy step).
5. **Smoke check:** `curl -fsSL https://<prod-url>/ > /dev/null` and a Playwright `@smoke` tag run against production for the Quick Capture flow.
6. Post a deploy summary comment on the merged PR with the commit SHA, bundle size, and deployed URL.

### 5.4 Rollback procedure

Because there is no DB and no server, rollback is just **redeploying the previous good commit's static bundle**. Two options, in order of preference:

**Option A — Revert (preferred, keeps history honest)**
```
git revert <bad-commit-sha>
git push origin main
```
The `deploy.yml` workflow runs on the new revert commit and ships the previous behavior. ETA: a few minutes.

**Option B — Re-run a previous successful deploy (fastest, ~30 s)**
1. Open the GitHub Actions tab → `deploy.yml`.
2. Locate the last green run on `main` before the regression.
3. Click **Re-run all jobs**. The artifact is rebuilt from that commit and redeployed.
4. Immediately follow up with Option A so `main` reflects the live state.

**After any rollback:**
1. Reopen the issue you just rolled back, label it `incident`, and add a short post-mortem note: what broke, how it was caught, what the fix will be, and what test was missing that let it through.
2. **Add the missing test first**, then write the fix, on a fresh `bugfix/*` branch.
3. Verify the new test fails on the bad commit and passes on the fix.

### 5.5 Data-safety notes (specific to this project)

Because user data lives in the user's own `localStorage`, deploys cannot corrupt user data — **except** when the persistence schema changes. Therefore:

- Any change to the stored schema **must** bump the `version` field and ship a forward-only migration in the persistence adapter.
- Migrations must be idempotent and survive partial application (e.g., a tab closed mid-migration on next load).
- A Playwright test must seed the **previous** schema version into `localStorage` and confirm the app upgrades it without data loss before that schema change is allowed to merge.
- Never remove old migration code on a whim — users may still be on an old version after months without opening the app.

---

## Cross-references
- Product intent & metrics: [PRD-personal-task-board.md](../../specs/prds/PRD-personal-task-board.md)
- Stack & authoring conventions: [agents.md](../../specs/agents.md)
- Architecture: [overview.md](../architecture/overview.md)
- Coding rules referenced from §3.1, §3.4, §4: [coding-standards.md](../conventions/coding-standards.md)
- Domain vocabulary used throughout: [glossary.md](../domain/glossary.md)
