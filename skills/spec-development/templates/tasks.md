# SPEC-{NNN} — Tasks

> **How to use this template**
>
> - Fill the header, then add one **Task block** per task (use the template under "Task block" below).
> - Each task must be self-sufficient — an agent executing it should not need to read sibling tasks. When a cross-task dependency is unavoidable, name it explicitly in **References** and quote the minimum needed (interface shape, file path, etc.).
> - Task IDs are `T-NNN`, zero-padded, sequential within the spec.
> - If this spec has more than ~10 tasks, move each task into `tasks/T-NNN-slug.md` and leave only the **Task index** table in this file. Both structures are supported.
> - Delete these instructions when the file is filled in.

---

## Header

**Parent epic:** [SPEC-{NNN}: {title}](./epic.md)
**Status:** `not-started` / `in-progress` / `completed`
**Created:** YYYY-MM-DD
**Last updated:** YYYY-MM-DD

### Task index

| ID | Title | Type | Size | Status | Languages | Blocked by | Blocks |
|---|---|---|---|---|---|---|---|
| T-001 | {title} | feature | M | not-started | TS | — | T-002 |
| T-002 | {title} | feature | S | not-started | TS, Go | T-001 | — |

Legend: `type` ∈ { feature, refactor, test, research, bugfix, infra, docs }. `size` ∈ { S, M, L }.

---

## Task block (copy for each task)

### T-NNN: {Short task title}

#### Meta

| Field | Value |
|---|---|
| Type | feature / refactor / test / research / bugfix / infra / docs |
| Size | S / M / L |
| Risk | low / medium / high — **optional**. Consumed by `Review pattern: spot-check` only: `high` forces a reviewer for this task; `medium` / `low` do not (a reviewer may still be spawned because the task's `Scope paths` hit a checklist §1–§7 surface, or because `Type` is `infra`). Absent ⇒ treated as `medium`. Under `adversarial` the field changes nothing — every task is reviewed. |
| Status | not-started / in-progress / completed |
| Languages | TS / Go / Python / SQL / … |
| Scope paths | `apps/web/src/features/…`, `packages/contracts/…` |
| Blocked by | T-XXX, T-YYY |
| Blocks | T-ZZZ |
| Epic sections | §7, §10, §13 |

#### Goal

One sentence. Verifiable outcome. "Implement X so that Y".

#### Context

2–3 sentences placing the task inside the epic. Why this slice, what it unblocks. Link epic sections; do not duplicate their content.

#### Scope

What is in this task.

- ...
- ...

#### Out of scope

What is explicitly NOT in this task, to prevent scope creep or ambiguity with adjacent tasks.

- ...
- ...

#### Technical specification

Concrete technical information the agent needs to execute. No implementation code; declarations, shapes, and contracts are expected.

##### Files to create or modify

| Path | Action | Purpose |
|---|---|---|
| `apps/web/src/features/x/foo.ts` | create | ... |
| `packages/contracts/x.ts` | modify | add Z field to Y type |

##### Data structures / types

Prefer structured declarations (TypeScript, Go, SQL DDL, etc.) for shapes. These are specifications, not implementations.

```ts
// apps/web/src/features/x/types.ts
interface Example {
  id: string;        // uuid v4
  name: string;      // 1–100 chars, trimmed
  createdAt: string; // ISO-8601 UTC
  status: 'draft' | 'active' | 'archived';
}
```

For cross-language contracts (frontend ↔ backend in different languages), use OpenAPI / JSON Schema / Protobuf as the source of truth and generate language-specific types. Pick the format based on the interface style: REST → OpenAPI, event / data payloads → JSON Schema, RPC / strict contracts → Protobuf.

##### Interfaces / API contracts

Prose or structured — describe purpose, inputs, outputs, errors, auth, idempotency. No handler implementation.

```
POST /api/v1/example
  Auth:       session cookie, role = {role}
  Body:       { name: string, status: 'draft' | 'active' }
  Success:    201 → { id: string, createdAt: string }
  Errors:
              400 — validation failed (field list)
              403 — caller lacks role
              409 — name conflicts with existing
  Idempotent: no; use idempotency key header if caller needs retries
```

##### Integration points

Where this task plugs into the rest of the system. Other modules / services / events / hooks that call it or that it calls.

- ...

##### Dependencies

- **Packages to add:** `package-name@^x.y` — purpose
- **Env vars:** `FOO_API_KEY`
- **External services:** ...

#### Inputs

What existing state, data, or configuration this task consumes. Link to where each comes from.

- ...

#### Outputs

What the task produces. Files, endpoints, UI surfaces, migrations, generated artefacts.

- ...

#### Acceptance criteria

Testable conditions. Given-When-Then when the flow is behavioural; plain checklist when structural. When adversarial review is enabled (v1.0), this section + the `Risk` field are the ground truth handed to the reviewer; if this section is absent the reviewer falls back to the epic / `spec.md`.

- **AC-1** — Given {state}, when {action}, then {observable outcome}
- **AC-2** — ...
- **AC-3** — ...

#### Tests

##### Strategy

What is automated, what level it runs at, what is checked manually. Default: maximize automation; manual only where automation cannot cover (visual polish, screen-reader flow, cross-device feel).

- Unit: ...
- Integration: ...
- E2E: ... — if the feature can key on browser-supplied signals (`Accept-Language`, locale, timezone, viewport), pin them in the runner config to a representative real user AND add an adversarial variant that flips the signal.
- Accessibility: axe-core on {pages}
- Visual regression: ...

##### Tools

`Vitest`, `Playwright`, `axe-core`, `supertest`, ...

##### Core scenarios (filled up-front)

Must-pass scenarios derived from user stories and acceptance criteria. Keep to 3–5 critical flows; map each to an AC.

- **S-1** — Given {state} / When {action} / Then {outcome} — covers AC-1
- **S-2** — ...

##### Additional scenarios (filled during implementation)

Edge cases the agent discovers while implementing. The agent appends here and references the written tests.

- (empty — agent fills)

##### Manual verification (run at task end)

Only steps automation cannot reliably cover. If empty, state `none — fully automated`. For UI tasks, a browser walkthrough of the primary flow is the default — automated-green only proves mechanics, not UX.

- [ ] ...

##### Verification rigour gate

Before marking the task complete, walk through the spec's `verification-checklist.md` end-to-end (it sits in this spec's directory — `./verification-checklist.md` from an inline `tasks.md`, `../verification-checklist.md` from a split `tasks/T-NNN.md`). The agent's hand-off summary uses the §8 template from that file. MANDATORY items cannot be skipped silently — they must be ticked, or explicitly waived with a one-line reason.

#### References

Links the agent needs to execute this task. Always include relevant epic sections; list any cross-task interfaces required.

- Epic: [§{N} {section}](./epic.md#...)
- Cross-task interface: `T-003 §Data structures → Example type`
- Design: {link}
- Source data: {link}
- External docs: {link}
