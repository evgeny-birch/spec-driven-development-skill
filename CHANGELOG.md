# Changelog

All notable changes to the `spec-development` skill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] — 2026-08-01

Contract-integrity release. An audit of 1.0 read the skill as a *system of cross-references* for the first time and found rules that were declared in one file, defined in a second, and enforced by an agent that received neither. **No existing `epic.md` / `spec.md` / `tasks.md` / `plan.md` becomes invalid**; new sections are additive and the one rename keeps its aliases.

### Fixed
- **The adversarial reviewer was starving.** It was instructed to map findings to the spec's `verification-checklist.md` MANDATORY items and to rules `HR-1..HR-4` in `SKILL.md` — while `SKILL.md` is explicitly not loaded for it and the checklist was not among its inputs. It has been judging by rules it could not read since 1.0. Its input list is now five items (spec · task block · diff · the checklist **including §10+** · the implementer's hand-off), `reviewer-instructions.md` is the normative source of that list, and the text of `VR-15`…`VR-18` is written into that file so nothing points at an unavailable one.
- **The hand-off template reported 12 of 18 rigour rules**, silently omitting three MANDATORY items from §0 of the same file. It now reports all 18, one line each.
- **`MANDATORY` was simultaneously un-waivable and waivable** — §"How to use" forbade waiving it, the §8 template offered `⚠️ waived` on every rule. The `waived` option now appears only where waiving is legal; a waived MANDATORY item is a `critical` finding.
- **The anti-hand-wave floor only bound the full-triplet track**, and `SKILL.md` copied `verification-checklist.md` into the spec directory *only* on that track — so on the other two the file the agent was told to honour might not exist. Both fixed.
- **Up to three task agents were told to write one Markdown table** in `plan.md` §11 with no owner declared. Task agents now *return* their row; the orchestrator is the sole writer.
- **`spot-check` was an undefined value inside a safety enum** and `Risk` was a field whose only consumer was that undefined mode. Both now have defined behaviour — and an undefined value in a safety field is more dangerous than a missing one, because the agent silently substitutes its own reading.
- **Five templates pointed at `docs/future-work.md` and `docs/prod-readiness.md`**, which the skill had no way to create. Both now have templates and a lazy-creation rule modelled on `docs/bugs/`.
- **`verification-checklist.md` §9 gated only §0–§7** before a wave merged — skipping §10+, the only part of the file specific to that spec.
- `templates/tasks.md` linked `../verification-checklist.md`, correct only for the split `tasks/` layout and broken in the default inline one.
- `docs/workflow.md` claimed "the 12 universal anti-hand-wave rules" (there are 18) and `SKILL.md` claimed the hand-off maps "rules 1–4" (the template said 1–12) — three sources, three numbers.

### Added
- **`waiver-abuse` finding category** — the only place an agent may legally declare a rigour rule inapplicable now has a contour of refutation. Categories: `spec-deviation` · `silent-failure` · `acceptance-miss` · `quality` · `test-gap` · `waiver-abuse`.
- **Claims framing for the reviewer** — read the diff and reach a verdict *before* reading the hand-off; treat it as assertions to disprove; a claim not independently verifiable from the diff is itself a finding of severity `major` or higher.
- **`epic.md` §26 Change log / `small-spec.md` §10** — an audit trail for the requirements, the counterpart to `review.log.md` for the reviews. A re-plan bumps `Version` and writes the row before regenerating tasks; under `Compliance critical: true` a missing row is a halt.
- **Destructive-DDL gate** — a task carrying `DROP COLUMN` / `DROP TABLE` / type narrowing / `NOT NULL`-on-populated does not share a wave with its dependents, its down-migration is verified before merge, and the change needs explicit confirmation *at merge time* naming what data is lost. A `DB change review` gate row invokes the project's DB-review skill **if one is installed** and is marked `aspirational` otherwise — no specific skill is named anywhere (the 1.0.1 portability precedent).
- **Epic completeness self-check** at `draft` → `in-review` — mechanical only: uncovered requirements, criteria with no observable outcome, leftover placeholder text, surfaces with no §10+ section, empty Out-of-scope. Reported as questions, blocks nothing outside compliance mode. Two of the five checks look for literal strings and **ship as commands whose real output must be pasted**, not as prose for the agent to interpret; the other three are semantic (does this criterion actually cover that requirement?) and stay a read — a parser can only check that identifiers co-occur, which is a different question.
- **Brownfield recon (Phase 0)** — `templates/recon.md`, written before the epic when the work touches code the author did not write. Every claim cites a path or symbol; an uncited section counts as unfilled.
- **Status ownership** — `epic.md` §1 (or `spec.md` §1) is the single source of truth; the statuses in `tasks.md` / `plan.md` are derived. Divergence stops and asks rather than being silently aligned.
- **Compliance declaration site for plan-less tracks** — `spec.md` §1 / `hotfix.md` §1. A `CLAUDE.md` compliance label covers every track and cannot be lowered by a per-spec row.
- New templates: `future-work.md`, `prod-readiness.md`, `recon.md`.

### Changed
- Verification-rigour rules carry a single identifier `VR-01`…`VR-18`. `HR-1`..`HR-4` remain as one alias line — existing specs referencing them still resolve.
- `SKILL.md` no longer restates the nine hand-off slots; the §8 template is the single source. Net growth across the whole release: **+59 lines** (525 → 584), inside a hard 60-line budget — every rule added to a prose contract lowers adherence to the rest, so additions had to pay for themselves by deleting duplication.

## [1.0.1] — 2026-07-23

### Fixed
- **Removed stack-specific coupling from the E2E gate.** The "E2E gate — operating rules" block in `docs/workflow.md` was hardcoded to one project's stack (fixed ports `:3001`/`:18081`/`:55434`/`:9099`, Firebase Auth Emulator + Admin SDK, `make e2e-stack-up`/`-down`, `pnpm exec playwright test`, a concrete snapshot path). All of it is now expressed as universal principles — isolated per-run test stack, dev stack untouched, positive + negative scenarios, no mocking the auth boundary, committed visual baselines — with the concrete commands/ports/services delegated to the project's `CLAUDE.md`. The skill is now portable to any stack.
- Dropped the `**Pebble-specific**` label and genericised the repro-step example in `bug.md` (`make e2e-stack-up` → "start the test stack"). Tool names that remain (Playwright, Vitest, axe-core, Postgres, …) are worded as cross-ecosystem examples, not requirements.

## [1.0.0] — 2026-05-29

First release at a major version (prior tags were pre-1.0 `v0.0.x`). Three orthogonal, opt-in execution mechanisms on top of the unchanged pre-1.0 core. **Fully backward-compatible: a `plan.md` with no execution rows behaves exactly as before — nothing to migrate.**

### Added
- **Adversarial review.** After a task is implemented, an independent reviewer subagent receives *only* the spec + task + final diff (never the implementer's reasoning, chat history, or prior rounds) and tries to *refute* the result, emitting a structured `PASS` / `FAIL` / `NEEDS_REVISION` verdict. The reviewer loads `reviewer-instructions.md`, not `SKILL.md`.
- **Execution-engine choice.** `task-tool` (default) / `dw` (the Workflow tool, for large parallel epics) / `auto` (dispatch by task count + parallel ratio). Under `dw`: one wave = one Workflow call, with the orchestrator owning pauses between waves.
- **Explicit stop modes.** `auto` / `per-wave` (default) / `per-task`, generalising the previous "Execution mode".
- **Compliance gate.** `Compliance critical: true` blocks the `dw` engine (override only via `--allow-dw-on-compliance` + confirmation) and forces adversarial review on.
- **Three-strikes safety rail.** Three consecutive non-PASS verdicts on one task hard-halts the run for a human decision.
- **Invoke flags:** `--engine`, `--stop`, `--review`, `--allow-dw-on-compliance`.
- New templates: `reviewer-instructions.md`, `review-log-template.md`, `review-verdict-template.md`.
- Four verification-rigour rules (HR-1..HR-4): exit-code honesty, no un-gated/silently-skipped layers, scope assertions to your own rows, domain failure must be non-2xx.

### Changed
- `plan.md` template: optional execution-preference rows in §1 Meta (Markdown table — no YAML), defaults annotated.
- `tasks.md` template: optional per-task `Risk` field; the Acceptance section feeds the reviewer.
- `SKILL.md` and `docs/workflow.md`: new "Execution preferences" and "Review orchestration" sections, kept terse to bound instruction dilution.

### Fixed
- `README.md` no longer omits `verification-checklist.md` from the "What lands in your repo" listing (it has shipped in every spec directory since the verification-rigour release).

## Prior — tags `v0.0.1` … `v0.0.3`

Pre-1.0 iterations (this is the first tag at a major version):

- Initial public release: three tracks (full triplet / small spec / hotfix), `SPEC/HF/BUG-NNN` numbering, bug-report convention.
- Verification-rigour rules (1–14) and the per-spec `verification-checklist.md`.
- Documentation-impact sections and infra/idempotence verification rules.

<!-- See git history for the detailed pre-1.0 commit log. -->
