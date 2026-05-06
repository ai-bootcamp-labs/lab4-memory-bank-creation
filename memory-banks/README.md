# Dev Dana — Memory Bank

This memory bank is the **canonical source of long-lived project context** for AI assistants (and humans) working on **Dev Dana**, a personal task board application. Read the relevant section before answering questions, generating code, writing specs, or making design decisions.

> **How to use this guide**
> 1. Identify the kind of task you're working on (architecture, code, domain logic, process, role-specific).
> 2. Open the matching folder below and read the linked entry points first.
> 3. Cross-reference [`../specs/`](../specs/) for product requirements (PRDs), epics, stories, and ADRs.
> 4. If information is missing or stale, prefer asking the user over guessing — then update the memory bank.

---

## Navigation

### 📐 [`architecture/`](./architecture/) — System design, tech stack, deployment
High-level structure of the application: components, data flow, technology choices, runtime environment, and deployment topology.

- [overview.md](./architecture/overview.md) — System overview and architectural decisions
- **Use when:** designing new features, choosing libraries, reasoning about performance, persistence, or deployment.

### 🧰 [`conventions/`](./conventions/) — Coding standards, testing patterns
Project-wide rules for how code is written, structured, formatted, and tested.

- [coding-standards.md](./conventions/coding-standards.md) — Style guide, naming, file layout, lint rules, testing patterns
- **Use when:** writing or reviewing code, scaffolding new modules, configuring tooling.

### 📖 [`domain/`](./domain/) — Business terms, rules, personas
Shared vocabulary and product semantics: what a "task," "column," or "board" means in Dev Dana, plus user personas and business rules.

- [glossary.md](./domain/glossary.md) — Domain terms, entities, invariants, personas
- **Use when:** naming things, modeling data, writing user-facing copy, clarifying ambiguous requirements.

### 🔄 [`workflows/`](./workflows/) — Development, review, deployment processes
How work flows through the project: branching, code review, testing gates, release process.

- [development-process.md](./workflows/development-process.md) — Day-to-day development lifecycle, review and deployment process
- **Use when:** planning a change, opening a PR, cutting a release, onboarding to the team's rhythm.

### 👥 [`roles/`](./roles/) — Role-specific context (developer, QA, PM)
Perspective-specific guidance tailored to the role the assistant is currently playing.

- `developer.md` *(planned)* — Implementation focus, code quality expectations
- `qa.md` *(planned)* — Test strategy, acceptance criteria, regression coverage
- `pm.md` *(planned)* — Product priorities, scope discipline, story shaping
- **Use when:** the user asks you to act as a specific role, or when tailoring tone, depth, and deliverables to that role.

---

## Related: [`../specs/`](../specs/)

The memory bank captures **how we work and what things mean**. The `specs/` folder captures **what we're building**:

- [`prds/`](../specs/prds/) — Product Requirements Documents
- [`epics/`](../specs/epics/) — Large bodies of work
- [`stories/`](../specs/stories/) — Implementable user stories
- [`adrs/`](../specs/adrs/) — Architecture Decision Records
- [`templates/`](../specs/templates/) — Authoring templates for the above

---

## Guidance for AI assistants

- **Prefer this memory bank over assumptions.** If a question is covered here, cite the file.
- **Keep entries short and current.** When you learn a new convention or decision, propose an update to the relevant file rather than letting it drift.
- **Match the role.** If the user invokes a role (e.g., "as QA…"), load the matching `roles/` entry first.
- **Link, don't duplicate.** When in doubt, link back to the canonical spec or ADR instead of restating it.
