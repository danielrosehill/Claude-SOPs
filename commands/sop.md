---
description: Run one of your standard operating procedures — or list, write, edit or retire them
argument-hint: [what you're about to do] — e.g. publish a plugin; or "list", "new", "sync"
---

Arguments, if given: `$ARGUMENTS`

Route on what was asked:

| Argument | Skill |
| --- | --- |
| a situation or SOP name | `sop-run` — match the index and follow it |
| `list`, `ls`, or empty | `sop-list` — show the library |
| `new`, `write`, `add` | `sop-write` — capture a procedure |
| `edit`, `fix`, `update` | `sop-edit` |
| `retire`, `remove`, `delete` | `sop-remove` |
| `sync`, `push`, `pull` | `sop-sync` |
| `setup`, `init` | `sop-setup` |

With no argument, show the index and ask which procedure they want — do not pick
one on inference.

Two things hold whichever route this takes:

- **Read `INDEX.md`, then exactly one SOP.** Never read the whole library folder
  to answer a question; the index carries the summary, and reading past it is
  the cost this structure exists to avoid.
- **An SOP's `autonomy` field records a default, not a licence.** It never
  authorises something the user has declined in this session, and irreversible
  or outward-facing steps still get confirmed.

If there is no library yet, say so once and offer `sop-setup` rather than
improvising a folder.
