---
name: sop-setup
description: Set up the SOP library for the first time — create the folder, make it a private git repo, record the path in the vault, and add the pointer block to the user's CLAUDE.md. Also the skill for moving the library somewhere else, or cloning an existing SOP repo onto a second machine. Use when the user asks to start keeping SOPs, when another sop-* skill finds no library, or when they mention mirroring their procedures across machines.
---

# Set up the SOP library

Four things, in this order. Each is separately useful — a user who wants only a
folder should get only a folder.

1. Create the library
2. Put it under git, and optionally on a private remote
3. Record the path in the vault
4. Add the pointer block to `~/.claude/CLAUDE.md`

Layout and lifecycles: [`references/library-layout.md`](../../references/library-layout.md).
Why the paths are what they are:
[`references/user-data-convention.md`](../../references/user-data-convention.md).

## 0. Check whether one already exists

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" path
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" repo    # exits 1 if not in a repo
```

If that directory exists and has `.md` files in it, there is already a library.
Do not re-init — say what was found and ask what they actually wanted. The common
real cases are *set it up on this second machine* (clone the repo, jump to step
3) and *move it* (below).

## 1. Create it

Default `~/.claude-user-data/sops` — the convention for user content, one flat
folder of readable markdown files. Ask before choosing something else only if
they have already hinted at a preference; otherwise take the default and say so.

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/scripts/sop.py" init
```

That makes the folder, `archive/`, a `README.md` and an empty `INDEX.md`. It is
idempotent and never overwrites an existing README.

Do not put it inside `~/.claude`. That tree gets synced and backed up wholesale
by other tooling, and a nested git repo there produces conflicts that are
tedious to unpick.

## 2. Version control

**The repo is `~/.claude-user-data`, not the `sops/` folder inside it.** That
root is the mirror point for every tool that follows the convention; a repo per
subfolder means a remote and a push to remember for each one.

Local git first — that alone gives history and is worth doing even if they never
push:

```bash
git -C ~/.claude-user-data init
git -C ~/.claude-user-data add -A
git -C ~/.claude-user-data commit -m "Start SOP library"
```

If `sop.py repo` already prints a path, the root is under git — skip the `init`
and just commit.

Then the remote, **if they want one**. This is what makes the collection mirror
across machines, and it is also the step that creates something outward-facing —
so show the exact command and get an explicit yes before running it:

```bash
gh repo create claude-user-data --private --source="$HOME/.claude-user-data" --push
```

- **Private, always.** Real SOPs name real hosts, repos and accounts. If the user
  asks for public, say plainly what the file contents imply and let them decide.
- `--source` + `--push` needs the local repo to exist and have a commit already,
  which is why step 2 runs in this order.
- No `gh`? Say so, leave the local repo in place, and tell them exactly what to
  run once they have it. A library with no remote is fully functional.

On a **second machine**, this step is instead:

```bash
gh repo clone <owner>/claude-user-data ~/.claude-user-data
```

## 3. Record it in the vault

Only needed when the path is not the default, but write it anyway — it makes the
next skill's lookup unambiguous.

```
~/.claude-plugins/claude-sops/config.json
```

```json
{
  "library": "~/.claude-user-data/sops",
  "repo": "<owner>/claude-user-data",
  "remote": "github",
  "claude_md": "~/.claude/CLAUDE.md",
  "seeded": "<YYYY-MM-DD>"
}
```

Create the directory if it is missing. Tell the user the path you wrote. Nothing
under `~/.claude-plugins/` is ever committed anywhere.

## 4. Seed the pointer into CLAUDE.md

The block, and the rules for editing it, are in
[`references/claude-md-snippet.md`](../../references/claude-md-snippet.md). The
parts that matter here:

- Back up to `~/.claude/CLAUDE.md.bak-<date>` first.
- If both `<!-- claude-sops:begin -->` and `<!-- claude-sops:end -->` are
  present, replace between them. Never append a second block.
- If exactly one marker is present, stop — the file has been hand-edited. Show
  the user and let them place it.
- **Show the block and ask before writing.** This file governs every future
  session; it is not an edit to make on inference.
- Substitute the real library path in both places it appears.

Then verify:

```bash
grep -c 'claude-sops:begin' ~/.claude/CLAUDE.md   # must be exactly 1
```

## Moving an existing library

```bash
git -C ~/.claude-user-data remote -v          # confirm the remote first
git -C ~/.claude-user-data mv sops <new-path>
```

Moving it *within* the repo is a `git mv`. Moving it out of the repo entirely
means the new location needs its own git, and the old repo keeps the history —
say which of the two is happening.

Then update `library` in the vault, and update the path **in both places** in
the `CLAUDE.md` block. A moved library with a stale snippet is the failure that
looks like the plugin not working: the index reads as empty, and an empty index
is indistinguishable from having no procedures.

## Finishing

Tell them what exists now, in one short list: the path, whether there is a
remote, and that `/sop` will find it. Then offer to write the first SOP — the
best moment to capture one is right after a procedure has been worked out, and
there is usually one in the session that led here.
