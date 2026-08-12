---
name: sop-remove
description: Retire an SOP that no longer applies — moving it to the library's archive, or deleting it outright when asked. Use when the user says a procedure is obsolete, that they do not do it that way any more, or that a system it depends on is gone; and when a replacement SOP has been written for the same job.
---

# Retire an SOP

Default to archiving. Deletion is available and is sometimes right, but a
retired procedure is often the only surviving record of how a system used to
work, and that is not obvious until someone needs it.

## Before anything, read it

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" show <id>
```

Then check what the retirement really is:

| Situation | What to do instead |
| --- | --- |
| One step is wrong | `sop-edit`. Do not retire a procedure over a renamed flag. |
| It has been superseded | Write the replacement first, list this id in the new one's `supersedes`, then retire. |
| It is stale, not obsolete | Staleness means unverified. Unverified is not the same as untrue. |
| Genuinely obsolete | Retire it. |

Ask the user which of these it is if the request is ambiguous — "we don't do
that any more" covers all four.

## Archive

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" retire <id>
```

Moves it to `archive/<YYYY-MM-DD>-<id>.md` and reindexes. Archived files are not
in `INDEX.md`, so the dispatcher will not find them — which is the point — but
they remain readable and greppable.

Add one line at the top of the archived file saying why it was retired and what
replaced it, if anything. In two years that line will be the only thing making
the file interpretable.

## Delete

Only when the user asks for deletion specifically, and only after saying what is
being lost.

```bash
LIB="$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path)"
git -C "$LIB" rm "<id>.md"
```

Use `git rm` rather than `rm` — the history keeps the content, which makes this
recoverable and is worth saying to the user as reassurance.

If they want it gone from history too, that is a rewrite of a repo they mirror
across machines. Say what that involves and let them decide; do not start it as
part of a retirement.

## Finish

```bash
LIB="$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path)"
ROOT="$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" repo)"
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" index
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" validate
git -C "$ROOT" add -- "$LIB"
git -C "$ROOT" commit -m "Retire SOP: <id>"
```

Stage `$LIB`, not `-A` — the repo root holds other tools' folders too.

`validate` after retiring catches the dangling reference — another SOP whose
`supersedes` or `## Notes` points at the id that just left. Fix those in the
same commit; a link to a procedure that is not there reads as a bug in the
library rather than a deliberate retirement.
