# SPEC-{NNN} — {Title}

> **How to use this template**
>
> - This is the **small-spec** shape: one file, no triplet. Use it when the work is a single well-understood change — one to three files of impact, low risk, executable in a single sitting.
> - If the work grows (new API surface, cross-cutting risk, partner coordination), **stop and convert to the full triplet** (`epic.md` + `tasks.md` + `plan.md`). Don't let a small-spec balloon.
> - Delete these instructions when the file is filled in.

---

## 1. Meta

| Field | Value |
|---|---|
| ID | SPEC-{NNN} |
| Title | {short title} |
| Track | small-spec |
| Status | draft / approved |
| Compliance critical | `true` / `false` — default `false`. This track has no `plan.md`, so compliance is declared here. A project `CLAUDE.md` compliance label overrides `false` and cannot be lowered by this row. |
| Created | YYYY-MM-DD |
| Last updated | YYYY-MM-DD |
| Owner | {name} |

Status flow: `draft` → `approved`. No `in-review` step — small scope doesn't warrant a partner gate.

## 2. Summary

One paragraph. What are we changing and why — enough context that an agent or a future reader understands the motivation without hunting through other files.

## 3. Goal

One sentence. A verifiable outcome: "After this change, {observable thing} happens / stops happening / behaves as {X}".

## 4. Scope

What's in and what's deliberately out, to keep the change from drifting.

**In scope:**

- ...

**Out of scope:**

- ...

## 5. Acceptance criteria

Testable conditions. Given-When-Then when behavioural; plain checklist when structural.

- **AC-1** — ...
- **AC-2** — ...

## 6. Implementation notes

Concrete technical information to execute. **No implementation code** — type declarations, schema formats (OpenAPI, JSON Schema, SQL DDL), and mermaid diagrams are allowed; function bodies and algorithms-as-code are not.

### Files to create or modify

| Path | Action | Purpose |
|---|---|---|
| `apps/web/...` | modify | ... |

### Interfaces / contracts (if any)

Prose or structured. Describe purpose, inputs, outputs, errors. No handler implementation.

### Dependencies / env vars

- ...

### Documentation impact

List every documentation surface this change must update in the same PR. Fill at draft time. One line per entry: file path + reason. Refer to the project's CLAUDE.md for the area→section mapping. If nothing is touched (truly internal refactor with zero observable surface), state `none — fully internal change`.

- `docs/architecture/0X-...` — {one-line reason}
- `docs/architecture/README.md` — "As of" bump + entry
- `docs/prod-readiness.md` — only if this introduces a pre-prod human step
- `docs/future-work.md` — only if §9 cross-spec defers anything with intent to do later

## 7. Tests

### Strategy

Unit / integration / E2E / manual — what runs at which level and why. Default: automate as much as the toolchain allows.

For E2E: if the feature can key on browser-supplied signals (`Accept-Language`, locale, timezone, viewport), pin them in the runner config AND add an adversarial variant that flips the signal.

### Core scenarios

Must-pass scenarios derived from acceptance criteria. Keep it to 3–5.

- **S-1** — Given / When / Then — covers AC-1
- **S-2** — ...

### Additional scenarios (filled during implementation)

Edge cases discovered while implementing.

- (empty — agent fills)

### Manual verification (run at task end)

Only steps automation cannot reliably cover. For UI tasks, a browser walkthrough of the primary flow is the default — automated green only proves mechanics, not UX. If truly fully automated, state `none — fully automated`.

- [ ] ...

## 7.5 Verification rigour gate

Before marking this spec complete, walk `./verification-checklist.md` (copied into this directory when the spec was created). The mandatory subset on this track:

- **§0 Universal — all of it, no exceptions.** Type-check, lint, existing suites green, core scenarios automated, negative paired tests, adjacent-configs sweep, and the three MANDATORY items `VR-15` (exit-code honesty — the green was read per step, not from a trailing command), `VR-16` (no silently-skipped layer), `VR-18` (domain failure surfaces as non-2xx).
- **§1–§7** — whichever surface families this change touches (persistence, UI, audit, concurrency, E2E, external API, i18n). An untouched surface is reported `n/a — <why>`, not omitted.
- **§10+** — optional on this track, but if the change touches a surface the universal floor does not cover, add it.

The hand-off uses the checklist's §8 template, which reports all 18 rules. MANDATORY items cannot be waived; see the waiver rules in §8.

## 8. Open questions

- ...

## 9. Follow-ups / cross-spec

If this change surfaces anything that belongs in the cross-spec files, list it here with the intended destination.

- **prod-readiness.md:** {item to append, reason}
- **future-work.md:** {deferred item, proposed FW-NN slot}

## 10. Change log

Rows added only when this spec changes after it was approved. Keep it to one line per change — the point is the trail, not the prose.

| Date | What changed | Reason | Approved by |
|---|---|---|---|
| YYYY-MM-DD | §5 AC-2 tightened | {what forced it} | {name} |

Under `Compliance critical: true`, changing this spec without a row here is a halt.
