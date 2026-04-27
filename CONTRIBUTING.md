# Contributing

Thanks for considering a contribution. This skill is small enough that the bar is low — if you've used it on a real project and noticed a sharp edge, that feedback is the most valuable thing you can bring.

## What's in scope

- **Bug fixes** in `SKILL.md` or any template — wording that misleads the agent, sections the agent skips, instructions that contradict each other.
- **New templates** for tracks that aren't covered (e.g. RFC, ADR) — open an issue first to discuss whether it belongs here or in a sibling skill.
- **Workflow refinements** in `docs/workflow.md` when the underlying skill changes.
- **Examples** showing the skill in action on a small open project (sanitised — no private data).

## What's out of scope

- Project-specific conventions (your stack, your i18n rules, your compliance flow). Those live in your project's `CLAUDE.md`, not here.
- Heavy framework lock-in (e.g. coupling templates to a specific issue tracker, language, or CI). The skill is meant to stay project-agnostic.
- Breaking changes to the `SPEC-NNN` / `HF-NNN` / `BUG-NNN` numbering scheme — too many downstream users would have to migrate.

## Workflow

1. **Open an issue first** for anything beyond a typo or one-line fix. A 3-line description of what you're trying to solve saves both of us a wasted PR.
2. **Fork → branch → PR.** Keep PRs focused: one fix or one feature per PR.
3. **Test the change against a real project.** Templates that look right in isolation can fail on contact with an actual epic. Run the skill end-to-end before opening the PR; mention which track you tested.
4. **Update `docs/workflow.md`** if your change affects user-facing flow. The two documents must stay in sync — `SKILL.md` is the contract with the agent, `workflow.md` is the contract with humans.

## PR checklist

- [ ] One concern per PR.
- [ ] Skill still parses (frontmatter intact, no broken links).
- [ ] If you changed a template, walked at least one real spec through it.
- [ ] If you changed `SKILL.md`, the corresponding section in `docs/workflow.md` matches.
- [ ] No private data in examples (project names, partner names, internal URLs, real spec/HF/BUG IDs from your repo).

## Licensing

By submitting a PR, you agree your contribution is licensed under the project's [MIT License](LICENSE).
