---
name: sop-sync
description: Commit and push the SOP library, or pull it down, so the same procedures are available on every machine. Use when the user asks to sync, save, push or back up their SOPs, when setting up a second machine, and at the end of any session that added or changed a procedure.
---

# Sync the library

The library is an ordinary private git repo. Nothing clever happens here — but
it is the step that makes the collection worth maintaining, because a procedure
that exists on one laptop is a note, not a system.

```bash
LIB="$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path)"
ROOT="$(python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" repo)"
```

**`$ROOT` is not `$LIB`.** The repo is `~/.claude-user-data` and the library is
the `sops/` folder inside it, shared with whatever else follows the convention.
`sop.py repo` exits 1 if there is no repo at all — that means `sop-setup` step 2
was never done, not that something broke.

## Push

```bash
git -C "$ROOT" status --short
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" index
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" validate
git -C "$ROOT" add -- "$LIB"
git -C "$ROOT" commit -m "<what changed>"
git -C "$ROOT" push
```

**Stage the library path, not `-A`.** A blanket add at the root sweeps in
whatever another tool left half-finished in its own folder, under a commit
message about SOPs. If `git status` shows changes outside `$LIB`, mention them
and leave them alone.

Reindex before committing, always. `INDEX.md` is generated, and a commit whose
index disagrees with the files beside it misroutes the dispatcher on the *other*
machine, where nobody will connect the two.

Write a real commit message — "Add SOP: close the month", not "update". The log
of this repo is a usable history of how the user's procedures changed, and one
line of care per commit is what keeps it that way.

**Validate before pushing**, not after. A broken front matter block propagates
to every machine and is then found on the one where it is least convenient.

## Pull

```bash
git -C "$ROOT" pull --ff-only
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" validate
```

`--ff-only` on purpose: a merge here is almost always two machines that both
edited the same SOP, and that wants a human eye, not an automatic three-way
merge of prose.

## Conflicts

Two shapes, one of them trivial:

**`INDEX.md` conflicts.** Never resolve it by hand. Take either side and
regenerate:

```bash
git -C "$ROOT" checkout --ours -- "$LIB/INDEX.md"
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" index
git -C "$ROOT" add -- "$LIB/INDEX.md"
```

**An SOP conflicts.** Show the user both versions of the conflicting section and
ask. Prose merges cleanly at the text level and incoherently at the meaning
level — two half-correct procedures spliced together is worse than either. If
both sides are genuinely different procedures that grew apart, that is a split,
not a merge; keep both under different ids.

Take the later `last_verified` when the bodies are otherwise identical: it means
one machine ran the procedure and the other did not.

## No remote yet

Perfectly normal, and local git alone is worth having. Offer the remote once,
show the command, and do not ask again in the same session:

```bash
gh repo create claude-user-data --private --source="$HOME/.claude-user-data" --push
```

Private. The contents name real hosts, repos and accounts.

## Reporting

Say what moved: how many files, which ids, and whether the push succeeded. If
nothing changed, say that in one line rather than narrating a clean status.
