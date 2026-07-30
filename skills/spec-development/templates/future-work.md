# Future work

> **How to use this file**
>
> - Copy this template to `docs/future-work.md` the first time any spec, hotfix, or bug report defers something to it.
> - This is the register for **deferred work that is not a defect**: missing features, polish, ideas surfaced mid-execution, band-aid follow-ups. Defects go to `docs/bugs/BUG-NNN-*.md` instead — the litmus test is whether you can cite the SPEC clause the code violates.
> - It is a queue, not a graveyard. The next planning round reads this file; an item that nobody picks up for several rounds should be closed as `wont-do` with a reason rather than left to rot.
> - Delete this block when the file is filled in.

## Numbering

`FW-NN`, two-digit zero-padded, sequential across the whole project: `max(existing FW-NN) + 1`. First entry is `FW-01`. Numbers are never reused, including for `done` and `wont-do` items — a closed number stays claimed so old references keep resolving.

## Status values

| Status | Meaning |
|---|---|
| `open` | in the queue, not yet scheduled |
| `scheduled` | a SPEC / HF has been opened for it — name it in the row |
| `done` | shipped; the row stays as historical record |
| `wont-do` | deliberately dropped; the reason is mandatory |

## Register

Most recent at the bottom. `Source` is where the item came from — a spec, hotfix, or bug — so the context is one click away.

| ID | Source | Description | Proposed track | Status |
|---|---|---|---|---|
| FW-01 | SPEC-NNN §21 | {one line: what is deferred and why it is worth doing later} | small-spec / full triplet / hotfix / none | open |
| FW-02 | HF-NNN §8 | {band-aid follow-up: what the proper fix would be} | full triplet | open |
| FW-03 | BUG-NNN | {coverage gap that could not be closed while the bug is open} | small-spec | open |

## What does NOT belong here

- **Defects** — production code contradicting a SPEC clause. File `docs/bugs/BUG-NNN-*.md`.
- **Pre-production human steps** — console settings, DNS, partner review, credential provisioning. Those go to `docs/prod-readiness.md`.
- **Open questions inside an active spec** — keep them in that spec's Open questions section until it closes.
