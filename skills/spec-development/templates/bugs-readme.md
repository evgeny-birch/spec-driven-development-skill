# Bug reports

This directory holds **defect reports** — one markdown file per defect — surfaced
while writing tests, reviewing code, or running the product. A bug report
documents the gap between *intended behaviour* (as defined by the originating
SPEC) and *actual behaviour* (what the code does today). It is **not** a fix.
Test agents file these so a follow-up hotfix or fix-spec has a precise repro,
severity, and pointer to the test that captured the wrong behaviour.

Who reads this directory:
- **Future SPEC authors** scoping a fix-spec — to know what's already known
  broken and at what severity.
- **Code reviewers** verifying that an `it.skip("blocked by BUG-NNN — …")` /
  `t.Skip("blocked by BUG-NNN — …")` / equivalent annotation has a real
  report behind it.
- **The orchestrator** at spec hand-off — to confirm every skip-with-reference
  has a matching `BUG-NNN-*.md` file.

Who closes a report:
- The agent or human who lands the corresponding fix. The fix lives in a
  separate hotfix branch or a follow-up SPEC; this directory only tracks the
  *defect record*. After the fix merges, the report's `status` flips to
  `closed` and stays in the directory as historical record.

This convention is provided by the `spec-development` skill
(`.claude/skills/spec-development/`). The first spec in this project that
needed defect tracking copied the templates here. See the project's
`docs/specs/` for the originating spec.

---

## Numbering rule

- Sequential, project-wide, three-digit zero-padded: `BUG-001`, `BUG-002`,
  …, `BUG-999`. No per-spec or per-package namespacing.
- File name is `BUG-NNN-{slug}.md`, where `{slug}` is a short kebab-case
  description (e.g. `BUG-001-auth-store-reset-leaves-pending-contact.md`).
- The next available number is `max(existing) + 1`. To find it from the
  shell:

  ```sh
  ls docs/bugs/BUG-*.md 2>/dev/null \
    | sed -E 's|.*/BUG-([0-9]+)-.*|\1|' \
    | sort -n | tail -1
  ```

- If no bug files exist yet, the first one is `BUG-001`.

---

## Severity scale

Four levels. Pick the highest level that applies; when in doubt, escalate
one level rather than down-grade.

| Level | Definition | Examples |
|---|---|---|
| **critical** | Data-loss, security vulnerability, PHI/PII leak, or production-incident-class defect. Drop-everything fix track. | Token leaks in logs; admin endpoint accepts unauthenticated requests; user data lost on a regular flow. |
| **high** | Wrong behaviour that is **user-visible** on a normal flow. Blocks a feature or produces a clearly wrong outcome the user can see. | Sign-out leaves a stale session cookie; admin "delete user" silently no-ops; the wrong locale renders on a localised page. |
| **medium** | Wrong behaviour that is **internal** or non-user-visible. The user cannot see it directly, but it violates the source SPEC and may compound. | Audit writer drops a field; an internal counter resets at the wrong time; reset-action leaves a stale field that the next flow then ignores. |
| **low** | Polish / cosmetic. Spacing off, copy slightly inconsistent with the design system, a console warning. Not breaking intended behaviour. | Border colour off by one shade; a non-blocking warning in dev console; English plural form on an EN-only page is wrong. |

`low` is **rare** in this directory. If a thing is "polish but not breaking
SPEC behaviour" the right place is `docs/future-work.md`, not here. See the
anti-pattern note below.

---

## Lifecycle

```
open  ──►  fix-tracked  ──►  closed
```

- **open** — the report has been filed; no fix branch exists yet. Default
  state when the file is created.
- **fix-tracked** — a hotfix branch / fix-spec is in flight that explicitly
  cites this `BUG-NNN`. The bug report's `status` field is updated when the
  fix work begins, with a pointer to the branch / spec ID.
- **closed** — the fix has merged to the project default branch and any
  blocked tests have been un-skipped. The bug report stays in this
  directory; it is **not** deleted. Update `status` to `closed` and add a
  one-line resolution note at the bottom of the file.

Closed reports are historical: they help future readers understand "yes,
that was a bug; here's how it manifested; here's where the fix lives".

---

## Skip-with-reference convention

When a test surfaces a defect, the test is annotated with a skip-with-reference
string that names the bug. A grep across the repo always reconciles
bug reports ↔ skipped tests. Format per language:

| Stack | Annotation |
|---|---|
| Vitest / Jest | `it.skip("blocked by BUG-NNN — {short reason}", …)` |
| Go | `t.Skip("blocked by BUG-NNN — {short reason}")` |
| Pytest | `@pytest.mark.skip(reason="blocked by BUG-NNN — {short reason}")` |
| JUnit (5+) | `@Disabled("blocked by BUG-NNN — {short reason}")` |
| RSpec | `skip "blocked by BUG-NNN — {short reason}"` |

The `BUG-NNN` token is mandatory and discoverable by grep:

```sh
grep -rEn 'blocked by BUG-[0-9]+' .
```

Every match must point to a real `docs/bugs/BUG-NNN-*.md`.

---

## Cross-references

- **`_template.md`** — the canonical structure for a `BUG-NNN-{slug}.md`
  file. Copy it, fill every section, rename. Do not invent new sections;
  do not remove sections (write `n/a` if a section truly does not apply,
  with a one-sentence justification).
- **`docs/specs/`** — the project's specs. Bug reports cite the spec
  section that defines the *intended* behaviour the code violates.
- **`docs/future-work.md`** — for non-defect deferred work. Read the
  anti-pattern note below before deciding which directory to write to.

---

## Anti-pattern: when *not* to file a bug

This directory is for **defects** — concrete deviations from intended
behaviour as defined by an existing SPEC. It is **not** for:

- **Missing features.** "We should also support X" → `docs/future-work.md`.
- **Non-blocking polish.** "This card spacing looks off but the SPEC didn't
  pin it" → `docs/future-work.md`.
- **Environment / tooling flakes.** "Test passed locally but failed in CI
  once" → task hand-off "Additional scenarios", not a bug report.
- **Aesthetic preferences with no SPEC backing.** → `docs/future-work.md`.
- **Things that surfaced *because* of the test-write but are intended
  behaviour the agent disagreed with** → discuss with the user, not file
  unilaterally.

Litmus test before opening a `BUG-NNN-*.md`: **can you cite the SPEC
section the current code violates?** If yes, file the bug. If no, the
right home is `docs/future-work.md` or a discussion with the orchestrator.
