# HF-{NNN} — {Title}

> **How to use this template**
>
> - This is the **hotfix** shape. Use it when production (or a critical dev-loop surface) is broken, the user can describe the problem, and the fix is needed now.
> - The doc is **written alongside the fix**, not before it. Fill sections 1–2 (Meta, Problem) BEFORE touching code, so the paper trail exists at triage. Fill sections 3–8 AFTER the fix is green and tested, BEFORE the single commit that bundles fix + test + doc.
> - Skip nothing that is safety-critical: no `--no-verify`, no force-push, no mock of the broken dependency to "make it pass". A regression test is mandatory — an urgent fix without a guard is how the same bug recurs.
> - Delete these instructions when the file is filled in.

---

## 1. Meta

| Field | Value |
|---|---|
| ID | HF-{NNN} |
| Title | {short} |
| Status | in-progress / fixed |
| Opened | YYYY-MM-DD HH:MM |
| Closed | YYYY-MM-DD HH:MM |
| Owner | {name} |
| Severity | sev-1 / sev-2 / sev-3 |
| Scope | {what surface is broken — page, endpoint, cron, migration…} |
| Compliance critical | `true` / `false` — default `false`. No `plan.md` on this track, so compliance is declared here. A project `CLAUDE.md` compliance label overrides `false`. |

Fill this at triage, before coding.

## 2. Problem

The user's words verbatim. Quote them — don't paraphrase. Add surface details only the agent can observe (URL, status code, log line, stack trace).

> "..."

Fill this at triage, before coding.

---

Sections below are filled AFTER the fix is green, BEFORE committing.

## 3. Timeline

- **Noticed:** YYYY-MM-DD HH:MM — by {whom}, via {channel}
- **Triaged:** YYYY-MM-DD HH:MM — hotfix opened
- **Fix committed:** YYYY-MM-DD HH:MM
- **Verified on {environment}:** YYYY-MM-DD HH:MM

## 4. Root cause

What was actually wrong. One paragraph. If you ran out of time to fully diagnose and the fix is a band-aid, say so explicitly — and list a follow-up in §8.

## 5. Fix

Files changed, one line per file. The commit reference lands below in §6.

| File | Change |
|---|---|
| `path/to/file` | ... |

## 5b. Documentation impact

Every architecture / project doc this hotfix touches goes here BEFORE the commit, not after. Enumerate up-front so partial doc coverage can't slip through. Refer to the project's CLAUDE.md for the area→section mapping. If the hotfix is purely an internal mechanism with no doc surface, state `none — internal mechanism only`.

| Doc | Why HF-{NNN} touches it | Updated |
|---|---|---|
| `docs/architecture/0X-...` | {reason: behaviour change in this subsystem; specific lines that became wrong} | [ ] |
| `docs/architecture/README.md` | "As of" bump + one-line HF entry at the top of the intro paragraph | [ ] |
| `docs/prod-readiness.md` | only if the fix introduces a new pre-prod gate (rare for hotfixes) | [ ] |
| `docs/future-work.md` | only if §8 defers something with `Source: HF-{NNN}` | [ ] |

Sweep the existing docs for stale claims about the broken behaviour — grep for the symptom keywords (e.g. job_id, key name, endpoint path). Failure modes tables and sequence diagrams routinely encode the OLD behaviour and become misleading after the fix.

## 6. Regression guard

Test added so this can't recur silently.

- **Path:** `tests/.../foo.spec.ts`
- **Level:** unit / integration / E2E
- **Asserts:** {what the test checks}

If a regression test is NOT feasible (legitimate cases: compiler bug upstream, third-party API defect), state why, and add an entry to `docs/future-work.md` with a plan to add coverage when it becomes feasible.

## 7. Verification

- [ ] Regression test added and green
- [ ] Existing automated suites (unit + integration + E2E) still green
- [ ] Fix manually verified on the broken surface
- [ ] For UI hotfixes: browser walkthrough of the affected flow
- [ ] For API hotfixes: the exact request from §2 now returns the expected response

The rigour floor binds on this track too — `./verification-checklist.md` §0, in full:

- [ ] **`VR-15` exit-code honesty** — the green was read from each step's own result, not from a trailing `grep`/`tail` or a backgrounded "exit 0". Under time pressure this is the first thing that slips.
- [ ] **`VR-16` no silently-skipped layer** — nothing self-skipped without a `BUG-NNN` reference; a layer that cannot run here is named in §8, not dropped.
- [ ] **`VR-18` domain failure surfaces as non-2xx** — if the fix touches an API surface.
- [ ] **§1 persistence items applied** — if the fix touches storage: existence asserted by a storage query, state transition by a post-condition read, assertions scoped to rows the test created (`VR-17`).
- [ ] **Adjacent-configs sweep** — grep `Makefile`, CI workflows, compose files, `.env.example`, README, scripts for anything the fix renamed. **Not deferrable, even here:** it is a grep, and a rushed fix that changes one env var and misses four other places is the classic hotfix regression.
- [ ] **Negative paired test** — the one §0 item that may be deferred under incident pressure. If deferred, name the follow-up: `deferred → FW-NN / BUG-NNN`. Anything else in §0 is not deferrable.

## 8. Follow-up

If the hotfix is complete (root-caused + permanent), leave this section as `none — fix is root-cause`.

If the hotfix is a band-aid (symptom patched, underlying design flaw remains), list what's left and where it's tracked:

- **Proper fix:** open SPEC-NNN — {reason, rough scope}
- **Deferred work:** append FW-NN to `docs/future-work.md` with `Source: HF-{NNN}`
- **Prod-readiness gate:** append to `docs/prod-readiness.md` if promotion requires anything new

## 9. Commit

Single commit bundles:

- The fix (files from §5)
- The regression test (§6)
- This document

Commit message references `HF-{NNN}` in the subject or body. Example:

```
{short fix subject} — HF-{NNN}

{body}

Co-Authored-By: ...
```
