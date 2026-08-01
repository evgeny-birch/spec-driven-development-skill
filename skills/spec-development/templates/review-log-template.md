# Review log — {SPEC-NNN}

> **How to use this file**
>
> - Copy this header to `docs/specs/SPEC-NNN-{slug}/review.log.md` at the start of an execution run that has review enabled.
> - The header records the **resolved** execution preferences (after precedence + any compliance overrides), so the audit trail explains why values may differ from `plan.md`.
> - One `review-verdict-template.md` entry is appended per task review (per round).
> - The field lines below are a **stable, machine-greppable contract** — fixed labels, lower-snake-case keys, verdict/severity/category as enums. Do not rename keys; a future cross-spec aggregator greps them.
> - Delete this block when the file is filled in.

run_started: {ISO-8601 timestamp}
engine: {task-tool | dw}
stop_mode: {auto | per-wave | per-task}
review_pattern: {adversarial | spot-check | none}
compliance_critical: {true | false}
compliance_overrides_applied: {none | e.g. "review forced enabled+adversarial; flag-only rejected; dw blocked — compliance_critical=true"}

---

<!-- Append one review-verdict-template.md entry per task review below this line. -->

<!--
Under review_pattern: spot-check, every task that was NOT reviewed gets exactly one skip line,
in this shape, so a reader can tell "not reviewed" from "reviewed and clean":

skipped: {task_id} — reason: {Risk: low | no §1–§7 surface overlap | …}

Keep the `skipped:` prefix and the ` — reason: ` separator stable: an aggregator greps them,
and it must be able to distinguish a skip line from a verdict entry.
-->
