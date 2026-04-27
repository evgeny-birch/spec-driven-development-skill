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

## What lands in your repo

After a full-triplet run:

```
docs/
├── specs/
│   └── SPEC-001-your-slug/
│       ├── epic.md         ← what & why
│       ├── tasks.md        ← work breakdown
│       └── plan.md         ← execution waves + dependency graph
└── bugs/                   ← seeded on first defect filing
    ├── README.md
    └── BUG-001-...md
```

Small specs collapse to a single `spec.md`; hotfixes use `HF-NNN-{slug}/hotfix.md` on a parallel sequence.

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
