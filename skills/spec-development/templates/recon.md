# SPEC-{NNN} — Recon (brownfield Phase 0)

> **How to use this template**
>
> - Copy to `docs/specs/SPEC-{NNN}-{slug}/recon.md` **before** the epic, whenever the work is brownfield: a change to existing code the author did not write, or does not already hold in their head.
> - This is reconnaissance, not design. It records **what is there**, not what you intend to do. Intent belongs in the epic.
> - **Citation rule:** every claim names a concrete path, symbol, file, or command output. A section with no citation counts as **unfilled** — an uncited recon is a paraphrase of the epic and worse than nothing, because it reads like evidence.
> - Keep it short. Five sections, a handful of lines each. If a section runs long, you are designing, not scouting.
> - Delete this block when the file is filled in.

**Spec:** SPEC-{NNN} · **Written:** YYYY-MM-DD · **Commit scouted:** `{sha}`

Recording the commit matters: a recon is a snapshot, and the epic that cites it is only as current as this line.

---

## 1. What exists today on the affected surface

The code that is already doing this job, or the nearest thing to it. Entry points, the modules involved, where the data lives.

- `path/to/file.ext:NN` — {what it does, in one line}
- `path/to/other` — {…}
- **Nearest existing analogue:** {the closest thing to what we are about to build, and how it works} — `path`

## 2. Conventions and constraints in force

What the codebase already decided, that this work must respect or deliberately break. Read from the code, not from documentation that may be stale.

- **Naming / layout:** {observed pattern} — e.g. `path/a`, `path/b` both follow it
- **Error handling:** {observed pattern} — `path:NN`
- **Testing:** {framework, where tests live, how they run} — `path`, `{command}`
- **Documented vs actual:** {anything where the docs and the code disagree — name both} — doc `path`, code `path:NN`

## 3. Integration points

Everything that will call our change, or that our change will call. This is the list that determines the blast radius.

| Direction | Counterpart | Where | Contract |
|---|---|---|---|
| calls us | {module / service / job} | `path:NN` | {shape / signature / event} |
| we call | {module / service / API} | `path:NN` | {shape} |

## 4. What must not be touched, and why

The load-bearing walls. Each entry states the reason, because "don't touch" without a reason gets overridden by the next agent.

- `path` — {why: public contract with N consumers / regulatory / undocumented behaviour something depends on / no test coverage to catch a break}
- {…}

## 5. What is missing, contrary to expectation

The most valuable section, and the one that is hardest to write. Things a newcomer would reasonably assume exist and that do not — no migration runner, no test for the critical path, no validation on the boundary, an env var read but never documented.

- **Expected:** {what you assumed existed} · **Actual:** {what is there instead} · **Evidence:** `{grep / path}`
- {…}

If this section is empty, say so explicitly and name what you checked — an empty section reads as "not looked at" otherwise.

---

## How the epic uses this file

`epic.md` §4 "Prior art" and §11 Architecture **reference this file** rather than restating it. Restating creates two copies that drift; the epic cites `recon.md §N` and moves on.
