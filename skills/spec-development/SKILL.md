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

### Phase 0 — recon, for brownfield work only

Ask once, at spec creation: **"is this a change to existing code you did not write, or do not already hold in your head?"** Record the answer in `epic.md` §1 Meta as `Brownfield: yes/no`. If yes:

1. Copy `templates/recon.md` → `docs/specs/SPEC-{NNN}-{slug}/recon.md` and fill it **before** the epic.
2. Every recon claim cites a concrete path, symbol, or command output. A section with no citation counts as unfilled — an uncited recon is a paraphrase of the epic that reads like evidence.
3. `epic.md` §4 "Prior art" and §11 then **reference** `recon.md` instead of restating it.

Greenfield work skips this entirely. The question is asked once and never again.

### When the user asks for a new spec

1. Find the next available number with `ls docs/specs/` (pattern: `SPEC-{NNN}-{slug}/`, zero-padded to three digits).
2. Ask the user for a short slug (kebab-case, e.g. `student-intake`).
3. Create `docs/specs/SPEC-{NNN}-{slug}/`.
4. Copy `templates/epic.md` into the new directory as `epic.md`.
5. Copy `templates/verification-checklist.md` into the new directory as `verification-checklist.md` — universal floor goes in immediately, project-/spec-specific surfaces (§10+) get filled as the epic takes shape.
6. Fill the Meta section (ID, title, `status=draft`, dates, owner).
7. Work with the user section by section. Do not auto-fill everything — the epic is a design conversation, not a dump.

### Before the epic leaves `draft` — completeness self-check

At the `draft` → `in-review` transition (or on demand), run a **mechanical** pass over the epic and report the gaps in the conversation as questions for the author. Not a review, not a subagent, not a verdict — the author's own read covers intent; this covers the clerical work a human reading for intent reliably skips:

1. Every `FR-N` is covered by at least one acceptance criterion, and every `AC-N` traces back to at least one requirement.
2. Every acceptance criterion names an **observable** outcome — something a third party can call met or not met ("must be fast" fails this).
3. No section still carries template placeholder text or an unfilled `{…}` slot.
4. Every surface the epic touches that the universal floor does not cover has a matching §10+ section in `verification-checklist.md`.
5. §21 Out of scope is non-empty.

Checks 3 and 5 are literals — **run** these and paste the real output (`VR-15`); 1, 2 and 4 stay a read, because a script can only check that identifiers co-occur, which is a different question from whether the criterion covers the requirement:

```
grep -nE '\{NNN\}|\{name|\{slug\}|\{title|\{link|\| YYYY-MM-DD|Lorem ipsum|Delete (these instructions|this block|the "How)' epic.md   # → nothing
sed -n '/^## 21\./,/^## 22\./p' epic.md | grep -c '^- [^.]'                                            # → > 0
```

Nothing here blocks: the author decides what to fix. Under `Compliance critical: true` it must have been run before `approved`, with each gap either fixed or recorded as a row in §22 Open questions.

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
5. Copy `templates/verification-checklist.md` into the directory as `verification-checklist.md`. §0 binds on this track too, and a file the agent is told to honour must exist.
6. Fill with the user, section by section. Do NOT auto-fill — brief discussion still saves rework.
7. When status is `approved`, execute directly from `spec.md`. No task agents, no waves — it's one sitting.
8. Same DoD applies as the full track (types, lint, tests, UX walkthrough for UI, i18n parity, etc.).
9. At hand-off, present the manual-verification checklist from §7 of the spec.

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
4. Copy `templates/hotfix.md` into the directory as `hotfix.md`, and `templates/verification-checklist.md` as `verification-checklist.md` — §0 binds here too (see the three tiers under "Per-spec verification checklist", including the one deferrable item).
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

### Verification rigour — anti-hand-wave rules

Past specs in many projects have shipped with "green tests" that proved mechanics, not behaviour — agents claim a task done because their tests went green, but the tests didn't actually verify the user-facing or persisted-state outcome the spec promised. The rules below are a universal floor against this failure mode. Each rule has been the source of a real bug-after-merge in some past project; they are not academic.

Apply to **every task**, regardless of language / framework / spec scope. Specs MUST author a `verification-checklist.md` in their directory (see template + workflow below) that quotes these rules verbatim and extends them with project-/spec-specific surfaces.

Each rule has exactly one identifier, `VR-01`…`VR-18`. `HR-1`..`HR-4` are the former names of `VR-15`..`VR-18` — pre-1.1 specs may still use them; new text must not.

- **VR-01 — Persistence asserted by storage query, not by helper-call inspection.** When a task says "X is recorded / written / inserted", the test queries the underlying store (`SELECT … FROM <table> WHERE …`, equivalent NoSQL get, file-existence check) and asserts presence + key fields. Calling the helper that *would* write does not count. The most common false positive: helper returned nil, the row was never created.
- **VR-02 — State-machine transitions asserted by post-condition row read.** When status changes, the test reads the row back and asserts the new status. "The handler returned 200" is necessary but not sufficient — the handler can return 200 from a code path that skipped the write.
- **VR-03 — Audit-row existence asserted by storage grep, not by code path.** When a task claims to write an audit-log row, the test queries the audit storage and asserts the actor / entity / action / reason fields match the input verbatim. The audit-helper returning nil is not sufficient.
- **VR-04 — Concurrency races asserted by post-condition row count.** When you claim "second concurrent write returns 409", the test runs N parallel writers (real goroutines / async tasks, not sequential calls) and asserts the resulting row count = 1 (only winner left a row). Sequential calls do not test concurrency.
- **VR-05 — Negative tests as first-class.** Every guard, validator, or rule has both a positive case (fires/passes correctly) AND a negative case (does NOT fire/pass on a clean input). Half-tested guards rubber-stamp.
- **VR-06 — Content assertions, not shape assertions.** A test that asserts "an item came back with N fields and an integer id" is necessary but NOT sufficient. Add at least one *content-shaped* property: text length > N chars, no placeholder strings (`Lorem ipsum`, `TODO`, `Step N`), no raw markdown fences in JSON output, mandatory non-empty strings. Catch the case where the shape is right but the content is junk. **Build-time-inlined env vars are a special case:** when the framework substitutes env values into the bundle at build time (Next.js `NEXT_PUBLIC_*`, Vite `VITE_*`, similar Webpack / esbuild substitutions), the test must assert against a built artefact, not against a dev-mode runtime — the runtime container's env is irrelevant for these once the bundle is produced. A "framework injects this at build" mismatch never shows up in dev, only in built containers.
- **VR-07 — Browser walkthrough for every UI surface.** Hand-off bar = green tests + live dev env up + pages personally clicked. If the executing agent cannot run a browser, surface this explicitly in the hand-off as `could not verify UX in env — needs eyeball by user on <route>`. Do NOT silently mark complete. Per project memory in many repos: green tests prove mechanics, not UX.
- **VR-08 — Computed style / DOM state asserted, not just className.** When you claim a UI behaviour ("renders right-to-left", "is disabled"), the test asserts `getComputedStyle(el).<prop>` (or framework equivalent: `aria-disabled`, `dataset.state`), not just `el.classList.contains('rtl-class')`. Utility-CSS frameworks (Tailwind, etc.) can fail to apply silently — the class is on the element but the style is not.
- **VR-09 — E2E adversarial pinning + flip variant.** Per project memory in many repos: Playwright headless defaults differ from real Chrome on `locale`, `timezoneId`, `Accept-Language`. Pin these to a representative real user in the runner config. Add at least one variant that *flips* the signal (e.g., `Accept-Language: he-IL` against an EN-default deployment) to verify the deployment default still holds.
- **VR-10 — Image / external-resource render asserted, not just fetched.** "200 OK on an image fetch" is necessary but not sufficient. The test asserts `image.naturalWidth > 0` after load (or the equivalent runtime confirmation that the bytes were valid). A 200 OK with corrupt bytes would pass an HTTP-status check but fail this — that's the whole point.
- **VR-11 — No mocked-only verification on tasks where the external API IS the unit under test.** Mocked tests prove plumbing, not behaviour. When the spec involves an external API (LLM provider, payment processor, third-party identity, etc.) AS the unit being verified, hand-off requires at least one real-stack run against real dependencies before the spec is closed (typically a `live-smoke` task gates this). Mocked-only verification does NOT close such tasks.
- **VR-12 — Done means another engineer can reproduce the green state from scratch.** Hand-off includes the exact command sequence (commit hash, branch, env) and expected output. "It works on my machine" never closes a task.
- **VR-13 — Live exercise IS the test for infra / deployment specs.** Paper validation (linter, config-validate, dry-run) proves syntax, not behaviour. For specs whose primary deliverable is infrastructure or deployment topology, the live-smoke target executed against real cloud / real dependencies is the only meaningful verification — not unit tests, not config validation. A task in such a spec is `done-paper` after paper validation but stays `done-paper` until the smoke target returns green from a real deploy. Flip to `done` only then. The first live exercise of a non-trivial infra spec routinely surfaces multiple bugs that paper validation cannot catch — UID/permission mismatches between host and container conventions, build-time vs runtime env-var bundling, lifecycle scripts that aren't re-runnable, healthcheck primitives that don't exist in distroless images, ACL prerequisites at the cloud provider that must be pre-declared. Plan the spec assuming the first live exercise will produce 3–8 hot-fix commits, and budget time accordingly.
- **VR-14 — Idempotent setup scripts must auto-detect their re-run state.** A bootstrap, migration, or provisioning script that runs N steps must not assume it always starts from a clean state. After step K runs once, the system is in a different shape; the next invocation must detect "step K already done" and either skip it or pick the right re-run path. Common failure mode: a script that disables root SSH at step 3, then is re-run as `root`, fails at step 1 because the entry-point user is no longer reachable. Mitigation pattern: every step that mutates a long-lived environment has a pre-check (file existence, user existence, probe response) that decides whether to skip / replay / pick the alternative path; the script's wrapper similarly auto-detects the entry path (which user to ssh as, which step to start from). Test idempotence by running the script twice in succession on a clean target.
- **VR-15 — Exit-code honesty — read each step, not the tail.** A multi-step run's success is read from each step's own result, not from the exit code of the last command in a pipe or a trailing `grep`/`tail` (which mask the real status). A backgrounded "exit 0" is not proof — read the artefact / log file the run produced, not the wrapper's exit code.
- **VR-16 — No un-gated or silently-skipped test layers.** Every test layer that matters runs in the project's gate. A test that skips itself **without** a `BUG-NNN` skip-with-reference is treated as rot, not as a pass. A coverage percentage does not catch a whole layer rotting; a no-silent-skip gate does. If a layer cannot run in the executing environment, surface it explicitly at hand-off — do not let it quietly drop.
- **VR-17 — Scope assertions to rows you created, never a global total.** Assertions target the specific rows / entities the test created (filter by seeded id / unique marker), never a global count or "the whole list has N". Global-count assertions are fragile under parallelism and cross-task contamination — and with the `dw` engine making parallel execution first-class, this is mandatory, not optional. Clean up what you create.
- **VR-18 — A domain failure must surface as a non-2xx.** A business / domain failure returns 4xx or 5xx — never an HTTP 2xx carrying an error body (`200` with `status:"failed"`). A success status on a failed operation defeats HTTP-level observability and any poll / health assertion that keys on the status code; it is a "green that lies".

### Definition of Done (global — applies to every task)

A task may be marked `completed` only when:

- [ ] Type checks pass (`tsc --noEmit`, `go vet`, `mypy`, or the project equivalent)
- [ ] Linter clean
- [ ] Core scenarios implemented as automated tests and green
- [ ] Additional scenarios documented in the task file
- [ ] Internationalisation keys exist for all languages the project supports (if task touches UI text)
- [ ] Accessibility audit passes for surfaces the task touches (if UI)
- [ ] **Verification rigour rules above applied** — all 18 accounted for in the hand-off's §8 report: satisfied, or `n/a` naming the surface the task does not touch, or — only on the three rules where the §8 template offers the option — waived with a reason explaining why the rule is inapplicable *to this diff*. MANDATORY items cannot be waived at all: a waived MANDATORY item is a `waiver-abuse` finding at review, not a judgement call. Walk the spec's `verification-checklist.md` end-to-end before marking complete.
- [ ] For UI tasks: executing agent has walked the affected surface in a real browser (screenshot / trace / live walkthrough) — automated green is about mechanics, not UX. If the agent cannot verify the UX in its environment, it must flag this explicitly at hand-off so the user knows to eyeball it.
- [ ] For features that read request headers, locale, timezone, or browser-provided signals: an adversarial test exists that flips the signal and asserts the deployment default still holds (green headless runs do not prove default-locale / default-region / default-tz behaviour).
- [ ] **Adjacent configs still agree with this task's architectural decisions** — when a task changes an integration boundary (auth model, storage layer, emulator→real, new service, port, env-var name), walk the adjacent configs that encode the SAME decision and update them in the same PR. Typical list: `.github/workflows/*.yml`, `Makefile`, `docker-compose*.yml`, `.env.example` / per-service example envs, `README.md` setup sections, dev scripts under `scripts/`. Grep for the old name / old service / old env-var — if any config mentions it, fix it here, not in a follow-up.
- [ ] **Architecture doc still reflects reality.** If the project has `docs/architecture/` (or equivalent), the task author walks the relevant section(s) and updates them in the SAME PR — new auth surface → auth section, new table → data-model section, new container/port → topology section, new audit action → observability section, new admin endpoint → admin-flow section. Bump the "as-of" date on the architecture doc's index. A code change without the doc update is a bug in the doc.
- [ ] Manual verification checklist executed and confirmed by the user
- [ ] PR description references the task ID and epic ID

### Task-agent completion hand-off

At task end the executing agent MUST output the completion summary using the **§8 Hand-off summary template** in the spec's `verification-checklist.md`. That template is the single source for the required slots — this file deliberately does not restate them, because two copies of one list drift (they already had).

Non-negotiable: fill every slot; an empty or missing slot means the task is not complete. Do not make the user hunt through files for the checklist — surface it at hand-off. The task agent also **returns its Progress-log row here** rather than editing `plan.md` (see "Orchestrator / task-agent model").

### Per-spec verification checklist

Every spec MUST author a `verification-checklist.md` in its directory (alongside `epic.md` / `tasks.md` / `plan.md` for full-triplet, alongside `spec.md` for small-spec, alongside `hotfix.md` for hotfix). The checklist is the agent-facing single-page contract for hand-off; it quotes the universal rigour rules above and adds project-/spec-specific surfaces.

**When to author:** the same time as the epic, before tasks.md. The author cannot enumerate spec-specific surfaces (e.g., "Hebrew Unicode guard", "S3 round-trip via mc", "IRT spread realism") until the epic exists, but the universal sections can be copied verbatim from the template.

**Workflow:**

1. Copy `templates/verification-checklist.md` → `docs/specs/SPEC-NNN-{slug}/verification-checklist.md` immediately after creating the directory.
2. The universal sections (§0 Universal, §1–§7 surface families, §8 Hand-off template, §9 Orchestrator pre-merge gate) stay verbatim — they are the floor.
3. Add a new section §10+ "Project / spec-specific surfaces" for whatever the epic touches that the universal floor does not cover. The template's §10+ block lists worked examples (localised content, object-storage round-trips, numerical fixtures, external API as SUT, multi-region) — read them there rather than here.
4. Tasks reference the checklist in their AC blocks where applicable. The orchestrator gates merges on the checklist's MANDATORY items **and its §10+ items** — §10+ is the only part of the file that is specific to this spec, so a gate that skips it gates nothing the author added.

**Which parts bind on which track — three tiers, no judgement call:**

- **§0 Universal** — every track, no exceptions. Full triplet, small-spec, hotfix.
- **§1–§7 surface families** — bind on whichever tracks and tasks touch that surface (persistence, UI, audit, concurrency, E2E, external API, i18n). Untouched surface ⇒ `n/a` with the reason, not silence.
- **The full file including §10+** — mandatory for full-triplet; optional for small-spec and hotfix (still recommended: copying the template costs five minutes).

*Pre-1.1 specs.* A spec directory created before 1.1 may carry no `verification-checklist.md` at all (small-spec / hotfix, where the copy step did not exist) or one whose §8 reports 12 rules instead of 18. When picking such a spec up, refresh it from `templates/verification-checklist.md` before the first hand-off — the rules are the floor; the copy in the spec directory is only a copy. Nothing else in a pre-1.1 spec needs changing.

*Hotfix carve-out.* Of the §0 items exactly **one** may be deferred under incident pressure — the **negative paired test** — and only with a §8 line naming where the follow-up is tracked. The **adjacent-configs sweep stays mandatory on every track**: it is a grep, and a hurried fix that renames one env var and misses the other four places is the classic hotfix regression. No other §0 item is deferrable anywhere.

## Plan execution

### Orchestrator / task-agent model

- **Orchestrator agent** = the session the user talks to. Responsible for reading `plan.md`, spawning task agents, verifying each completion, merging waves, running automated review gates, surfacing user-required reminders at epic end.
- **Task agent** = a sub-agent spawned per task (via the Agent tool). Executes one task end-to-end (implementation → tests → DoD checklist → hand-off). **Returns** its Progress-log row as part of the hand-off; it does **not** edit `plan.md`.

The user should NOT have to coordinate agents — the orchestrator does it.

**Progress log has one writer.** The orchestrator writes `plan.md` §11 after accepting a hand-off. Task agents never write it — up to `Concurrency cap` agents editing one Markdown table in one file is a lost-update race with no owner.

**Status flows bottom-up once work starts.** Up to `approved` the epic leads — that transition is the author's call and nothing is executing yet. After it, **tasks are the ground truth for progress**: the first task to start makes the epic `in-progress`; all tasks `completed` makes the plan `completed` and the epic `done`, with the plan sitting in the middle as the aggregate. The orchestrator writes all three and never adopts the more optimistic value. Exactly one combination is impossible and stops the run to ask — **a task executing while the epic is not yet `approved`**, i.e. building against requirements nobody agreed. Everything else is a normal intermediate state; an `approved` epic with a `not-started` plan is just work that has not begun.

### Concurrency cap

Default cap: **3 parallel task agents**. Overrideable per-plan in plan.md §1 (Meta). Raising the cap requires explicit user approval at plan-creation time.

### Execution preferences (v1.0)

Three orthogonal axes — **engine**, **stop mode**, **review** — configured by optional rows in `plan.md` §1 Meta (see the plan template). All optional; absent ⇒ default ⇒ **identical to pre-1.0**. pre-1.0 `plan.md` files without these rows are untouched.

**Resolution precedence** (highest first): invoke flag → `plan.md` Meta rows → project `CLAUDE.md` execution defaults → skill defaults. Each layer overrides only the fields it sets; everything unset falls to the skill default.


**Engine** — `task-tool` (default) / `dw` (the Workflow tool) / `auto`.
- `auto` dispatch: choose `dw` iff `tasks_count ≥ DW min tasks` (default 8) **and** `parallel_ratio ≥ DW min parallel ratio` (default 0.6), where `parallel_ratio = widest-wave-width / total-tasks`. Otherwise `task-tool`.
- **Workflow constraints (when `dw` is resolved):** (1) the Workflow tool needs the user's opt-in — if it is not available, do **not** run silently: ask to confirm launching a workflow, or degrade to parallel `Agent` calls, stating the choice. (2) A Workflow run executes to completion and cannot pause mid-run, so **one wave = one Workflow call**; the orchestrator owns pauses *between* calls (stop mode).

**Stop mode** — `auto` / `per-wave` (default) / `per-task`. Ask at Wave 1 kickoff; record in `plan.md` §1; the user may change it between waves.
- `auto` — run all waves without stopping; pause only on failure or a user-required gate.
- `per-wave` — pause after each wave (merge + review) for approval. (= the pre-1.0 "paused-between-waves".)
- `per-task` — pause after each task. Under `dw` this degrades to "one task per wave" (a Workflow run can't pause mid-task) — a documented limitation, not a rejected combination.

**Right-sized validation (no silent guessing).** These are behavioural rules, not a parser — the substrate is an LLM reading prose. On an **unrecognised value** or an **unrecognised execution row** (e.g. a typo `Engne` or `Engine: dwx`), the orchestrator **stops and asks** rather than applying a silent default — strictness is concentrated on the safety rows (`Engine`, `Review *`, `Compliance critical`), because a typo there silently removes protection. The message states: the field, the received value, the accepted values, the file (and line where possible), and a one-line fix hint. A **missing optional row** is not an error — it takes the default.

**Invoke flags** (override for one invocation): `--engine auto|task-tool|dw`, `--stop auto|per-wave|per-task`, `--review adversarial|spot-check|none`, `--allow-dw-on-compliance` (see Compliance mode under Review orchestration).

**Backward-compatibility guarantees.** A `plan.md` with no execution rows behaves exactly as pre-1.0 — no new prompts, no engine/review behaviour. The slash command and natural-language triggers are unchanged. `SPEC/HF/BUG` numbering is unchanged. The hotfix track gains none of this (always Task tool, never DW, no auto-review). The small-spec track gains optional review only (engine is always Task tool).

### Review orchestration (v1.0)

When `Review enabled: true`, each task gets an independent **adversarial review** after implementation. Pattern: `adversarial` (default) / `spot-check` / `none`.

**What each pattern does** — no value is left to interpretation:
- `adversarial` — every task is reviewed.
- `spot-check` — a task is reviewed if **at least one** holds: `Risk: high`; its `Scope paths` intersect a surface covered by checklist §1–§7 (persistence, UI, audit, concurrency, E2E, external API, i18n); `type: infra`. Absent `Risk` counts as `medium`, which alone does not trigger review. **Every skipped task gets one line in `review.log.md`** naming the task and why it was skipped — a silent skip would reproduce `VR-16` at the orchestration layer.
- `none` — no reviewer is spawned. Recorded once in the `review.log.md` header, not per task.

**Reviewer contract — the input firewall.** The reviewer is a separate subagent that receives **only** these five inputs, in this order: (1) the spec content (`epic.md`/`spec.md`); (2) the single task block incl. `Acceptance`/`Risk` — on the small-spec track, `spec.md` in full with §5 as acceptance ground truth; (3) the final diff/output; (4) the spec's `verification-checklist.md` **including its §10+ sections**; (5) the implementer's §8 hand-off summary. `templates/reviewer-instructions.md` is the **normative source** of this list — if the two ever disagree, that file wins.

It must **not** receive the implementer's reasoning, intermediate tool calls, conversation history, or prior review rounds. Inputs 4 and 5 are artefacts, not reasoning, so they do not breach the firewall — whereas judging by a checklist the reviewer was never handed breaches the review. The hand-off reaches the reviewer as **claims to refute, not evidence**; a claim not independently verifiable from the diff is itself a finding.

Its primary instruction is `templates/reviewer-instructions.md` — **`SKILL.md` is NOT loaded for the reviewer**, which is why every rule it must apply is written out in that file or arrives as one of the five inputs. The firewall itself is largely free: a freshly spawned subagent starts without the implementer's context.

**Per engine:**
- **Task tool** — after the implementation Agent call, the orchestrator makes a second `Agent` call for the reviewer, prompt = `reviewer-instructions.md` + the five inputs above, and (for structured output) instructs the exact `review-verdict-template.md` format.
- **`dw`** — the reviewer is a pipeline stage: `agent(reviewerPrompt, {agentType: 'reviewer', schema: VERDICT})`. `agentType` loads `reviewer-instructions.md`; `schema` validates the verdict at the tool-call layer.

**Verdict** — structured (`review-verdict-template.md`): `PASS` / `FAIL` / `NEEDS_REVISION` + findings (`severity` critical/major/minor, `category` spec-deviation/silent-failure/acceptance-miss/quality/test-gap/waiver-abuse, `suggested_action`). Categories are tied to `verification-checklist.md` MANDATORY items + the §"Verification rigour" rules — one anti-hand-wave system, not two. `waiver-abuse` covers the one place an agent may legally declare a rigour rule inapplicable: every waiver and every `n/a` in the hand-off is verified against the diff. Append every round to `review.log.md` (header from `review-log-template.md`).

**`fail_action`** — `revise` (default; return findings to the implementer and re-run) / `halt` (stop on first non-PASS) / `flag-only` (record the finding, accept the task, continue).

**Three-strikes (hard safety rail).** Keep a per-task strike counter within a run; increment on each `FAIL`/`NEEDS_REVISION`. At **3 consecutive non-PASS verdicts** on one task, **hard-halt regardless of `fail_action`** — output the task ID, the three rounds of findings, and ask the user to decide. (`flag-only` doesn't loop; `halt` stops at one. Three-strikes guards the `revise` loop against running forever.)

**Compliance mode.** Declared in `plan.md` §1 — or, on the two tracks that have no `plan.md`, in `spec.md` §1 (small-spec) / `hotfix.md` §1 (hotfix) — or by a project `CLAUDE.md` compliance label. A `CLAUDE.md` label covers **every** track and a per-spec row cannot lower it: a spec may raise compliance, never waive it.

Evaluate **before engine dispatch**: (1) `dw` is **blocked** — override only via `--allow-dw-on-compliance`, which triggers an explicit confirmation prompt (decline ⇒ halt); (2) `Review enabled` is forced `true`; (3) `Review pattern` is forced `adversarial`; (4) `Review fail action: flag-only` is rejected (must be `revise`/`halt`); (5) an epic modified without a §26 Change-log row is a **halt**, not a warning, and the row is mirrored into `review.log.md` on the next run; (6) the epic completeness self-check must have been run before `approved`, with every gap fixed or recorded in §22 Open questions. **On the hotfix track** (no epic, no `approved` state) clauses 5–6 do not apply, and clauses 2–3 deliberately override this track's default of no automatic review — that is the point of declaring a hotfix compliance-critical. These forced values are logged in the `review.log.md` header so the audit trail shows why they differ from `plan.md`.

### Merge strategy

**Wave-boundary merge.** All task agents in a wave work on branches off the same base. After all tasks in the wave complete and pass DoD, the orchestrator merges the wave into the integration branch, resolves conflicts (escalating ambiguous ones to the user), runs automated review gates, then starts the next wave from the merged base.

**Schema changes break wave rollback — gate them.** Reverting a wave reverts code; it does not un-drop a column. So:

- **Destructive DDL** = `DROP COLUMN` / `DROP TABLE` / type narrowing / `NOT NULL` on a populated column without a default — anything that discards or narrows existing data.
- A task containing destructive DDL **does not share a wave with tasks that depend on it**. Enforced when waves are computed, not discovered at merge.
- The **down-migration is verified before the wave merges** — for *every* migration, not only destructive ones ("reversible" is a claim until exercised). Rolling back a wave rolls back its schema too.
- Destructive DDL is enumerated in `epic.md` §19 Rollout plan at draft time **and** requires explicit user confirmation at the moment the wave merges. The confirmation names what data is lost — declaring it at draft time is not the confirmation.
- Additive migrations (new column, backfill, new index) wave normally.

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
4. **If the blocker invalidates the epic's assumptions** (ambiguity, contradiction, missing data), the orchestrator pauses the plan and proposes a re-plan: update the epic section(s), regenerate affected tasks, recompute the plan graph. Present the diff to the user for approval before resuming. **Before any task is regenerated**, the epic's `Version` is bumped and a row is added to `epic.md` §26 Change log (small-spec: §10) recording what changed, which tasks it invalidated, why, and who approved it. Otherwise the requirements have no audit trail while the reviews have one — and nobody can later answer "did the requirement move mid-flight".

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
| DB change review | wave contains a migration or DDL | the project's DB-review skill **if one is installed** (check per the rule above); if none, mark the gate `aspirational` — the wave rule and the destructive-DDL confirmation under "Merge strategy" still apply, they need no tooling |
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

## Cross-spec files (created lazily, like `docs/bugs/`)

Two project-level registers are referenced by the templates. Neither is created up front; both are created the first time something is deferred to them. If the file does not exist yet:

1. Copy `templates/future-work.md` → `docs/future-work.md`, or `templates/prod-readiness.md` → `docs/prod-readiness.md`.
2. Append the entry that triggered creation.
3. Continue the task — creating the register is never a reason to stop.

- **`docs/future-work.md`** — deferred work that is **not** a defect: missing features, polish, band-aid follow-ups, coverage gaps. `FW-NN` numbering. Litmus test against `docs/bugs/`: if you can cite the SPEC clause the code violates, it is a BUG; if you cannot, it is future work.
- **`docs/prod-readiness.md`** — pre-production steps **no agent can perform** (console setting, DNS, partner sign-off, credential provisioning). Every item has a named human owner and a blocking / non-blocking flag.

Both are surfaced in the orchestrator's epic-end hand-off; do not let the person doing the deploy discover them.

---

## Directory layout this skill expects

```
<repo>/
├── docs/
│   ├── specs/
│   │   ├── SPEC-001-{slug}/         ← full triplet
│   │   │   ├── epic.md
│   │   │   ├── tasks.md
│   │   │   ├── plan.md
│   │   │   ├── verification-checklist.md
│   │   │   ├── review.log.md        ← only when review is enabled for a run
│   │   │   └── recon.md             ← only for brownfield specs (Phase 0, written before the epic)
│   │   ├── SPEC-003-{slug}/         ← small spec (one file)
│   │   │   ├── spec.md
│   │   │   └── verification-checklist.md   ← §0 binds here too; copied at creation
│   │   └── HF-001-{slug}/           ← hotfix (one file)
│   │       ├── hotfix.md
│   │       └── verification-checklist.md ← §0 binds here too (one item deferrable)
│   ├── bugs/                        ← created lazily on the first BUG
│   │   ├── README.md                ← convention (from bugs-readme.md template)
│   │   ├── _template.md             ← per-bug template (from bug.md template)
│   │   └── BUG-001-{slug}.md
│   ├── future-work.md               ← created lazily on the first deferral (from future-work.md template)
│   └── prod-readiness.md            ← created lazily on the first pre-prod step (from prod-readiness.md template)
```

Create `docs/specs/` if it does not yet exist. All three spec shapes live in the same directory — shape reads from the folder contents, not from the ID prefix (except for the `HF-` vs `SPEC-` split). `docs/bugs/`, `docs/future-work.md`, and `docs/prod-readiness.md` are all created lazily on first use.

## Files in this skill

- `SKILL.md` — you are here. Workflow and rules.
- `templates/epic.md` — epic template (full triplet).
- `templates/tasks.md` — task-list template (full triplet).
- `templates/plan.md` — execution-plan template (full triplet).
- `templates/small-spec.md` — one-file spec template.
- `templates/hotfix.md` — one-file hotfix template.
- `templates/verification-checklist.md` — per-spec agent-facing checklist (universal floor + project-specific extension slots). Copied into every spec directory.
- `templates/bug.md` — per-bug report template (`docs/bugs/BUG-NNN-{slug}.md`).
- `templates/bugs-readme.md` — `docs/bugs/README.md` template (numbering, severity, lifecycle, skip-with-reference convention, anti-pattern note).
- `templates/future-work.md` — `docs/future-work.md` template (`FW-NN` register of deferred non-defect work). Created lazily.
- `templates/prod-readiness.md` — `docs/prod-readiness.md` template (pre-production human steps with named owners). Created lazily.
- `templates/recon.md` — brownfield Phase-0 recon template (what exists today; written before the epic, cited by it).
- `templates/reviewer-instructions.md` — primary instruction for the adversarial reviewer subagent (v1.0); loaded instead of `SKILL.md`.
- `templates/review-log-template.md` — header for the per-run `review.log.md` (v1.0); records resolved execution prefs + compliance overrides.
- `templates/review-verdict-template.md` — one structured review-verdict entry (v1.0); fixed enums for greppability.

## Portability

This skill is intentionally project-agnostic. To use it in a new repo, copy `.claude/skills/spec-development/` from any project that already has it. No further setup needed beyond ensuring the target repo has (or will have) a `docs/specs/` directory and a `CLAUDE.md` describing project-specific conventions.
