---
name: sop-run
description: Find the user's standard operating procedure for the task at hand and follow it. Reads the SOP index, matches the situation, opens exactly one procedure and executes it at the autonomy level that SOP declares. Use whenever a multi-step recurring task comes up — deploying, publishing, filing, ordering, onboarding, closing a period — before working the sequence out from scratch, and whenever the user says "run the SOP for X" or "how do I usually do X".
---

# Run an SOP

The dispatcher. It exists so that a library of any size costs one table to
consult and one file to read.

## 1. Read the index, and only the index

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" list --json
```

Or read `INDEX.md` at the path from `sop.py path` — same content, and the table
is easier to scan when reporting back to the user.

**Do not glob the library folder and read everything.** That defeats the entire
design; a mature library is tens of files, and pulling them all in to answer one
question is the cost this structure exists to avoid.

If there is no library, say so once and offer `sop-setup`. Do not improvise a
folder.

## 2. Match on the situation

Match the user's circumstance against the **Use when** (`trigger`) column, not
against the title. Titles collide; triggers discriminate.

| What you find | What to do |
| --- | --- |
| One clear match | Proceed to step 3. |
| Two or more plausible | Show the candidates with their triggers and ask. Do not merge two procedures. |
| A near match | Say what it covers and what it does not, and ask whether to adapt it or write a new one. |
| Nothing | Say so plainly and carry on with the task normally. Offer `sop-write` afterwards. |

An SOP unverified for over a year is flagged ⚠️ in the index. Follow it, but say
the date up front and treat surprises as the SOP being wrong rather than the
world being wrong.

## 3. Read the one file

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" show <id>
```

Check `## Preconditions` before starting. A procedure entered halfway with a
precondition unmet usually fails several steps later, somewhere unrelated, and
looks like a different problem entirely.

`requires` lists tools that must be present. Check them; a missing CLI is a
clearer thing to report at step 0 than at step 6.

## 4. Execute at the declared autonomy

| `autonomy` | Behaviour |
| --- | --- |
| `auto` | Run through. Report at the end. |
| `confirm` | One confirmation before the first step that changes anything outside this machine. Default when absent. |
| `confirm-each` | Ask at every mutating step. |
| `manual` | Do not execute. Read the procedure back and let the user drive. |

Two things the field does not do. It **does not widen permission**: an SOP
marked `auto` does not authorise something the user has just declined, and it
does not override the ordinary rule that outward-facing and irreversible actions
get confirmed. And it **does not lower the bar on reporting**: say which steps
ran, which were skipped, and why.

Follow the steps as written. If one is wrong, stop at that step rather than
routing around it silently — a procedure quietly deviated from is a procedure
that never gets corrected.

## 5. Verify, then close the loop

Run the `## Verification` section. "The commands exited 0" is not verification;
the section names an external signal for a reason.

Then, and this is the step that keeps the library alive:

- **It worked as written** → update `last_verified` to today. One line, via
  `sop-edit`. Do not ask permission for this; it records a fact.
- **It worked but the steps had drifted** → `sop-edit` to fix the drift, and say
  what changed.
- **It failed** → do not re-date it. Add what happened to `## If it goes wrong`,
  and tell the user the SOP is now known-broken at step *n*.

An SOP nobody re-dates becomes an SOP nobody trusts, and then a library nobody
reads.
