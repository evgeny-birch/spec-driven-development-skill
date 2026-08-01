# spec-development

A [Claude Code](https://claude.com/claude-code) skill that walks you and the agent through writing, decomposing, and executing product specifications — in three shapes that match the size of the work.

> Brainstorm → epic → tasks → plan → execute, with the AI as a co-author at every phase.

## What you get

Three tracks the skill picks between, all living under `docs/specs/`:

| Work | Track | Artefact |
|---|---|---|
| New feature, multi-surface, partner coordination, new data model / API | **Full triplet** | `epic.md` + `tasks.md` + `plan.md` |
| Single well-understood change, 1–3 files, low risk, one sitting | **Small spec** | `spec.md` |
| Production / critical surface broken, fix needed now | **Hotfix** | `hotfix.md` (alongside the fix) |

Plus a bug-report template and a deterministic `SPEC-NNN` / `HF-NNN` / `BUG-NNN` numbering scheme so artefacts stay greppable.

The full process is documented in [`docs/workflow.md`](docs/workflow.md).

## Install

There are **two pieces** to install — a skill (for the AI agent) and a workflow document (for humans, also referenced by the agent through `CLAUDE.md`):

```
skills/spec-development/   →  ~/.claude/skills/  (or  <project>/.claude/skills/)
docs/workflow.md           →  <project>/docs/workflow.md
```

### 1. The skill — operational instructions for the agent

Drop it into either Claude Code skills location:

**Globally** (available in every project on your machine):
```bash
cp -r skills/spec-development ~/.claude/skills/
```

**Per-project** (vendored into a single repo):
```bash
cp -r skills/spec-development .claude/skills/
```

The directory name `spec-development` matches the skill's `name` in frontmatter — Claude Code uses that to register it. Don't rename the folder.

### 2. The workflow doc — process map for humans (and the agent through `CLAUDE.md`)

Copy `docs/workflow.md` into the project where you'll use the skill:

```bash
mkdir -p your-project/docs && cp docs/workflow.md your-project/docs/
```

Then add a one-line reference in your project's `CLAUDE.md` so the agent picks it up alongside the skill:

```markdown
## Workflow

See [`docs/workflow.md`](./docs/workflow.md) for how we plan and ship work (three tracks, six phases, gates).
```

Why two artefacts: the skill is the *contract with the agent* — terse, mechanical, scoped to the act of writing a spec. `workflow.md` is the *contract with humans* — narrative, explains the why, the phase boundaries, what shows up in the repo. Both are useful, neither is a copy of the other.

You can skip step 2 if you're a solo user who already has the workflow in your head — but for any team, the doc is what onboards a new contributor in five minutes.

## How to invoke

Once installed, any of these triggers the skill:

- `/spec-development` — explicit slash command
- "let's write a spec for this"
- "create SPEC for …"
- "break this epic into tasks"
- "update SPEC-042"
- "small spec for …" / "one-file fix for …"
- "hotfix: prod is broken"

When the phrasing is ambiguous between tracks, the skill asks before creating any file.

## v1.0 — execution preferences (optional, backward-compatible)

v1.0 adds three orthogonal, opt-in ways to tune how a plan executes. **A `plan.md` that sets none of them runs exactly as pre-1.0** — nothing to migrate. Set them as optional rows in `plan.md` §1 Meta, or override per run with flags.

- **Engine** — `task-tool` (default, the pre-1.0 model) · `dw` (the Workflow tool, for large parallel epics) · `auto`.
- **Stop mode** — `auto` · `per-wave` (default) · `per-task`.
- **Adversarial review** — an independent reviewer subagent gets only the spec + task + final diff (never the implementer's reasoning) and tries to *refute* the result, returning a structured PASS / FAIL / NEEDS_REVISION verdict. A three-strikes rail stops infinite revision loops.

Set `Compliance critical: true` to block the `dw` engine and force adversarial review on. Invoke flags (`--engine`, `--stop`, `--review`, `--allow-dw-on-compliance`) override per run. Full contract in the skill's `SKILL.md` and the human map in [`docs/workflow.md`](docs/workflow.md).

## v1.1 — contract integrity

v1.1 fixes a class of defect an audit of v1.0 exposed: **rules the skill declared but never delivered to the agent expected to apply them.** A rule an agent cannot read is worse than no rule — it looks like a gate and isn't one.

- **The reviewer now receives what it judges by.** It was told to map findings to the spec's `verification-checklist.md` and to four named rules, and was handed neither. It now gets five inputs — spec, task, diff, the checklist *including its spec-specific §10+ sections*, and the implementer's hand-off — with the rule text written into its own instruction file.
- **Waivers are reviewable.** An agent could close any rigour rule with "waived because…" and nothing checked it. New finding category `waiver-abuse`: every waiver and every `n/a` is verified against the diff. The hand-off reaches the reviewer as *claims to refute*, never as evidence.
- **One identifier per rule.** `VR-01`…`VR-18` replaces the old dual naming (`HR-1`..`HR-4` survive as aliases; nothing to migrate). The hand-off template reports all 18 — it used to report 12, silently omitting three mandatory ones.
- **The rigour floor binds on all three tracks**, not just the full triplet. `§0` is universal; on a hotfix exactly one item is deferrable (the negative paired test, with a named follow-up).
- **Requirements get an audit trail.** `epic.md` §26 Change log: what changed, which tasks it invalidated, why, who approved. A re-plan bumps the version and writes the row *before* regenerating tasks.
- **`spot-check` finally does something.** It reviews tasks that are `Risk: high`, touch a §1–§7 surface, or are `type: infra` — and logs every skip with a reason.
- **Destructive DDL is gated at the wave boundary**, because reverting a wave does not un-drop a column.
- **One writer for the progress log, one source of truth for status.** Divergence stops and asks instead of being silently resolved.
- **Brownfield recon (Phase 0)** for work in code you didn't write, and a mechanical completeness self-check on the epic before decomposition — its two literal checks ship as commands whose output gets pasted, the three semantic ones stay a read.

Upgrading an installed copy: `~/.claude/skills/spec-development/` is a **copy**, not a link — re-run the install command after pulling, or symlink it to the repo, or the agent keeps reading the old contract.

## What lands in your repo

After a full-triplet run:

```
docs/
├── specs/
│   └── SPEC-001-your-slug/
│       ├── epic.md                   ← what & why
│       ├── tasks.md                  ← work breakdown
│       ├── plan.md                   ← execution waves + dependency graph
│       └── verification-checklist.md ← per-spec anti-hand-wave hand-off contract
└── bugs/                   ← seeded on first defect filing
    ├── README.md
    └── BUG-001-...md
```

Small specs collapse to a single `spec.md` (plus `verification-checklist.md`); hotfixes use `HF-NNN-{slug}/hotfix.md` on a parallel sequence. When review is enabled (v1.0), a run also writes a `review.log.md` alongside the plan.

## When NOT to use this

- Throwaway scripts, one-line fixes, prototype throwaway code — the ceremony exceeds the work.
- Teams that already have a heavyweight RFC/PRD process — don't bolt this on top, pick one.
- Solo projects where you trust yourself to hold the plan in your head — this skill earns its keep when the agent needs structured context, or when humans need to review intent before code.

## Project context

Projects often have stack-specific or compliance-specific conventions (i18n, a11y, brand rules, regulatory boundaries). Those belong in your project's `CLAUDE.md`, not in this skill. The skill reads both.

## Contributing

Issues and PRs welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

MIT — see [`LICENSE`](LICENSE).
