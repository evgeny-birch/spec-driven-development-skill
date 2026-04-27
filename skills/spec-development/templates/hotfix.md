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
