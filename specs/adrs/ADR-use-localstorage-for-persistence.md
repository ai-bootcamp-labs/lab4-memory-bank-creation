# ADR: Use Browser `localStorage` for Task Persistence

## Status
Accepted — 2026-05-05

## Context
The Personal Task Board (see [PRD-personal-task-board.md](../prds/PRD-personal-task-board.md)) is a single-user Kanban app for "Dev Dana", a solo developer juggling 2–3 projects. The PRD imposes several constraints that directly shape the persistence decision:

- **No backend** is an explicit tech constraint (`agents.md`, PRD §5).
- **Time-to-Interactive < 1,000 ms** on first load (PRD M2).
- **100 % data durability across refreshes** (PRD M4).
- **No network calls, no analytics, no PII transmitted** — all data stays on the user's device (PRD §5 Security/Privacy).
- **Bundle size < 200 KB gzipped** (PRD M5).
- **Single-user, single-board, single-device** (PRD §7 In Scope); multi-device sync is explicitly out of scope.
- Expected working set is small: up to ~200 tasks total, each with title (≤ 100 chars) + description (≤ 500 chars) + a few metadata fields → well under 200 KB serialized.

We need a persistence layer that survives page refreshes and browser restarts, requires zero setup from the user, and does not introduce network latency, accounts, or operational cost.

## Decision
We will persist all board state (tasks, columns, ordering, schema version) as a single JSON document in the browser's **`localStorage`** under a namespaced key (e.g., `taskBoard:v1:state`).

- All store mutations write through a thin persistence adapter.
- Writes are **debounced ≤ 200 ms** and **flushed on `beforeunload`** to guarantee durability without thrashing.
- Reads happen once on app load; if the key is missing or the JSON is invalid/unparseable, we fall back to an empty board and log a warning (no crash).
- A `schemaVersion` field is stored alongside the data to enable future migrations.

## Consequences

**Positive**
- Zero backend, zero infrastructure cost, zero ops burden — fully aligned with the "no backend" constraint.
- Synchronous API → simple to integrate with React state; no async/loading UI required for persistence.
- Instant load: data is read locally in microseconds, supporting the < 1,000 ms TTI target (M2).
- Strong privacy posture: no data leaves the user's machine, no PII transmitted (PRD §5).
- No accounts, no auth, no setup — matches Dana's "open and use" expectation.
- Works fully offline.

**Negative / Trade-offs**
- **Single-device only**: data does not sync across browsers, profiles, or machines (acknowledged and explicitly out of scope in PRD §7).
- **Storage quota**: typical browser limit is ~5–10 MB per origin; comfortable for ≤ 200 tasks but would not scale to large datasets.
- **Synchronous, main-thread API**: very large writes could block the UI; mitigated by debouncing and the small payload size.
- **Per-origin / per-browser scoping**: clearing site data, using private/incognito mode, or switching browsers loses the board. We mitigate this by being explicit in the UI/docs and by keeping a versioned schema so a future export/import feature can restore data.
- **No server-side validation or backup**: a corrupt write could lose state; mitigated by `try/catch` around parse, schema-version check, and fallback to empty state with a console warning.
- **No multi-tab concurrency control**: simultaneous edits in two tabs of the same browser could overwrite each other. Acceptable for a solo single-board MVP; a `storage` event listener can be added later if needed.

## Alternatives Considered

1. **`IndexedDB`** (e.g., via `idb` or Dexie)
   - *Pros:* Larger quota (hundreds of MB), async non-blocking API, structured queries.
   - *Cons:* Async API complicates state management for a tiny dataset, adds a dependency (bundle cost vs M5), and offers no benefit at our scale (~200 tasks). **Rejected** as over-engineering for the MVP; revisit only if we exceed `localStorage` quota or need indexed queries.

2. **`sessionStorage`**
   - *Pros:* Same simple API as `localStorage`.
   - *Cons:* Cleared when the tab/window closes, violating M4 (100 % durability across refresh/restart). **Rejected**.

3. **Cookies**
   - *Pros:* Universal browser support.
   - *Cons:* 4 KB per-cookie limit, sent on every HTTP request (privacy + perf cost), not designed for app state. **Rejected**.

4. **Cloud database (Firebase / Supabase / custom REST + Postgres)**
   - *Pros:* Multi-device sync, durable backup, sharing, analytics.
   - *Cons:* Directly violates the "no backend" constraint, requires accounts/auth, introduces network latency (hurting M2), adds operational cost, and exposes user data to a third party (against PRD §5 privacy posture). Multi-device sync is explicitly out of scope (PRD §7). **Rejected** for MVP; could be revisited as a post-MVP epic.

5. **File System Access API (download/upload JSON)**
   - *Pros:* User-controlled, portable, good for backup/export.
   - *Cons:* Manual save/load is not transparent persistence; poor UX for the "open and it's there" requirement. **Rejected** as the primary store, but a strong candidate for a future export/import feature layered on top of `localStorage`.

6. **In-memory only (no persistence)**
   - *Pros:* Simplest possible implementation.
   - *Cons:* Loses all data on refresh — fails M4 outright. **Rejected**.
