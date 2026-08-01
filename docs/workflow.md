# Development workflow

How we take an idea from "let's build this" to "it's shipped". Anchored around the `spec-development` skill which owns the templates and mechanics; this document is the map.

Audience: you (the user), future contributors, and the AI agent working with you.

---

## Three tracks — pick one

Not every piece of work deserves a full epic. The skill supports three shapes; the size of the artefact matches the size of the work.

| Work | Track | Artefact | When |
|---|---|---|---|
| New feature, multi-surface, new data model / API, partner coordination | **Full triplet** | `epic.md` + `tasks.md` + `plan.md` | Real features; anything that needs partner review or a wave plan. |
| Single well-understood change, 1–3 files, low risk, one sitting | **Small spec** | `spec.md` (one file) | Obvious scope, no partner coordination, can be built and tested end-to-end without decomposition. |
| Prod / critical dev surface is broken, fix needed now | **Hotfix** | `hotfix.md` (one file, written alongside the fix) | Incidents. Act first; document alongside; commit once. |

All three live under `docs/specs/`. Regular work shares a single `SPEC-NNN` sequence. Hotfixes use a parallel `HF-NNN` sequence so they stand out in the directory listing.

If the tracks feel blurry on a specific piece of work, **ask before creating files**. Mis-picking either over-documents a tiny change or under-documents a real feature — both are expensive to unwind.

---

## What a spec is

A spec is the artefact that captures the work. The shape depends on the track above:

| Track | Files |
|---|---|
| Full triplet | `epic.md` (what / why), `tasks.md` (breakdown), `plan.md` (execution plan) |
| Small spec | `spec.md` — single file; Meta, Summary, Goal, Scope, ACs, Tests, Open questions |
| Hotfix | `hotfix.md` — single file; Meta, Problem, Timeline, Root cause, Fix, Regression guard, Verification, Follow-up |

The skill definition and templates live in `.claude/skills/spec-development/`. Everything in this document is enforced by that skill.

---

## The six phases (full-triplet track)

```
  Phase 1              Phase 2                Phase 3
  Brainstorm    →      /spec-development  →   Review epic
  (free chat)          (skill walks sections) (approve status)
                                                  ↓
  Phase 6              Phase 5                Phase 4
  Execute       ←      Generate plan      ←   Decompose to tasks
  (orchestrator)       (skill computes waves) (skill expands blocks)
```

The small-spec and hotfix tracks collapse most of these phases (details below).

### Phase 0 — Recon (brownfield only)

If the work touches existing code you did not write — the normal case in contract work or on an inherited service — the skill asks once at spec creation, records `Brownfield: yes` in the epic's Meta, and writes `recon.md` **before** the epic: what exists today on the affected surface, the conventions actually in force, the integration points, what must not be touched and why, and what is missing contrary to expectation. Every claim cites a path or a symbol; an uncited section counts as unfilled, because a recon without citations is a paraphrase of the epic that reads like evidence. The epic then cites `recon.md` instead of restating it.

Greenfield work skips this. The question is asked once and never again.

### Phase 1 — Brainstorm (optional for small work)

In plain chat, no skill invoked. Describe what you want to build: goal, user, why now, constraints, doubts. The agent challenges assumptions, surfaces hidden requirements, raises risks. The goal of this phase is **alignment on direction**, not wording. No file is produced.

Skip this phase for small-scope work (UI tweak, bugfix, obvious refactor). Go straight to Phase 2.

### Phase 2 — Draft the epic

Invoke the skill with any of these:

- `/spec-development`
- "let's write a spec for this"
- "turn this into an epic"
- "create SPEC for …"

What the skill does:

1. Picks the next `SPEC-{NNN}` number from `docs/specs/`.
2. Asks for a short kebab-case slug.
3. Creates `docs/specs/SPEC-{NNN}-{slug}/` and copies the epic template.
4. Copies the **verification-checklist** template into the same directory as `verification-checklist.md` — universal anti-hand-wave floor goes in immediately; spec-/project-specific surfaces (§10+) get filled as the epic takes shape.
5. Walks the epic template **section by section** — not a dump. For each section it proposes a draft based on the Phase 1 conversation; you correct, refine, or reject.
6. Skips sections that do not apply to this epic's scope (no `N/A — reason` placeholders).
7. Leaves status = `draft` at the end.

The epic is a conversation, not a form. Expect back-and-forth.

### Phase 3 — Review the epic

Read the file end to end. When you are happy with direction, change status in §1 (Meta) to `in-review`. Share with partners if needed. When ready, set it to `approved`.

At the `draft` → `in-review` transition the skill runs a **completeness self-check** and reports the gaps to you as questions. It is deliberately mechanical — the clerical work you skip when reading for intent: a requirement no acceptance criterion covers, a criterion with no observable outcome ("must be fast"), leftover template placeholder text, a surface with no matching §10+ checklist section, an empty Out-of-scope. It is not a second opinion on your intent and it blocks nothing; you decide what to fix. Under `Compliance critical: true` it must run before `approved`, with every gap fixed or written into §22.

The skill refuses to decompose a `draft` epic — scope has not stabilised yet.

**If the epic changes after that**, the skill bumps §1 `Version` and adds a row to §26 Change log — what changed, which tasks it invalidated, why, who approved — **before** regenerating any task. Reviews already had an audit trail (`review.log.md`); this gives the requirements one, so "did the requirement move mid-flight?" has an answer. Under `Compliance critical: true` an epic edited without that row is a halt, not a warning.

### Phase 4 — Decompose into tasks

Trigger: "break this into tasks" / "write tasks" / "decompose SPEC-NNN".

The skill produces `tasks.md` in two steps:

1. **Task index first** — the table of tasks only (ID, title, size, languages, scope paths, dependencies). Discuss slicing here: too big? too small? wrong boundaries? Re-shape before expanding.
2. **Task blocks** — once the index is agreed, expand each task with its full technical spec, acceptance criteria, and **Core test scenarios** (written up-front, 3–5 critical flows mapped to ACs).

If there are more than ~10 tasks, the skill moves each task into its own file under `tasks/T-NNN-{slug}.md` and leaves only the index in `tasks.md`.

### Phase 5 — Generate the plan

Trigger: "write the plan" / "generate execution plan".

From tasks.md, the skill computes:

- **Dependency graph** (mermaid, from `blockedBy` fields)
- **Execution waves** — sets of tasks with no dependencies between them, executable in parallel
- **Critical path** — longest blocking chain
- **Dry-run preview** — path overlaps within a wave (merge risk), external dependencies, language mix
- **Sub-waves** — if a wave has more tasks than the concurrency cap (default 3)
- **Review gates** — which run automatically at wave end (see below), which require user action

### Phase 6 — Execute the plan

Trigger: "execute" / "run the plan" / "start Wave 1".

The **orchestrator agent** (the session you are talking to) asks which execution mode to use:

- `auto` — run all waves back-to-back; pause only on failure or user-required review at epic end
- `paused-between-waves` — stop after each wave, wait for approval to continue

Then for each wave:

1. Spawns up to 3 **task agents** in parallel (via the Agent tool).
2. Each task agent executes its task end-to-end: implementation → tests → DoD checklist → hand-off summary. It **returns** its progress-log row in that hand-off — task agents never edit `plan.md` themselves, because several agents writing one Markdown table is a lost-update race. The orchestrator is the only writer of the Progress log.
3. After all tasks in the wave complete, orchestrator merges the wave branch and runs automatic review gates.
4. If mode is `paused-between-waves`, stops and asks before the next wave.

**Status flows bottom-up.** Up to `approved` the epic leads and you set it. After that the tasks do: the first one to start makes the epic `in-progress`, all of them closing makes the plan `completed` and the epic `done`. The orchestrator writes all three and never picks the more flattering value. One combination stops the run and asks you — a task executing while the epic is not yet `approved`, which means work is being built against requirements nobody approved.

At epic end, the orchestrator prints one consolidated hand-off containing all user-required follow-ups (see "Review gates" below).

---

## Small-spec track

For work too small to justify the full triplet. One document, one sitting.

```
  draft spec.md  →  approve  →  execute directly from the same file
```

### Trigger

- "small spec for …"
- "just a one-file fix"
- "tiny change to …"

### Flow

1. Skill picks the next `SPEC-NNN` (same sequence as full specs — shape is the differentiator, not the ID).
2. Asks for a kebab-case slug.
3. Creates `docs/specs/SPEC-{NNN}-{slug}/spec.md` from the small-spec template.
4. Walks the file with you section by section (Meta, Summary, Goal, Scope, ACs, Tests, Open questions).
5. No `in-review` step — you eyeball `draft`, flip to `approved`, execution starts from the same file.
6. No decomposition, no plan, no waves — executed directly by the agent / task agent.
7. Same Definition of Done applies as the full track (types, lint, tests, UX walkthrough for UI, i18n parity).

### When to upgrade to a full triplet

If during execution the change grows (new API surface, cross-cutting risk, partner coordination emerges) — stop, tell the user, convert `spec.md` into an `epic.md` and decompose. Keep the SPEC-NNN ID. Don't cram a full spec's worth of context into the small-spec shape.

---

## Hotfix track

Production (or a critical dev surface) is broken. The fix is urgent. The doc is written alongside the fix — not before, not after.

```
  triage  →  HF doc (Meta + Problem)  →  regression test (red)
    →  fix  →  test green  →  HF doc (the rest)  →  single commit
```

### Trigger

- "hotfix: …"
- "prod is broken"
- "срочно фикс …"

### Flow

1. Skill picks the next `HF-NNN` (parallel sequence, independent from SPEC).
2. Creates `docs/specs/HF-{NNN}-{slug}/hotfix.md` from the hotfix template.
3. Fills §1 Meta + §2 Problem from your words (verbatim — don't paraphrase).
4. **Before touching code:** writes the regression test. It must be red — if it doesn't reproduce the bug, the test isn't testing what you think it's testing.
5. Applies the fix. Test goes green. Existing suites stay green.
6. Fills §3 Timeline through §8 Follow-up.
7. Single commit bundles fix + regression test + HF doc. Commit message references `HF-{NNN}`.

### Never skipped, even in a hotfix

- Regression test (mandatory — no exceptions without a documented reason).
- DoD basics: types, lint, existing suites green.
- Safety rails: no `--no-verify`, no force-push, no mocking the broken dependency to "make tests pass".
- The HF doc itself — "I'll write it later" loses the paper trail exactly when the next incident hits.

### When the hotfix is a band-aid

If the fix patches a symptom but leaves the underlying design flaw, §8 Follow-up names the proper fix — either a new small-spec or a `docs/future-work.md` entry with `Source: HF-NNN`. Don't let band-aids pile up invisibly.

---

## Bug reports during execution

Any track can surface a defect: a test captures intended SPEC behaviour, the production code does the wrong thing, the test fails. **The fix never happens in the same task.** Instead the agent files a bug report and continues. The fix lands later as a hotfix (HF-NNN) or a follow-up SPEC, citing the BUG-NNN.

**The full convention lives in `.claude/skills/spec-development/templates/bug.md` and `bugs-readme.md`.** Project-local copies are seeded in `docs/bugs/` on the first defect filing — `docs/bugs/README.md` (numbering rule, severity scale, lifecycle) and `docs/bugs/_template.md` (per-bug form).

### Filing a bug — quick rules

1. **Litmus test first.** Can you cite the SPEC section the code violates? If yes — file. If no — `docs/future-work.md` or discuss with the orchestrator. This directory is for defects, not missing features or polish.
2. **Numbering.** Sequential project-wide: `max(existing BUG-NNN) + 1`, three-digit zero-padded. First bug is `BUG-001`.
3. **Severity (4 levels).** `critical` (data-loss / security / PHI/PII leak / production incident) · `high` (wrong user-visible behaviour) · `medium` (wrong non-user-visible behaviour) · `low` (polish). When in doubt, escalate.
4. **Skip-with-reference.** The originating test is annotated with the BUG token so a grep reconciles bugs ↔ skips. The exact form by stack:
   - Vitest / Jest: `it.skip("blocked by BUG-NNN — short reason", …)`
   - Go: `t.Skip("blocked by BUG-NNN — short reason")`
   - Pytest: `@pytest.mark.skip(reason="blocked by BUG-NNN — short reason")`
   - JUnit: `@Disabled("blocked by BUG-NNN — short reason")`
   - RSpec: `skip "blocked by BUG-NNN — short reason"`
5. **Lifecycle.** `open → fix-tracked → closed`. Closed bugs stay in the directory as historical record.
6. **Hand-off implication.** Every spec hand-off (small-spec, full-triplet T-FINAL, hotfix close) checks: every `blocked by BUG-` annotation in the diff has a matching `docs/bugs/BUG-NNN-*.md`. A spec is NOT done if the reconciliation fails.

### What NOT to file as a bug

- Missing features → `docs/future-work.md`.
- Non-blocking polish without SPEC backing → `docs/future-work.md`.
- Environment / tooling flakes (test passed locally, failed in CI once) → task hand-off "Additional scenarios", not a bug.
- Things the agent disagrees with that ARE intended behaviour → discuss, don't file unilaterally.

### When the test reveals a defect

The agent does NOT skip the test silently. The test captures **intended** behaviour, gets a `skip-with-reference` annotation pointing at the new BUG-NNN, and stays in the diff. When the fix lands, the skip is removed and the test runs. The test is the bug's regression guard, baked in from day one.

---

## Commands cheat-sheet

### Explicit commands

| Command | Who runs | When |
|---|---|---|
| `/spec-development` | user | invokes the skill to start or update a spec |
| `/code-review` | user or orchestrator | plugin command. Requires a GitHub PR + remote. Spawns multi-agent review of the PR diff |
| `/security-review` | user or orchestrator | plugin command. Requires `origin/HEAD` to resolve (run `git remote set-head origin --auto` if not set). Focused security-only pass |
| `/ultrareview` | **user only** | thorough multi-agent cloud review at epic end — orchestrator will remind you |

### Implicit phrases the agent recognises

**Full-triplet track:**

- "let's write a spec" / "create SPEC for …" → Phase 2 (full triplet)
- "break into tasks" / "decompose this epic" → Phase 4
- "write the plan" / "generate plan" → Phase 5
- "execute" / "run the plan" / "start Wave 1" → Phase 6
- "update SPEC-NNN" → continues an existing spec
- "status of SPEC-NNN" → reads Progress log from plan.md

**Small-spec track:**

- "small spec for …" / "just a tiny change" / "one-file fix for …" → small-spec track

**Hotfix track:**

- "hotfix: …" / "срочно фикс …" / "prod is broken: …" → hotfix track

When the phrasing is ambiguous, the agent asks which track before creating any file.

---

## How to run `/code-review` and `/security-review`

Both are **Claude Code plugin commands** (`claude-code-plugins/code-review` in particular), not skills local to this repo. They have preconditions that the plan generator should check before promising them as gates.

### `/code-review` — requires a GitHub PR

1. The repo must have a GitHub remote (`git remote -v` non-empty) and the current branch must be pushed to it.
2. A PR must exist for the branch (`gh pr view` returns a record). Create with `gh pr create --base main --head <branch>` if it doesn't.
3. Invoke `/code-review <PR#>`. The command spawns multiple sub-agents, collects issues, filters false positives, and either prints the findings or (with `--comment`) posts inline comments on the PR.
4. The command is **billable** — it runs Sonnet/Opus agents in parallel. Don't invoke it on every push; run it once before merge.

### `/security-review` — requires origin/HEAD resolvable

1. Remote must be set, and `origin/HEAD` must resolve (the command uses `git log origin/HEAD...` internally).
2. If `git symbolic-ref refs/remotes/origin/HEAD` errors, run `git remote set-head origin --auto` once to pin it to the origin's default branch.
3. Invoke `/security-review`. Output lands in the terminal; by default the command does not post to GitHub.

### Integration with the spec workflow

- For a full-triplet spec, `plan.md` lists `/code-review` and `/security-review` as wave-end gates. Before the plan is written, verify the plugin is installed and the preconditions are met (remote + PR for code-review; `origin/HEAD` for security-review). If not, wire the plugin OR mark the gate `aspirational — not wired as of YYYY-MM-DD` so nobody relies on a gate that silently no-ops.
- For a small-spec or hotfix, `/code-review` is still appropriate before merging but not mandatory. `/security-review` is appropriate whenever the change touches an auth / session / PII / credential surface.

---

## Review gates — what runs automatically, what you run

### Automatic (orchestrator runs without asking)

| Gate | When | Skill / command |
|---|---|---|
| Code review | end of each wave | `code-review` |
| Security review | wave touches auth / sessions / consent / PII / PHI / payments | `security-review` |
| **E2E suite (isolated stack)** | end of each wave that changes user-visible flow, API contract, schema, or auth/middleware | the project's E2E command against an isolated test stack (see operating rules below) |
| DB change review | wave contains a migration or DDL | the project's DB-review skill if one is installed; otherwise the gate is marked `aspirational` and the wave rules below still apply |
| Architecture review | at epic end, non-trivial epics | prompt-level (dedicated skill to come) |

#### E2E gate — operating rules

> Universal verification rules (computed-style asserts not className, audit-row by storage query, image render asserted by `naturalWidth > 0`, locale/tz/Accept-Language pinning + adversarial flip variant, etc.) live in the skill — see `.claude/skills/spec-development/SKILL.md` § "Verification rigour" and `templates/verification-checklist.md`. The rules below are the **E2E operating constraints** a project layers on top of that universal floor — adapt the specifics (commands, ports, services) to your stack and pin them in the project's `CLAUDE.md`.

The E2E gate exists so a wave merges only after the full surface still works end-to-end, including visual regression. It is a hard gate; if it fails, the wave does not merge.

- **Runs against an isolated test stack only.** The test stack binds its own ports/containers, distinct from the user's dev stack, so the two never collide. The orchestrator brings the test stack up before the gate, runs the suite, and tears it down after.
- **The test stack is per-run.** Bring-up reseeds deterministic fixtures into throwaway storage; tear-down removes everything. Never reuse a stale stack across runs — single-use fixtures (invite tokens, one-time rows) from a previous run cause spurious failures.
- **Suite scope per wave.** The orchestrator picks specs whose surface overlaps the wave's `Scope paths`. A regression set (the always-on smoke specs) runs on every wave regardless. The full suite + visual baselines runs at epic end.
- **Visual baselines.** Each spec that owns a visual surface owns a baseline snapshot committed alongside the spec's tests. Re-baseline only via an explicit update flag and only when the change is intended.
- **Positive AND negative scenarios.** Each new spec must include the happy path AND at least one negative path the spec is meant to defend against (e.g. invalid input, expired token, last-admin guard, self-mutation guard).
- **No mocks of the auth boundary.** Substitute the real auth mechanism only with its official emulator or test double, and exercise real tokens against it. Faking the verifier is forbidden.
- **The dev stack stays alive throughout.** The user verifies UX manually after the orchestrator hands off, against the same code that just passed the test stack. Agents NEVER stop or restart the dev stack.

### User-triggered (orchestrator reminds you at epic end)

| Gate | You run |
|---|---|
| Thorough multi-agent review | `/ultrareview` |
| Manual functional verification | work through the aggregated checklist the orchestrator surfaces |

You do not need to hunt for the manual checklist — it appears in the orchestrator's final hand-off message.

---

## Execution preferences (v1.0)

v1.0 adds three independent ways to tune how a plan runs. They are **opt-in**: a `plan.md` that sets none of them runs exactly as it did in pre-1.0. You set them as optional rows in `plan.md` §1 Meta, or override them per run with invoke flags. The full contract is in the skill (`SKILL.md` § "Execution preferences" / "Review orchestration"); this is the human map.

The three axes are orthogonal — any combination is valid:

- **Engine** — *who runs the tasks.* `task-tool` (default; the pre-1.0 one-agent-per-task model) or `dw` (the Workflow tool, for large, highly parallel epics), or `auto` (the skill picks `dw` only when the work is big and parallel enough — ≥ 8 tasks and a wide-enough widest wave). Choosing `dw` has two honest caveats: it needs the Workflow tool to be available (if not, the skill asks or falls back to parallel agents — never silently), and a Workflow run can't pause mid-flight, so each wave is one Workflow call with you approving between waves.
- **Stop mode** — *when it pauses for you.* `auto` (run straight through), `per-wave` (default; pause after each wave — the old "paused-between-waves"), or `per-task` (pause after every task).
- **Review** — *adversarial review of each task.* When enabled, after a task is built an independent reviewer subagent gets **only** five things — the spec, the task, the final diff, the spec's `verification-checklist.md` including its spec-specific §10+ sections, and the implementer's hand-off — and never the implementer's reasoning or chat history. The hand-off arrives as *claims to refute*, not as evidence: a claim the reviewer cannot verify against the diff is itself a finding. It tries to **refute** the result, returning a structured verdict (PASS / FAIL / NEEDS_REVISION). `fail_action` decides what happens on a non-PASS: `revise` (default — send it back), `halt`, or `flag-only` (record and move on). A hard safety rail: three non-PASS rounds on one task stops the run and asks you to decide — no infinite revision loops.

  *Patterns:* `adversarial` reviews every task. `spot-check` reviews only the tasks that earn it — `Risk: high`, or a task whose `Scope paths` touch a checklist §1–§7 surface (persistence, UI, audit, concurrency, E2E, external API, i18n), or `type: infra` — and writes one line into `review.log.md` for every task it skipped, with the reason. That last part matters: a review mode that silently skips work is indistinguishable from a review that found nothing. `none` disables review and is recorded once in the log header.

**Compliance mode.** Set `Compliance critical: true` on a plan and the skill hardens it: the `dw` engine is blocked (override only with `--allow-dw-on-compliance` and a confirmation), and review is forced on as `adversarial` with `flag-only` disallowed. The forced values are written into `review.log.md` so the audit trail shows why they differ from the plan.

**Invoke flags** (one-run overrides, highest precedence): `--engine`, `--stop`, `--review`, `--allow-dw-on-compliance`. Precedence top-down: flag → `plan.md` rows → project `CLAUDE.md` defaults → skill defaults.

A note on validation: these are instructions to an agent reading prose, not a config parser. The rule is **"don't guess — stop and ask"**: a typo or unknown value in the safety rows (engine, review, compliance) stops the run with a clear message rather than silently falling back, because a silent fallback there could quietly drop a protection.

### How to pick a mode (worked example)

Writing the spec is unchanged — invoke the skill (`/spec-development`, "write a spec for…") and it walks `epic → tasks → plan` as always. Mode selection happens on the **plan**, and affects **execution only**.

**Want it exactly like before (pre-1.0)?** Do nothing. Set no execution rows. You get Task tool, pause-between-waves, no review.

**Want the new behaviour?** Add the rows you care about to `plan.md` §1 Meta — e.g.:

```
| Engine | dw |                ← Workflow tool (or auto / task-tool)
| Stop mode | per-wave |       ← auto / per-wave / per-task
| Review enabled | true |
| Review pattern | adversarial |
```

**Just for one run, without editing the plan?** Override with flags when you kick off execution:

```
execute --engine dw --review adversarial --stop per-wave
```

Precedence (highest first): **flag → `plan.md` rows → project `CLAUDE.md` defaults → skill defaults.**

When you say "execute", the orchestrator confirms the stop mode, and — if you chose `dw` — checks the Workflow tool is available (if not, it asks or falls back to parallel agents, never silently). Then it runs wave by wave; with review on, each task gets a reviewer verdict before the wave merges.

## Working rules (summary)

Full rules are in `.claude/skills/spec-development/SKILL.md`. Highlights:

- **Specs contain no implementation code.** Type declarations, schema formats (OpenAPI, JSON Schema, Protobuf), SQL DDL, and mermaid diagrams are allowed — these are specifications. Function bodies, algorithms-as-code, and test assertions-as-code are not.
- **Each task must be self-sufficient.** A task agent should not need to read sibling tasks to execute. When a cross-task dependency is unavoidable, the task quotes the interface in its References.
- **Tests: core scenarios up-front, additional during implementation.** Core scenarios are written at task creation (3–5 flows mapped to acceptance criteria). Additional scenarios are appended by the executing agent as they discover edge cases. Manual verification is the exception — automate by default.
- **Definition of Done applies globally** (types pass, linter clean, core tests green, additional scenarios documented, i18n covered, a11y passes, manual checklist confirmed, PR references task + epic).
- **Verification rigour is enforced per task, on every track.** Every spec ships with a `verification-checklist.md` — full triplet, small spec, and hotfix alike — carrying the 18 universal anti-hand-wave rules `VR-01`…`VR-18` (persistence by storage query, audit-row by storage grep, concurrency by post-condition row count, computed-style not className, image render asserted, exit-code honesty, no silently-skipped layer, domain failure as non-2xx, …) plus project-/spec-specific surfaces. Three tiers: **§0 binds everywhere, no exceptions**; §1–§7 bind on whichever surfaces the change touches; the full file including §10+ is mandatory for a full triplet and optional for the two small shapes. The one concession to incident pressure: on a hotfix the *negative paired test* may be deferred with a named follow-up — the adjacent-configs sweep may not. See `SKILL.md` § "Verification rigour" and `templates/verification-checklist.md`.
- **Wave-boundary merge.** Tasks in a wave branch from the same base; the wave merges as a unit after all tasks and automated reviews pass.
- **Schema changes are gated, because a wave rollback does not un-drop a column.** A task carrying destructive DDL (`DROP COLUMN`, `DROP TABLE`, type narrowing, `NOT NULL` on a populated column) is kept out of the same wave as anything depending on it, the down-migration is verified before the wave merges, and the destructive change itself needs your explicit confirmation at merge time — naming what data is lost. Additive migrations (new column, backfill, index) wave normally.
- **Compliance is declared where the track can carry it.** `Compliance critical` lives in `plan.md` §1 for a full triplet, and in `spec.md` §1 / `hotfix.md` §1 for the two shapes that have no plan. A compliance label in the project's `CLAUDE.md` covers every track and cannot be lowered by a per-spec row — a spec may raise compliance, never waive it.
- **Failure policy.** One task blocking mid-wave does not halt the wave. Dependent tasks defer to a later wave. If the blocker invalidates the epic, the orchestrator proposes a re-plan.

---

## Tips and edge cases

- **You can pause an epic and return later.** Leave open items in §22 Open questions. Say "continue SPEC-NNN" in a later session.
- **Drafting an epic often surfaces gaps you did not see in brainstorming.** ("Which region ships first?", "What if consent is revoked mid-test?") That is working as intended — back to Phase 1 for those points, then continue.
- **For very small specs**, skip Phase 1. Tell the skill directly: "single-task spec to fix X". Skill will propose a minimal epic with only the relevant sections.
- **For very large epics**, brainstorm (Phase 1) may span multiple sessions. Keep free-form notes; when ready, ask "turn these notes into an epic".
- **Re-planning is cheap.** If Wave 1 results change your assumptions, update the epic, regenerate affected tasks, recompute the plan. Do it — don't push through a stale plan.
- **Do not skip the Review phase.** An epic approved in a rush produces tasks that wander, which produces rework. The skill enforces this by refusing to decompose a draft epic.

---

## Project-specific context

This workflow applies to any project that hosts the `spec-development` skill. Project-specific conventions (tech stack, brand rules, compliance requirements, monorepo layout, source materials) live in the project's `CLAUDE.md` and are read by the agent alongside the skill.
