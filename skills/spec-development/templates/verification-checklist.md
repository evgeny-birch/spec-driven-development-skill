# SPEC-{NNN} — Verification checklist (per task hand-off)

> **How to use this template**
>
> - Copy this file to `docs/specs/SPEC-{NNN}-{slug}/verification-checklist.md` when the spec is created.
> - Sections §0–§9 are the **universal floor** — keep them verbatim. They are project-agnostic and apply to every spec.
> - Add a new section §10+ **"Project / spec-specific surfaces"** with checklist items for whatever the epic touches that the universal floor does not cover. See examples at the bottom.
> - Delete the "How to use this template" block when the file is filled in.

**Read this BEFORE you mark a task `completed` and BEFORE you write the hand-off summary.**

This is a single-page contract between the executing agent and the orchestrator. Past specs in many repos have shipped with green tests that proved mechanics, not behaviour. This spec is too important to repeat that. If any item below is unchecked, the task is NOT complete; either fix it or surface it as a known gap with a filed BUG-NNN.

How to use:
1. Copy this file's checklists into your task hand-off message.
2. Tick each item with how you verified it (command + expected output, OR storage query + result, OR screenshot path).
3. Items marked **MANDATORY** cannot be waived — see the waiver rules under §8. Items marked *contextual* apply only when the task touches the relevant surface.
4. Items you cannot verify in your environment must be flagged explicitly (`could not verify in env — needs eyeball by user`), not silently skipped.

---

## §0 Universal — every task, no exceptions

- [ ] **Type-check passes.** `tsc --noEmit` (TS) / `mypy app` (Python) / `go vet ./...` + project linter — exit 0.
- [ ] **Lint passes.** Project's standard linter (`eslint`, `ruff`, `gofmt`, etc.) — clean.
- [ ] **Existing test suites still green.** Project's standard test command — no new failures introduced by this task.
- [ ] **Core scenarios from the task block are implemented as automated tests AND green.** Each scenario maps to an AC.
- [ ] **Negative paired tests exist.** For every guard / validation / rule the task introduces, at least one "fires correctly" AND at least one "does NOT fire incorrectly" test. Half-tested guards are common rubber-stamp footguns.
- [ ] **Additional scenarios discovered during implementation appended to the task file.** If you found edge cases, write them down — they protect the next reader.
- [ ] **Adjacent-configs sweep done.** When the task changes an integration boundary (auth model, port, env-var name, new service, storage path), grep across `Makefile`, `infra/`, `.github/workflows`, `*Dockerfile`, README, scripts, `.env.example`. All hits fixed in the same PR.
- [ ] **MANDATORY — Exit-code honesty (`VR-15`).** A multi-step run's "green" is read from each step's own result, not the exit code of the last command in a pipe / a trailing `grep`/`tail`. A backgrounded "exit 0" is not proof — read the artefact/log the run produced.
- [ ] **MANDATORY — No un-gated or silently-skipped layers (`VR-16`).** Every test layer that matters runs in the gate; a self-skipped test without a `BUG-NNN` reference is rot, not a pass. A layer that can't run here is surfaced at hand-off, not dropped.
- [ ] **MANDATORY — Domain failure surfaces as non-2xx (`VR-18`).** Any domain/business failure the task can produce returns 4xx/5xx, never a 2xx with an error body. A `200` with `status:"failed"` is a green that lies.
- [ ] *contextual* **Reproducible from scratch (`VR-12`).** The hand-off's "Reproduce the green state" block carries the exact commands and expected output — commit, branch, env. "Works on my machine" closes nothing.
- [ ] *contextual* **Live exercise for infra / deployment work (`VR-13`).** Paper validation (linter, config-validate, dry-run) proves syntax, not behaviour. A task whose deliverable is infrastructure stays `done-paper` until a smoke target returns green from a real deploy.
- [ ] *contextual* **Idempotent setup scripts (`VR-14`).** A bootstrap / migration / provisioning script detects its own re-run state at every mutating step. Tested by running it twice in succession.
- [ ] **PR description references the task ID and SPEC-NNN.**
- [ ] **Hand-off summary uses the §8 template at the bottom of this file.**

---

## §1 DB / persistence-touching tasks

When the task writes to or reads from a persistent store (Postgres, MongoDB, Redis, file system):

- [ ] **MANDATORY — Existence asserted by storage query, not by helper-call inspection.** Every "writes a row / object / record" claim has a `SELECT … WHERE …` (or equivalent fetch) in the test. Calling the helper does not count.
- [ ] **MANDATORY — Status / state-machine transition asserted by post-condition row read** after the action. "The handler returned 200" is necessary but not sufficient.
- [ ] **MANDATORY — Foreign-key cascades verified.** If your migration adds an FK with `ON DELETE CASCADE` (or equivalent), write a test that creates parent + child, deletes the parent, asserts the child is gone via storage query.
- [ ] **MANDATORY — Optimistic-concurrency tested with row count after action.** When you claim "second concurrent write returns 409", the test post-condition shows row count = 1 (only the winner left a row).
- [ ] **MANDATORY — Scope assertions to rows you created, not a global total (`VR-17`).** Assertions filter by seeded id / unique marker; never assert a global count or "the whole list has N" — fragile under parallelism and cross-task contamination (first-class with the `dw` engine). Clean up rows you create (`afterAll`/`finally`).
- [ ] **Migration up + down round-trip.** Forward → down → forward succeeds without manual cleanup against a fresh DB.
- [ ] **Index existence asserted** when migration adds an index (e.g., `pg_indexes` query for Postgres).
- [ ] **Enum membership asserted** when migration adds enum values.

---

## §2 UI / Frontend tasks

When the task ships a user-visible surface:

- [ ] **MANDATORY — Real-browser walkthrough.** You opened the page in a real browser on the live dev stack and clicked through. Per project memory in many repos: green tests are mechanics, NOT UX. If your environment cannot run a browser, surface this in the hand-off as `could not verify UX — needs eyeball by user on <route>`, do NOT mark complete silently.
- [ ] **MANDATORY — Accessibility scan with zero violations.** Run axe-core or equivalent on every page the task touches.
- [ ] **MANDATORY — Keyboard-only path through the surface.** Tab order logical; every interactive element reachable; focus trapped in modals; Escape closes; focus returns to trigger after close.
- [ ] **MANDATORY — Empty / loading / error states have explicit visual treatment.** No "blank screen on first load" states.
- [ ] **MANDATORY — Computed style / DOM state asserted, not just className.** When you claim a UI behaviour ("renders right-to-left", "is disabled", "shows error styling"), the test asserts `getComputedStyle(el).<prop>` (or framework equivalent: `aria-disabled`, `dataset.state`), not just `el.classList.contains('rtl-class')`. Utility-CSS frameworks can fail to apply silently.
- [ ] **MANDATORY — i18n keys exist for every supported language.** No hardcoded strings in markup.
- [ ] *contextual* **URL state round-trip** (filterable lists / form-driven URLs): query params reflect filters; reload preserves; clear-filters clears.
- [ ] *contextual* **Network-intercepted polling** (live-progress views): polling cadence asserted by intercept count over time; polling stops on terminal status.
- [ ] *contextual* **Image / media rendered, not just fetched** — `image.naturalWidth > 0` after load; visible bbox non-zero in screenshot. A 200 OK with corrupt bytes would pass an HTTP check but fail this — that's the whole point.

---

## §3 Audit-log-touching tasks

When the task claims to write an audit-log row:

- [ ] **MANDATORY — Row asserted by storage grep.** Query the audit storage and assert exactly one row with the expected actor / entity / action / reason fields.
- [ ] **MANDATORY — Reason / feedback string verbatim.** When task says "the reason is recorded", the read row's reason column equals the input string verbatim. Truncation or encoding bugs caught here.
- [ ] **MANDATORY — Actor identity matches the authenticated principal.** Verify the row's actor id equals the test's authenticated user, not a fixture default.

---

## §4 Concurrency / race-conditions

When the task ships any kind of optimistic-concurrency or async behaviour:

- [ ] **MANDATORY — Real parallel writers, not sequential calls.** Test runs N workers (goroutines / async tasks) actually simultaneously and asserts post-condition row count = 1 winner.
- [ ] **MANDATORY — Conflict response shape verified.** When the system signals a conflict (409 / version_stale / etc.), the test asserts the response body / structure — not just the status code.
- [ ] **MANDATORY — No partial writes on conflict.** When a conflict is returned, the test asserts no side-effect rows (audit, side-tables) were written by the loser.

---

## §5 E2E adversarial pinning

When the task ships any E2E test:

- [ ] **MANDATORY — Locale, timezone, and key request headers pinned in runner config.** Per project memory in many repos: Playwright / Cypress headless defaults differ from real Chrome. Pin `locale`, `timezoneId`, `extraHTTPHeaders` (especially `Accept-Language`) to a representative real user.
- [ ] **MANDATORY — Adversarial flip variant.** At least one variant per signal-sensitive test that *flips* the signal (e.g., `Accept-Language: <other-locale>`) and asserts the deployment default still holds.
- [ ] **MANDATORY — Real-browser run captured at least once per scenario family.** Trace viewer artefact (Playwright trace / Cypress recording) for one run with `headless: false`.

---

## §6 External-API-touching tasks (LLMs, payment, identity, etc.)

When the task involves an external API as the unit being verified:

- [ ] **MANDATORY — Mock-boundary payload capture.** When the task says "the prompt / request includes X verbatim", the test captures the payload at the mock boundary and asserts string-containment / structural match. Don't trust that "the helper that builds the payload was called".
- [ ] **MANDATORY — Negative parser test.** If the external API returns malformed / out-of-range output, the parser rejects BEFORE any persistence write. Test counts persisted rows before + after the malformed call; difference = 0.
- [ ] **MANDATORY — Cost / metric row written for every attempt** (when the task ships a usage ledger). Storage query confirms count matches actual call count + non-zero metric value.
- [ ] **MANDATORY — Provider attribution by storage row read.** When you claim "Provider X handled this", the persisted row shows `provider='X'`. Don't trust mock-call inspection.
- [ ] **MANDATORY — Live-stack run before spec closes.** Mocked-only verification does NOT close tasks where the external API IS the unit under test. The spec's `T-NNN-live-smoke` (or equivalent) task executes against real dependencies.

---

## §7 i18n / Localised content

When the task ships content in multiple languages:

- [ ] **MANDATORY — Language-specific Unicode codepoint asserted.** When the task generates / displays content for language L, the test asserts at least one codepoint from L's primary script is present where expected. Auto-translation drift produces L-flavoured output that passes shape tests; this catches it.
- [ ] **MANDATORY — Schema / DB column language enforcement.** Items with `language=L1` produce L1 content; items with `language=L2` produce L2 content. Test parameterised over each supported language.
- [ ] **MANDATORY — Mock-boundary language template verification.** When the workers/handlers run on language L, the prompts / templates / strings sent downstream are the L-variant, not auto-translated from another. Asserted by capturing the payload and checking for L-codepoints.
- [ ] *contextual* **RTL rendering in DOM** when L is RTL: `[dir="rtl"]` selector matches; computed `direction: rtl`; text contains L-codepoints.

---

## §8 Hand-off summary template (paste at end of completion message)

Copy this template and fill it in for the orchestrator. Anything missing means the task is not complete.

```md
### Task T-NNN — Completion summary

**What changed (1 paragraph):**
…

**Files touched:**
- `path/to/file.ts` — created / modified — purpose
- …

**Tests added / modified:**
- `path/to/test.ts::describe block` — what it asserts
- …

**UX verification status:**
- For UI tasks: "Verified in real browser on `<route>` — clicked through happy path, edge cases X / Y / Z; screenshots at `…`"
- OR: "Could not verify UX in this environment — please eyeball `<route>` for `<concerns>`"
- For non-UI tasks: "n/a — backend / infra task"

**Storage / persistence spot-checks done (paste actual queries + results):**
- Audit row: `SELECT … → 1 row, action='<expected>', actor_id=…`
- (other storage assertions specific to this task)

**Verification rigour rules — all 18, one line each:**
- VR-01 (persistence by storage query): ✅ / n/a — <no persistent store touched>
- VR-02 (state-machine post-condition read): ✅ / n/a — <no status transition>
- VR-03 (audit row by storage grep): ✅ / n/a — <no audit log>
- VR-04 (concurrency by post-condition count): ✅ / n/a — <no concurrent path>
- VR-05 (negative test paired): ✅ / ⚠️ waived — <reason>
- VR-06 (content not shape): ✅ / ⚠️ waived — <reason>
- VR-07 (browser walkthrough): ✅ on `<route>` / could-not-verify — <reason> / n/a — <no UI surface>
- VR-08 (computed style not className): ✅ / n/a — <no UI surface>
- VR-09 (E2E adversarial pinning + flip): ✅ / n/a — <no E2E suite>
- VR-10 (image / external resource render asserted): ✅ / ⚠️ waived — <reason> / n/a — <no media>
- VR-11 (live-stack run, external API as SUT): ✅ via T-NNN-smoke / n/a — <no external API under test>
- VR-12 (reproducible from scratch): ✅ — see "Reproduce the green state" below
- VR-13 (live exercise for infra specs): ✅ / n/a — <not an infra / deployment spec>
- VR-14 (idempotent setup scripts, tested twice): ✅ / n/a — <no setup / provisioning script>
- VR-15 (exit-code honesty — each step read): ✅
- VR-16 (no un-gated or silently-skipped layer): ✅
- VR-17 (assertions scoped to rows you created): ✅ / n/a — <no data created>
- VR-18 (domain failure surfaces as non-2xx): ✅ / n/a — <no HTTP surface>

**Waiver rules — read before writing `⚠️ waived` or `n/a`:**
- `⚠️ waived` is available **only** on the lines above that offer it (`VR-05`, `VR-06`, `VR-10`). Every other rule is either tagged MANDATORY on a surface family (`VR-01`–`VR-04`, `VR-07`–`VR-09`, `VR-11`, `VR-17`, `VR-18`) or *contextual* in §0 (`VR-12`–`VR-14`): if the task touches that surface it must be `✅`; if it does not, use `n/a` with the reason. `VR-15` and `VR-16` are always `✅` — a task that ran anything read its results honestly and skipped no layer.
- A waived MANDATORY item is a `waiver-abuse` finding of severity **critical** at review, not a judgement call.
- Every `n/a` and every waiver states **why the rule is inapplicable to this diff** — which surface you did not touch, or which precondition does not hold. "Inconvenient", "out of scope", "no time", and a bare `n/a` with no reason are all rejected: the reviewer verifies each one against the diff, and a reason the diff contradicts is a `waiver-abuse` finding.

**Spec-level checklist items addressed (project / spec-specific from §10+):**
- (list)

**Manual verification checklist** (from the task's Manual verification subsection):
- [ ] Item 1 — done / how
- [ ] Item 2 — done / how
- …

**Additional scenarios appended to the task file:**
- (anything edge-case discovered during work)

**Open questions / deviations from the task spec:**
- (anything the orchestrator needs to know)

**BUGs filed (if any):**
- BUG-NNN: short title — severity — link to docs/bugs/BUG-NNN-*.md

**Reproduce the green state:**
1. `git checkout <commit-sha>`
2. `<command 1>` → expected `<output>`
3. `<command 2>` → expected `<output>`
```

---

## §9 Orchestrator pre-merge gate

These run *after* the agent's hand-off, before the wave is merged:

- [ ] All §0–§7 boxes checked above for every task in the wave, **and every §10+ project/spec-specific item**. The §10+ sections are the reason this file is per-spec; a gate that skips them gates nothing this spec actually added.
- [ ] Code-review skill executed if available; findings addressed or filed as BUGs.
- [ ] Security-review executed if the wave touches sensitive paths (auth, data, PII / PHI, payments, secrets).
- [ ] CI green on the merged wave branch.
- [ ] No new type errors or lint warnings introduced.
- [ ] **Documentation impact section filled and every entry checked off.** Each spec / hotfix carries a Documentation impact list (epic.md §11.4 / small-spec.md §6 / hotfix.md §5b). Confirm the list was filled at draft time AND every listed doc is updated in this PR. Sweep the existing docs for stale claims about the changed behaviour — grep symptom keywords (endpoint paths, key names, status codes, env vars) and update sequence diagrams / failure-mode tables that encode the OLD behaviour. Partial doc updates are how the architecture rots.

When all task agents in a wave hand off green AND these orchestrator gates pass, the wave merges into the integration branch and the next wave starts.

At epic end, the orchestrator surfaces:
- Thorough-review command (e.g., `/ultrareview`) — user-triggered, multi-agent cloud review.
- Aggregated manual verification checklist (composed across all tasks).
- Final BUG list with severities.
- Cross-spec deltas: `docs/future-work.md` and `docs/prod-readiness.md` entries committed.

---

## §10+ Project / spec-specific surfaces

> **Spec author: replace this block with sections specific to YOUR spec.** Examples drawn from past specs:
>
> - **§10 LLM-output content quality** — assert no Lorem-ipsum / placeholder text; question text length ≥ N chars; no Markdown fence leakage in JSON output.
> - **§11 Localised content (HE / AR / RU / etc.)** — assert specific Unicode codepoint range present in expected fields; mock-boundary HE template verification (catches EN-template-leak); RTL rendering computed-style assert.
> - **§12 Object-storage round-trips** — `mc cp` / `aws s3 cp` retrieves uploaded bytes; hash compare with what the worker thought it wrote; anonymous-bucket access denied; cross-permission probes (writer cannot read outside prefix; reader cannot write).
> - **§13 Numerical / scientific assertions** — golden-curve fixtures with known expected outputs; spread / shape assertions against degenerate flat outputs.
> - **§14 Multi-region behaviour** — pin region in tests; flip-region adversarial variant.
> - **§15 Cost / cost-reconciliation** — `SUM(<cost_table>.<col>)` matches dashboard total; per-stage breakdown matches GROUP BY query.
> - **§16 Provider rotation / load-balancing alive** — distinct providers / shards represented in the post-batch storage query; no single provider exceeds the configured cap.
> - **§17 Concurrency stress** — N-way parallel writers under realistic latency; post-condition invariant holds.
>
> Add only sections that match what your spec actually touches. Each item should have the same "MANDATORY" / "*contextual*" tagging as the universal sections.
