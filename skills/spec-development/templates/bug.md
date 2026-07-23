# BUG-NNN — {short title in sentence case}

> Copy this file to `docs/bugs/BUG-NNN-{slug}.md`. Replace every `{…}`
> placeholder. Do not delete sections; write `n/a` plus a one-sentence
> justification if a section truly does not apply.

## Meta

| Field | Value |
|---|---|
| ID | `BUG-NNN` |
| Title | `{short title in sentence case}` |
| Slug | `{kebab-case-slug-used-in-the-filename}` |
| Severity | `low` / `medium` / `high` / `critical` |
| Discovered | `YYYY-MM-DD` |
| Discovered during | `SPEC-NNN, T-XXX-{slug}` (which spec + which task surfaced this) |
| Status | `open` / `fix-tracked` / `closed` |
| Originating SPEC | `SPEC-NNN §X.Y` (the spec section that defines the *intended* behaviour the code violates) |
| Suggested fix track | `hotfix` (prod-broken, urgent) / `fix-spec` (design-level, follow-up SPEC) |

## Component

- **File path(s):** `{relative/path/to/file.ext}` — repeat for every
  file the defect lives in.
- **Package(s):** `{language-specific package or import path}` (e.g.
  `internal/admin` for Go, `apps/web/features/admin-users` for a frontend
  feature, `com.example.foo` for Java).
- **Feature area:** `{auth / onboarding / payments / admin / shared / api / db / …}` —
  high-level zone the defect belongs to. Use the project's own taxonomy.

## Repro

Numbered, plain prose. Reproducible from a clean checkout. Include the
exact command lines anyone needs to run.

1. `{step one — e.g. "start the test stack"}`
2. `{step two — e.g. "curl -X POST … -d '…'"}`
3. `{step three — e.g. "observe the response body"}`

If the repro is via a failing test, give the exact path:

```
{relative/path/to/test_file_test.tsx}::{test name}
```

## Expected vs actual

**Expected** (per `{Originating SPEC §X.Y}`): one short paragraph stating
what the code *should* do. Quote the SPEC clause if it's terse enough.

**Actual:** one short paragraph stating what the code *currently* does.
This is what the test observed (or what the manual repro produced).

## Evidence

Failing-test snippet, log output, or a link to a screenshot. If the data
is sensitive (PHI, PII, real auth tokens, real session cookies, real
customer identifiers), **scrub before committing**. Replace user
identifiers with `user-redacted`, tokens with `<TOKEN>`, etc.

```
{paste failing assertion, stack trace, or relevant log lines here}
```

## Test status

The originating test is annotated with the canonical skip-with-reference
string so a grep across the repo always reconciles bugs ↔ skips:

- **File:** `{relative/path/to/test_file.test.tsx}`
- **Test name:** `{the it/describe path}`
- **Annotation:** language-specific skip-with-reference. Use the project's
  test framework's idiomatic skip:
  - Vitest / Jest: `it.skip("blocked by BUG-NNN — {short reason}", …)`
  - Go: `t.Skip("blocked by BUG-NNN — {short reason}")`
  - Pytest: `@pytest.mark.skip(reason="blocked by BUG-NNN — {short reason}")`
  - JUnit: `@Disabled("blocked by BUG-NNN — {short reason}")`
  - RSpec: `skip "blocked by BUG-NNN — {short reason}"`

If the bug surfaced without a paired test (manual finding, code review),
write `n/a — surfaced via {…}` and explain.

## Recommendation

One or two sentences explaining why this is `hotfix` vs `fix-spec`:

- `hotfix` — production is currently wrong and the fix is small + isolated.
  Open a small PR off the project default branch referencing `BUG-NNN` in
  the title.
- `fix-spec` — the fix needs a design pass (e.g. picks among multiple
  reasonable behaviours, or the SPEC itself was ambiguous). Add to the
  next planning round; cite this bug as the trigger.

## Resolution (filled when status flips to `closed`)

> Leave blank while `open` / `fix-tracked`. Filled by whoever lands the
> fix. One paragraph: where the fix lives (PR / commit / SPEC), what
> changed, which tests were un-skipped.
