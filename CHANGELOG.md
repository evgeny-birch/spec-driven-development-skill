# Changelog

All notable changes to the `spec-development` skill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
