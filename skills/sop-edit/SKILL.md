---
name: sop-edit
description: Amend an existing SOP — fix a step that has drifted, add a failure mode that was just hit, adjust its trigger or autonomy, or re-date last_verified after a successful run. Use when the user says an SOP is out of date or wrong, when following one produced a surprise, and routinely at the end of any SOP run that worked.
---

# Edit an SOP

Most edits here are small and routine. The frequent one — re-dating
`last_verified` after a clean run — needs no ceremony at all.

## Find it and read it whole

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" show <id>
```

Read the entire file before changing a line. A step that looks wrong in
isolation is often carrying a constraint stated three sections down, and
"simplifying" it removes the reason it exists.

## The four kinds of edit

### Re-date after a successful run

`last_verified: <today>`. Nothing else. Do this whenever the procedure ran as
written — it is a fact being recorded, not a change needing approval.

**Only when the procedure actually ran.** Editing the file is not verifying it.
Two dates that mean different things collapsing into one is how a library fills
up with confident, wrong instructions.

### Fix drift

A step no longer matches reality: a renamed flag, a moved endpoint, an extra
confirmation the tool now demands.

- Change the step, and re-date `last_verified` only if the corrected version was
  run.
- Say in `## Notes` what changed and when, if the old way might still be seen in
  the wild. A reader hitting the old behaviour needs to know it was expected.
- Do not leave the old step commented out. Git has it.

### Add a failure mode

Something went wrong; the fix is now known. It goes in `## If it goes wrong`,
and it is the most valuable thing that ever gets added to an SOP.

Write three things: **what it looked like**, **what it actually was**, and
**what fixed it**. The symptom is what the next reader will have; the cause is
what they will not be able to guess. Include the plausible cause that was ruled
out, if ruling it out cost time.

### Change the metadata

- **`trigger`** — the SOP was not found when it should have been, or was found
  when it should not. Widen or narrow the circumstance; adding the near-miss
  case usually fixes both directions at once.
- **`autonomy`** — a downgrade (`auto` → `confirm`) needs no discussion; make it.
  An upgrade needs the user to say so explicitly in this session. Do not infer
  "they never object" as consent.
- **`id`** — a rename. Rename the file to match, update anything in `supersedes`
  that points at it, reindex. `validate` will catch a half-done rename.

## Splitting instead of growing

An SOP that has grown two `## Steps` branches for two situations is two SOPs. It
was easier to append than to split, which is why it happened, but a branching
procedure fails the same way every time: the reader follows the wrong branch
confidently. Split it, cross-link in `## Notes`, and put both ids in
`supersedes` if the original is retired.

## Finish

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" index
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" validate
git -C "$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path)" commit -am "Update SOP: <id> — <what changed>"
```

Reindex whenever front matter changed; `INDEX.md` is generated and a stale one
misroutes the dispatcher. Show the user a before/after of the changed lines
only — not the whole file.
