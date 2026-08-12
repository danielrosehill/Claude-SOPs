# The `CLAUDE.md` snippet

The library is useless if nothing knows it is there. One block in the user-level
`CLAUDE.md` (`~/.claude/CLAUDE.md`) fixes that, and it is the only edit this
plugin ever makes outside its own vault.

## The block

Substitute the resolved library path for `~/claude-sops`.

```markdown
<!-- claude-sops:begin -->
<!-- Managed by the claude-sops plugin. Edit the SOPs, not this block. -->

## Standard operating procedures

Recurring procedures I have written down live in `~/claude-sops/`, indexed in
`~/claude-sops/INDEX.md`.

Before improvising any multi-step procedure — publishing, deploying, filing,
ordering, onboarding, closing a period — read `INDEX.md` and check the **Use
when** column for one that already covers it. If one matches, follow it rather
than working it out again. Read only the SOP you matched.

If the same sequence comes up a second time and no SOP covers it, offer to
write one.
<!-- claude-sops:end -->
```

## Rules for editing it

**Idempotent, always.** If both markers are present, replace what is between
them. Never append a second block — two copies disagree the moment one is
updated, and the file grows a little each time the path changes.

**Both markers or neither.** If exactly one marker is present the file has been
hand-edited; stop and show the user rather than guessing where the block ends.

**Back up first.** Copy to `~/.claude/CLAUDE.md.bak-<date>` before writing.
`CLAUDE.md` is often the most carefully tuned file a user owns.

**Show the diff, then ask.** This edits a file that governs every future
session. It is not a step to take on inference, even mid-procedure.

**Removal is deletion of the block**, including both markers and the blank line
that separated it from its neighbours — not commenting it out.

## Placement

Append at the end unless the user says otherwise. `CLAUDE.md` files usually have
a deliberate order, and inserting into the middle of one is a judgement call
that belongs to the user, not the plugin.

If the file does not exist, create it with just this block and say so — an empty
`~/.claude/CLAUDE.md` is a normal state, not a problem to solve.

## A note on wording

The snippet says *check the index before improvising*, not *always read the
index*. The difference matters: the second is a chore attached to every session
and gets ignored within a week. Keep any local rewording on the same side of
that line.
