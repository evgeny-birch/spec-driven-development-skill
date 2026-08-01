# Production readiness

> **How to use this file**
>
> - Copy this template to `docs/prod-readiness.md` the first time a spec or hotfix defers a pre-production step to it.
> - This is the register of **human steps that no agent can perform** and that must happen before the work is live: a console setting, a DNS record, a partner sign-off, a credential provisioned by someone with access, a legal review, a billing limit raised.
> - The value of this file is that it exists **before** launch day. An item discovered at deploy time has already cost you the outage.
> - Delete this block when the file is filled in.

## What belongs here

An item qualifies if **all three** hold:

1. It must happen before the change is live in production.
2. A human with access outside the repository has to do it (it cannot be committed).
3. Nothing in the automated pipeline will fail if it is forgotten — which is exactly why it needs a written owner.

If the pipeline *would* catch it, it is not a prod-readiness item; make it a test or a gate instead.

## Register

`Source` names the spec or hotfix that raised the item. `Blocking?` = does production go live without it, or not.

| Item | Source | Owner | Blocking? | Done |
|---|---|---|---|---|
| {console setting / DNS record / partner review / credential — one line, concrete enough to act on without asking} | SPEC-NNN §19 | {name} | yes / no | [ ] |
| {…} | HF-NNN §8 | {name} | yes | [ ] |

Rules for the table:

- **One item per row, phrased as an action** — "set X to Y in the Z console", not "check the Z config".
- **`Owner` is a person, never a team or `TBD`.** An unowned blocking item is an outage with a scheduled date.
- **`Blocking? = yes` means the release does not ship.** If everything is marked blocking, the column carries no information — be honest about which items are genuinely gating.
- **Rows are not deleted after launch.** They are the record of what the environment needed, and the next environment (staging → prod, region two) usually needs the same list.

## At epic close

The orchestrator surfaces the open items from this file in its final hand-off, so nothing here is discovered by the person doing the deploy.
