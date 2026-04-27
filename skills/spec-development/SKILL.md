---
name: spec-development
description: Use this skill when the user wants to create, update, or decompose a product specification. A spec is a triplet of documents — `epic.md` (what and why), `tasks.md` (work breakdown), `plan.md` (execution order). Project-agnostic; drop this skill folder into any repo. Invoke when the user says things like "let's write a spec", "create a new spec", "break this epic into tasks", "update SPEC-NNN", or asks for help planning a feature.
user-invocable: true
---

# spec-development

Writes and maintains specifications in three shapes, depending on the size and urgency of the work. All three live under `docs/specs/` and share a single numbering sequence for regular work (`SPEC-NNN`), with hotfixes on a parallel `HF-NNN` sequence.

## Which track?

Pick the track BEFORE writing anything. Mis-pick and you either over-document a tiny change or under-document a real feature.

| Situation | Track | Artifact |
|---|---|---|
| New feature, multi-surface, partner coordination, new data model / API, cross-cutting risk | **Full triplet** | `epic.md` + `tasks.md` + `plan.md` |
| Single well-understood change, 1–3 files of impact, low risk, one-sitting execution | **Small spec** | `spec.md` (one file) |
| Production is broken, user describes the problem, fix is needed now | **Hotfix** | `hotfix.md` (one file, written alongside the fix) |

When the user invokes the skill and the intent is ambiguous, **ask which track** before creating any file. Phrasing clues:

- "write a spec for …", "new feature …", "break this down" → full triplet candidate
- "small spec for …", "just a tiny change", "one-file fix" → small-spec candidate
- "hotfix: …", "prod is broken", "срочно фикс" → hotfix

If a small-spec starts to outgrow itself during execution (new API surface, cross-cutting concern emerges), **stop and convert to a full triplet** rather than cramming. Conversely, a full-triplet epic that shrank in scope during review can be demoted to a small-spec — one `spec.md` replaces the triplet.

Templates for all three shapes live under `templates/`.

---

## Track 1 — Full triplet (epic + tasks + plan)

Three linked documents in `docs/specs/SPEC-{NNN}-{slug}/`:

1. **`epic.md`** — full description of the work: goals, context, requirements, UX, data model, architecture, acceptance criteria. Read by both an AI agent (as a prompt to decompose work) and a human (to validate intent).
2. **`tasks.md`** — breakdown of the epic into concrete, actionable tasks. Agent-optimised — enough technical detail that any single task can be executed without reading its siblings.
3. **`plan.md`** — execution plan: order, dependencies, parallelisable work.

### Workflow

```
epic.md (draft → in-review → approved)  →  tasks.md  →  plan.md  →  execute
```

If the user asks to skip ahead ("just make the task list for this half-drafted epic"), push back: misaligned epic produces wasted tasks.

### When the user asks for a new spec

1. Find the next available number with `ls docs/specs/` (pattern: `SPEC-{NNN}-{slug}/`, zero-padded to three digits).
2. Ask the user for a short slug (kebab-case, e.g. `student-intake`).
3. Create `docs/specs/SPEC-{NNN}-{slug}/`.
4. Copy `templates/epic.md` into the new directory as `epic.md`.
5. Fill the Meta section (ID, title, `status=draft`, dates, owner).
6. Work with the user section by section. Do not auto-fill everything — the epic is a design conversation, not a dump.

### When the user asks for tasks

1. Confirm the epic is at least `in-review`. Refuse to decompose a `draft` epic.
2. Copy `templates/tasks.md` into the spec directory (or create `tasks/` dir — see "Splitting tasks into files").
3. Draft the task index first, then expand each task block. Discuss the index with the user before expanding — alignment on slicing saves hours later.
4. Every task must follow the rules in "Task-writing rules" below.

### When the user asks for a plan

1. Confirm `tasks.md` exists.
2. Copy `templates/plan.md`. Use task metadata (`blockedBy`, `blocks`, `size`) to compute order and parallel tracks.

---

## Track 2 — Small spec (single file)

One document under `docs/specs/SPEC-{NNN}-{slug}/spec.md`. No triplet, no decomposition, no plan.

Use when: single well-understood change, one to three files of impact, low risk, executable in a single sitting.

### Workflow

```
spec.md (draft → approved)  →  execute directly from this file
```

No `in-review` step — small scope doesn't warrant a partner gate. User eyeballs `draft`, flips to `approved`, execution starts from the same file.

### When the user asks for a small spec

1. Find the next SPEC-NNN number (same sequence as full specs — `ls docs/specs/`). Small and full specs share the namespace; the shape is what differs, not the ID.
2. Ask for a kebab-case slug.
3. Create `docs/specs/SPEC-{NNN}-{slug}/`.
4. Copy `templates/small-spec.md` into the directory as `spec.md`.
5. Fill with the user, section by section. Do NOT auto-fill — brief discussion still saves rework.
6. When status is `approved`, execute directly from `spec.md`. No task agents, no waves — it's one sitting.
7. Same DoD applies as the full track (types, lint, tests, UX walkthrough for UI, i18n parity, etc.).
8. At hand-off, present the manual-verification checklist from §7 of the spec.

### When a small spec grows

If during execution the change turns out to be bigger than anticipated (new API, new data model, multi-surface work) — **pause, tell the user, and convert to full triplet**. The existing `spec.md` becomes the kernel of the new `epic.md`; rename the file and expand it. Keep the SPEC-NNN ID.

---

## Track 3 — Hotfix (act first, document alongside)

One document under `docs/specs/HF-{NNN}-{slug}/hotfix.md`. Sits next to SPEC folders in `docs/specs/` — hotfixes are spec artefacts, just a different shape and a parallel numbering sequence (`HF-001`, `HF-002`, …).

Use when: production (or a critical dev surface) is broken, the user describes the problem, and the fix is needed now.

### Workflow

```
problem triaged  →  HF doc sections 1–2 created  →  regression test (red)
  →  fix  →  test green  →  HF doc sections 3–8 filled  →  single commit
```

Sections 1–2 (Meta + Problem) go in BEFORE coding — that's the triage paper trail. Sections 3–8 (Timeline, Root cause, Fix, Regression guard, Verification, Follow-up) go in AFTER the fix is green and BEFORE the commit. The single commit bundles fix + regression test + HF doc.

### When the user asks for a hotfix

1. Find the next HF-NNN number via `ls docs/specs/ | grep '^HF-'`. HF numbering is independent of SPEC.
2. Ask for a kebab-case slug (or derive from the user's problem description).
3. Create `docs/specs/HF-{NNN}-{slug}/`.
4. Copy `templates/hotfix.md` into the directory as `hotfix.md`.
5. Fill §1 Meta + §2 Problem from what the user said. Quote their words verbatim in §2; don't paraphrase.
6. Reproduce the bug. Write the regression test FIRST — it should be red before the fix is applied. An urgent fix without a regression guard is how the same bug recurs.
7. Apply the fix. Test turns green.
8. Fill §3 Timeline, §4 Root cause, §5 Fix, §6 Regression guard, §7 Verification, §8 Follow-up.
9. Single commit bundles fix + test + HF doc. Commit message references `HF-{NNN}`.
10. If the fix is a band-aid (symptom patched, underlying design flaw remains), §8 points to either a new small-spec or an entry in `docs/future-work.md` with `Source: HF-{NNN}`.

### What never gets skipped, even in a hotfix

- Regression test (§6). No exceptions without a documented reason and a follow-up ticket.
- DoD basics: types pass, linter clean, existing automated suites still green.
- Safety rails: no `--no-verify`, no force-push, no mock of the broken dependency to "make it pass".
- The HF doc itself. "I'll write it later" is how we lose the paper trail — which is exactly when the next similar incident hits.

If you cannot meet these on the hotfix timeline, tell the user; don't silently bypass.

---

## Content rules

### No implementation code — but contracts yes

Epics and tasks describe **what the system is**, not **how it is built**. Allowed:

- **Type / interface declarations** (TypeScript, Go structs, Python TypedDict, SQL DDL) — these are specifications.
- **Schema formats** (OpenAPI, JSON Schema, Protobuf, GraphQL SDL) for cross-language or cross-service contracts.
- **Mermaid diagrams** (flowchart, sequenceDiagram, stateDiagram-v2, erDiagram, C4-style flowchart).

Disallowed:

- Function / method bodies, algorithms as code, loops, control flow as code, test assertions as code. Describe these in prose or mermaid.

### Choosing a contract format (cross-language interfaces)

Pick the format that best fits the interface style:

| Interface style | Use |
|---|---|
| REST HTTP API | OpenAPI (generate TS / Go / Python types from it) |
| Async events, data payloads, config schemas | JSON Schema |
| RPC, strict typed inter-service calls | Protobuf / gRPC |
| GraphQL edges | GraphQL SDL |
| Single-language internal types | Native (TS `interface`, Go `struct`, etc.) |

For any contract used by more than one service or language, make the schema the source of truth and note "generate types from this schema" in the task.

### Skip sections that do not apply

For small-scope specs (a UI tweak, a single bug fix), omit irrelevant sections entirely. Do **not** leave `N/A — reason` placeholders — they add noise. Keep only sections that carry signal.

For medium or large specs, include all sections. When in doubt, include.

Sections almost always present in a non-trivial epic: Meta, Summary, Business goal, Users & roles affected, User stories & scenarios, Functional requirements, Acceptance criteria, Out of scope.

Sections often omitted for truly small specs: Glossary, Data model, Algorithms, API, Permissions matrix, Analytics, Rollout plan, Risks & assumptions, Success metrics.

Section numbers in the template are for reference, not sequence — do not renumber on omission.

### Concrete over abstract

Name real personas in scenarios, quote real copy, link to real source materials. Avoid "a user", "some data", "various options".

### Project-specific context

Project-specific conventions (tech stack, brand rules, source materials, compliance requirements, monorepo layout) live in the project's `CLAUDE.md` or `AGENTS.md`, not in this skill. Read those first before filling an epic — they inform sections like Architecture, UX, and Non-functional.

## Task-writing rules

### Self-sufficiency

Each task must be executable without reading sibling tasks. When a cross-task dependency is unavoidable (shared type, produced artefact), quote the interface in the task's **References** and link to where it is defined (e.g., `T-003 §Data structures → Example type`). A little controlled duplication beats forcing the agent to chase links.

### Task metadata

Every task carries:

| Field | Purpose |
|---|---|
| `type` | feature / refactor / test / research / bugfix / infra / docs — steers expectations |
| `size` | S / M / L (see below) — signals complexity, not hours |
| `languages` | TS, Go, Python, SQL, … — agent selects toolchain |
| `scope paths` | explicit blast radius in the monorepo |
| `blocked by` / `blocks` | dependencies for planning |
| `epic sections` | back-links for context |

**Size tiers:**

- **S** — 1 file or 1 component, trivial logic
- **M** — a few files, crosses modules, involves small design decisions
- **L** — multiple layers, needs further decomposition — candidate for split

### Splitting tasks into files

Rule of thumb based on task count in the epic:

- **≤ 10 tasks** — one `tasks.md` with full task blocks inline.
- **> 10 tasks** — extract each task to `tasks/T-NNN-{slug}.md`, leave only the task index table in `tasks.md`.

Judgement call by the author — not a hard cutoff. For a 9-task spec where each task is large and dense, splitting earlier is fine.

### Test flow (per task)

Tests inside each task carry four sub-sections:

1. **Strategy** — what level (unit / integration / e2e / a11y / visual), what tools. Default: automate as much as the toolchain allows (Playwright, Vitest, axe-core, etc.). Manual verification is the exception, not the rule. For E2E: pin `locale`, `timezoneId`, and `extraHTTPHeaders` (especially `Accept-Language`) in the runner config to a representative real user — headless defaults differ from real browsers and hide header-sensitive bugs.
2. **Core scenarios (filled up-front)** — 3–5 must-pass scenarios derived from user stories and acceptance criteria. Written by the task author before implementation. Each maps to an AC.
3. **Additional scenarios (filled during implementation)** — the executing agent appends edge cases it discovered. This grows as implementation progresses and becomes part of the task artefact.
4. **Manual verification (run at task end)** — only items automation cannot reliably cover (visual polish, screen-reader flow, device-specific feel). Empty is a good outcome.

### Definition of Done (global — applies to every task)

A task may be marked `completed` only when:

- [ ] Type checks pass (`tsc --noEmit`, `go vet`, `mypy`, or the project equivalent)
- [ ] Linter clean
- [ ] Core scenarios implemented as automated tests and green
- [ ] Additional scenarios documented in the task file
- [ ] Internationalisation keys exist for all languages the project supports (if task touches UI text)
- [ ] Accessibility audit passes for surfaces the task touches (if UI)
- [ ] For UI tasks: executing agent has walked the affected surface in a real browser (screenshot / trace / live walkthrough) — automated green is about mechanics, not UX. If the agent cannot verify the UX in its environment, it must flag this explicitly at hand-off so the user knows to eyeball it.
- [ ] For features that read request headers, locale, timezone, or browser-provided signals: an adversarial test exists that flips the signal and asserts the deployment default still holds (green headless runs do not prove default-locale / default-region / default-tz behaviour).
- [ ] **Adjacent configs still agree with this task's architectural decisions** — when a task changes an integration boundary (auth model, storage layer, emulator→real, new service, port, env-var name), walk the adjacent configs that encode the SAME decision and update them in the same PR. Typical list: `.github/workflows/*.yml`, `Makefile`, `docker-compose*.yml`, `.env.example` / per-service example envs, `README.md` setup sections, dev scripts under `scripts/`. Grep for the old name / old service / old env-var — if any config mentions it, fix it here, not in a follow-up.
- [ ] Manual verification checklist executed and confirmed by the user
- [ ] PR description references the task ID and epic ID

### Task-agent completion hand-off

At task end, the executing agent MUST output a short completion summary to the user containing:

1. What changed (high-level, one paragraph)
2. Files touched (paths)
3. Tests added / modified
4. **UX verification status** (UI tasks only): "verified in browser on routes X, Y, Z" OR "could not verify UX in this environment — please eyeball {specific surfaces}". Never silently skip. Automated-green is not UX-verified.
5. **Manual verification checklist** — the full list from the task's Manual verification subsection, presented as a checklist the user can tick through
6. Any Additional scenarios appended to the task file
7. Any Open questions that emerged

This is non-negotiable. Do not let the user hunt through files for the manual checklist — surface it at hand-off.

## Plan execution

### Orchestrator / task-agent model

- **Orchestrator agent** = the session the user talks to. Responsible for reading `plan.md`, spawning task agents, verifying each completion, merging waves, running automated review gates, surfacing user-required reminders at epic end.
- **Task agent** = a sub-agent spawned per task (via the Agent tool). Executes one task end-to-end (implementation → tests → DoD checklist → hand-off). Writes its row into the Progress log before returning.

The user should NOT have to coordinate agents — the orchestrator does it.

### Concurrency cap

Default cap: **3 parallel task agents**. Overrideable per-plan in plan.md §1 (Meta). Raising the cap requires explicit user approval at plan-creation time.

### Execution modes (ask at Wave 1 kickoff)

Before spawning the first task agent of any plan, the orchestrator asks the user which mode to run:

- **`auto`** — run all waves sequentially without stopping. Orchestrator only pauses on failures or user-required review gates at epic end.
- **`paused-between-waves`** — after each wave completes (including merge + automated review), orchestrator stops and asks the user to approve starting the next wave.

Record the chosen mode in plan.md §1. It can be changed between waves by the user.

### Merge strategy

**Wave-boundary merge.** All task agents in a wave work on branches off the same base. After all tasks in the wave complete and pass DoD, the orchestrator merges the wave into the integration branch, resolves conflicts (escalating ambiguous ones to the user), runs automated review gates, then starts the next wave from the merged base.

### Dry-run preview

Before Wave 1, the orchestrator produces the Dry-run preview section of plan.md (§5):

- **Path overlaps** — cross-reference `Scope paths` of tasks in the same wave; flag shared paths as medium/high merge risk.
- **External dependencies** — aggregate packages, env vars, external services across all tasks.
- **Language / toolchain mix** — list per language.

Surface the preview to the user before kicking off. No additional tooling needed — reads task metadata only.

### Failure policy

When a task agent fails or declares itself blocked:

1. **Mark the task `blocked`** in the Progress log with a short cause.
2. **Continue other in-flight tasks in the same wave** — do not halt the whole wave.
3. **Move dependent tasks to a later wave** — any task whose `blockedBy` contains the blocked task is deferred until the block is resolved.
4. **If the blocker invalidates the epic's assumptions** (ambiguity, contradiction, missing data), the orchestrator pauses the plan and proposes a re-plan: update the epic section(s), regenerate affected tasks, recompute the plan graph. Present the diff to the user for approval before resuming.

### Review gates — automatic vs user-triggered

**Before promising a gate in `plan.md`, verify the gate's skill or plugin actually exists.** A gate that points at a missing implementation silently never fires — the plan becomes a lie, the waves complete without review, and nobody knows until someone manually runs the command at the end.

Check:
- `ls .claude/skills/` — for project-local skills
- `ls .claude/commands/` — for project-local slash commands
- `~/.claude/plugins/installed_plugins.json` — for installed Claude Code plugins
- Plugin preconditions — e.g. `/code-review` requires a GitHub PR + configured remote; `/security-review` requires `origin/HEAD` to resolve. Document preconditions next to the gate entry in `plan.md`.

If a gate is desired but not wired, either wire it in the same PR that writes the plan, OR mark the plan entry explicitly: `aspirational — not wired as of YYYY-MM-DD`. No silent declarations.

Review steps the orchestrator runs without asking (only when the tool exists and preconditions are met):

| Gate | Trigger | Skill / command used |
|---|---|---|
| Code review (light) | end of each wave (requires: GitHub remote + PR) | `/code-review` plugin command |
| Security review | wave touches auth / sessions / consent / PII / PHI / payments (requires: `origin/HEAD` resolvable) | `/security-review` plugin command |
| Architecture review | epic end, non-trivial epics | prompt-level review agent (dedicated skill may be added later) |

Review steps the orchestrator CANNOT run — must be surfaced to the user at epic end:

| Gate | What user runs |
|---|---|
| Thorough code review | `/ultrareview` (multi-agent cloud review; user-triggered and billable) |
| Functional user acceptance | aggregated manual verification checklist across all completed tasks |

### Orchestrator completion hand-off (epic end)

When the last wave completes and automated reviews are done, the orchestrator MUST output a single final summary containing:

1. **Epic completed:** SPEC-NNN title
2. **Tasks executed:** count + link to Progress log
3. **Automated review results:** code-review, security-review, architecture review — pass / fail / findings
4. **User actions required — run these commands:**
   - `/ultrareview` — for thorough multi-agent code review of the branch
   - Any other user-triggered commands the project requires
5. **Aggregated manual verification checklist** — one checklist combining all tasks' manual items, grouped sensibly (e.g., by feature area or by check type)
6. **Open questions / deviations from the epic**

Do not ask the user to hunt through plan.md or task files. Everything they must act on appears in this final summary.

## Bug reports during spec execution

Bug reports are a **side-output** of any track (full triplet / small-spec / hotfix). They are not their own track. When a task agent surfaces a defect — production code does not match intended behaviour as defined by the originating SPEC — the agent files a bug report under `docs/bugs/BUG-NNN-{slug}.md` and continues. The fix happens in a separate hotfix or fix-spec, not in the task that surfaced it.

This convention is project-agnostic and ships with this skill. Severity scale, lifecycle, numbering, and the skip-with-reference test annotation are codified once here so every spec inherits the same rules.

### When to file a bug

- A test that captures intended SPEC behaviour fails because the production code does the wrong thing.
- A code review or manual repro reveals behaviour that contradicts a SPEC clause.

When NOT to file a bug — see the anti-pattern section in `templates/bugs-readme.md`. Litmus test: **can you cite the SPEC section the code violates?** If yes, file. If no, the right home is `docs/future-work.md` or a discussion with the orchestrator.

### How to file (first-time setup)

If `docs/bugs/` does not exist in the project yet:

1. Create `docs/bugs/`.
2. Copy `templates/bugs-readme.md` → `docs/bugs/README.md`.
3. Copy `templates/bug.md` → `docs/bugs/_template.md`.
4. Continue with step 1 of "How to file (steady state)".

### How to file (steady state)

1. Determine the next `BUG-NNN`: `max(existing BUG-NNN in docs/bugs/) + 1`, three-digit zero-padded. If the directory is empty, start at `BUG-001`.
2. Copy `docs/bugs/_template.md` → `docs/bugs/BUG-NNN-{slug}.md`. Slug is short kebab-case.
3. Fill every section. Severity per the four-level scale in `docs/bugs/README.md` (low / medium / high / critical). Suggested fix track: `hotfix` (prod-broken, urgent, isolated fix) or `fix-spec` (design pass needed).
4. Annotate the originating test with the project's idiomatic skip-with-reference. The `BUG-NNN` token is mandatory and must be discoverable by `grep -rEn 'blocked by BUG-[0-9]+'`. Per-stack idioms are listed in `templates/bug.md` § "Test status".
5. Continue executing the task. Skipped tests do NOT count toward the file's executed coverage. Coverage gates either accept the gap as bug-blocked (if AC explicitly allows it) or the orchestrator escalates.

### Hand-off implications

Every spec hand-off (small-spec, full-triplet T-FINAL, hotfix close) must:

- Confirm every `it.skip("blocked by BUG-")` / `t.Skip("blocked by BUG-")` / equivalent in the diff has a matching `docs/bugs/BUG-NNN-*.md` file with all template sections filled.
- List filed BUG-NNN with severity and suggested fix track in the hand-off summary.
- Append a `docs/future-work.md` entry per open BUG so the next planning round picks it up — unless a fix-spec has already been started.

A spec is NOT considered done if a skip-with-reference annotation in its diff points at a non-existent BUG-NNN, or if a filed BUG-NNN has missing template sections.

---

## Directory layout this skill expects

```
<repo>/
├── docs/
│   ├── specs/
│   │   ├── SPEC-001-{slug}/         ← full triplet
│   │   │   ├── epic.md
│   │   │   ├── tasks.md
│   │   │   └── plan.md
│   │   ├── SPEC-002-{slug}/         ← full triplet, >10 tasks
│   │   │   ├── epic.md
│   │   │   ├── tasks.md             ← index only
│   │   │   ├── tasks/
│   │   │   │   ├── T-001-{slug}.md
│   │   │   │   └── T-002-{slug}.md
│   │   │   └── plan.md
│   │   ├── SPEC-003-{slug}/         ← small spec (one file)
│   │   │   └── spec.md
│   │   └── HF-001-{slug}/           ← hotfix (one file)
│   │       └── hotfix.md
│   └── bugs/                        ← created lazily on the first BUG
│       ├── README.md                ← convention (from bugs-readme.md template)
│       ├── _template.md             ← per-bug template (from bug.md template)
│       ├── BUG-001-{slug}.md
│       └── BUG-002-{slug}.md
```

Create `docs/specs/` if it does not yet exist. All three spec shapes live in the same directory — shape reads from the folder contents, not from the ID prefix (except for the `HF-` vs `SPEC-` split). Create `docs/bugs/` lazily on the first defect filing.

## Files in this skill

- `SKILL.md` — you are here. Workflow and rules.
- `templates/epic.md` — epic template (full triplet).
- `templates/tasks.md` — task-list template (full triplet).
- `templates/plan.md` — execution-plan template (full triplet).
- `templates/small-spec.md` — one-file spec template.
- `templates/hotfix.md` — one-file hotfix template.
- `templates/bug.md` — per-bug report template (`docs/bugs/BUG-NNN-{slug}.md`).
- `templates/bugs-readme.md` — `docs/bugs/README.md` template (numbering, severity, lifecycle, skip-with-reference convention, anti-pattern note).

## Portability

This skill is intentionally project-agnostic. To use it in a new repo, copy `.claude/skills/spec-development/` from any project that already has it. No further setup needed beyond ensuring the target repo has (or will have) a `docs/specs/` directory and a `CLAUDE.md` describing project-specific conventions.
