# SOP library

Standard operating procedures — the recurring things done a particular way,
written down once so they do not have to be re-derived.

```
INDEX.md          generated table of every SOP; the only file read by default
README.md         this file
<id>.md           one procedure per file — flat, readable, editable by hand
archive/          retired procedures, kept for the history
```

`INDEX.md` and `README.md` are reserved names. Everything else ending in `.md`
is a procedure.

## How it is used

An assistant reads `INDEX.md`, matches the situation against the **Use when**
column, and opens exactly one `<id>.md`. Nothing else here enters context. That
is the point of the split: the library can grow to a hundred procedures without
any of them costing anything until the moment one is needed.

## Working on it

With the [claude-sops](https://github.com/danielrosehill/Claude-SOPs) plugin
installed:

| Ask for | What happens |
| --- | --- |
| "run the SOP for X" / `/sop X` | matches the index, follows the procedure |
| "write this up as an SOP" | captures what just happened into a new file |
| "the SOP for X is out of date" | amends it and re-dates `last_verified` |
| "retire the X SOP" | moves it to `archive/` |
| "sync my SOPs" | commits and pushes this repo |

By hand, the tooling is a single stdlib Python script in the plugin:

```bash
python3 scripts/sop.py list
python3 scripts/sop.py new deploy-the-site --title "Deploy the site"
python3 scripts/sop.py validate
python3 scripts/sop.py index      # regenerate INDEX.md after any edit
```

## House rules

- **`INDEX.md` is generated.** Edit the SOPs and regenerate; hand edits are lost.
- **The filename is the id.** No timestamps, no numbering — this folder is meant
  to be browsable by a person who has never used the plugin.
- **`last_verified` means it was seen to work**, not that the file was touched.
  An SOP unverified for a year is flagged ⚠️ in the index.
- **One procedure per file.** If a step needs three paragraphs of background, the
  background belongs somewhere else, linked.
- **No secrets.** Reference credentials by name or `op://` path; never paste a
  value. This repo is private, but it is still a repo.
