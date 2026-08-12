---
name: sop-write
description: Capture a recurring procedure as a new SOP in the user's library — usually written from what just happened in this session, while the details are still known. Use when the user says "write this up as an SOP", "remember how we did this", "add this to my procedures", or when the same multi-step sequence has now come up twice and nothing in the index covers it.
---

# Write a new SOP

The best SOP is written immediately after the procedure has been worked out,
from the session that worked it out. Written later from memory, it loses exactly
the parts that made it worth keeping: the wrong turns and the reason for each
non-obvious step.

Format spec: [`references/sop-format.md`](../../references/sop-format.md).

## 1. Check it does not already exist

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" list
```

A near-duplicate is worse than no SOP — two procedures for one job means every
future run starts with a choice nobody can make correctly. If something close
exists, use `sop-edit` to extend it instead, and say that is what you are doing.

No library at all? `sop-setup` first.

## 2. Check it is actually an SOP

Push back, once, if it is really one of these:

- **Reference material** — how a system is shaped, rather than how the user acts
  on it. That belongs in a docs repo, linked from an SOP's `## Notes`.
- **A one-off** — it will not recur, so it is a session, not a procedure.
- **A preference** — "always use `uv`" has no steps and no order. That is a
  standing instruction for `CLAUDE.md`.
- **A single command** — an SOP that wraps `npm test` adds a lookup and no
  information.

If they confirm after that, write it. Stated once is enough.

## 3. Draft from the session, not from imagination

Reconstruct from what actually happened. The material is in the transcript:
which commands ran, which failed first, what the error looked like, what the
fix turned out to be.

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" new <id> \
  --title "<verb phrase>" \
  --trigger "<the situation>" \
  --tags "<a,b>"
```

That writes a scaffolded file from the template. Fill it in:

- **`trigger`** is the field that decides whether this is ever found again.
  Write the *circumstance*, not a restatement of the title, and name the
  near-miss case if there is one.
- **`## Steps`** — exact commands, in the order that worked. Not the order they
  were tried in.
- **`## If it goes wrong`** — the failures actually hit in this session. This is
  the highest-value section and the one always omitted, because at the moment of
  writing the failure feels obvious and unrepeatable. It is neither.
- **`## Verification`** — an external signal. A URL that resolves, a row that
  exists, an artifact that appears.
- **`autonomy`** — default `confirm`. Choose `auto` only for something wholly
  reversible; choose `confirm-each` or `manual` if it touches money, production
  or other people.
- **`last_verified`** — today only if the procedure was genuinely run end to end
  in this session. If it was assembled from fragments, date it and say in
  `## Notes` that it has not been run as a whole.

**No secrets.** Reference credentials by name or `op://` path. Redact
identifiers that are personal rather than structural — an account number in a
step is almost always replaceable by "the account in the vault".

## 4. Reindex and validate

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" index
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" validate
```

`validate` catches the common slips: an id that does not match the filename, a
placeholder left in from the template, a missing `## Steps`, a duplicate id.
Clear every finding — it exits non-zero for a reason.

## 5. Commit

```bash
git -C "$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path)" add -A
git -C "$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path)" commit -m "Add SOP: <title>"
```

Push with `sop-sync` if there is a remote. Then show the user the path and the
`trigger` line — the trigger is the one field they should read back, because it
is the one that decides whether the next session finds this at all.
