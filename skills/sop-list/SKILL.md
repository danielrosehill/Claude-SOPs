---
name: sop-list
description: Show what is in the SOP library — everything, or filtered by tag, or only the procedures that have gone stale. Use when the user asks what SOPs they have, whether one exists for something, which ones need reviewing, or wants an overview of their procedures before deciding what to write next.
---

# List the library

Discovery, review and pruning. `sop-run` matches one procedure; this one shows
the shape of the whole collection.

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" list                 # everything
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" list --tag deploy    # one tag
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" list --stale 365     # unverified for a year+
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" list --json          # full front matter
```

Reading `INDEX.md` directly gives the same information as a rendered table,
which is usually the better thing to show a person.

No library → say so and offer `sop-setup`. An empty one is not an error; it is a
new library.

## Reporting it

Show the table: id, title, trigger, verified date. Sort by tag when there are
enough to group, otherwise leave it alphabetical. **Do not open the SOP files to
summarise them** — the front matter is the summary, and reading forty files to
answer "what do I have" is the cost this whole structure exists to avoid.

When the user asked whether an SOP exists for something specific, answer that
question rather than printing everything: name the match, or say there is none
and offer `sop-write`.

## The staleness pass

Worth offering unprompted when the list is being reviewed anyway.

`--stale 365` is a starting filter, but age alone is a weak signal — read it
against `frequency`:

| | Interpretation |
| --- | --- |
| `frequency: monthly`, unverified 8 months | Suspect. Either it stopped being done, or it is being done without the SOP. Both are worth knowing. |
| `frequency: yearly`, unverified 13 months | Normal. It will be re-dated at the next run. |
| `frequency: rare`, unverified 3 years | Fine, and exactly why it was written down. |

For each genuinely stale one, offer three outcomes and let the user pick: verify
it on the next run, edit it now if they know what changed, or retire it if the
procedure no longer exists. **Do not re-date anything here.** `last_verified`
means the procedure was seen to work, and a review is not a run.

## Health checks worth running while you are here

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" validate
```

Then look for the things validation cannot see:

- **Two SOPs with overlapping triggers.** The dispatcher will pick one
  arbitrarily. Narrow one or merge them.
- **A trigger that restates the title.** It matches nothing the title didn't,
  so the SOP is effectively unfindable by situation.
- **An SOP with no `## If it goes wrong` content.** Usually means it has been
  written but never run in anger.

Report these as a short list, not a lecture, and offer to fix them one at a time.
