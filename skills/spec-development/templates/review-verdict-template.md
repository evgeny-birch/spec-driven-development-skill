# Review verdict entry

> **How to use this file**
>
> - One entry per task review round, appended to `review.log.md`.
> - `verdict`, `severity`, and `category` are **fixed enums** — keep them exact so the log stays greppable (see `review-log-template.md`).
> - `findings` is empty only when `verdict: PASS`. A `PASS` with findings is a contradiction; a `FAIL`/`NEEDS_REVISION` with no findings is invalid.
> - Delete this block when adapting; keep the field structure.

## {task_id}: {task_title}
implementer: {subagent_id_or_label}
reviewer: {subagent_id_or_label}
round: {n}
verdict: {PASS | FAIL | NEEDS_REVISION}
findings:
  - severity: {critical | major | minor}
    category: {spec-deviation | silent-failure | acceptance-miss | quality | test-gap | waiver-abuse}
    description: {what is wrong, tied to a spec clause / acceptance criterion / rigour rule}
    suggested_action: {revert | patch | re-implement}
  - severity: {…}
    category: {…}
    description: {…}
    suggested_action: {…}

<!--
Enum reference:
  verdict   : PASS | FAIL | NEEDS_REVISION
  severity  : critical | major | minor
  category  : spec-deviation | silent-failure | acceptance-miss | quality | test-gap | waiver-abuse
  action    : revert | patch | re-implement
PASS  → findings empty.
FAIL  → re-implementation warranted (critical findings).
NEEDS_REVISION → addressable by patches.
-->
