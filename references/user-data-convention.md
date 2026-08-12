# `~/.claude-user-data/` — the convention

Content produced by or for a person, through Claude tooling, goes in a flat
readable folder under:

```
~/.claude-user-data/<tool>/
```

For this plugin that is `~/.claude-user-data/sops/`, holding one markdown file
per procedure. Not a database, not a nested tree, not a serialised blob — files
you can open in any editor, read without Claude running, grep, and edit by hand.

The root honours `$CLAUDE_USER_DATA` if it is set, so a machine that already
points other plugins somewhere keeps everything together:

```
${CLAUDE_USER_DATA:-$HOME/.claude-user-data}/<tool>/
```

Note the default differs from the older resolver some plugins use
(`${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins`). That path is
XDG-correct and effectively invisible — nobody browses `~/.local/share` — which
is the opposite of the goal here. Setting `$CLAUDE_USER_DATA` reconciles the two
in one place.

## The rule that decides the folder

Two kinds of thing get written outside a plugin, and they want opposite
treatment. The question is not *what is it* but *would you want it on your other
machine*.

| | Goes in | Mirrored? | Example |
| --- | --- | --- | --- |
| **User content** — the thing you would be annoyed to lose | `~/.claude-user-data/<tool>/` | yes, via a private git repo | SOPs, notes, drafts, saved research |
| **Machine config** — how this box is wired up | `~/.claude-plugins/<plugin>/` | no, never committed | paths, which account, which printer, `op://` references |
| **Anything the tool owns** | the plugin directory | replaced on `/plugin update` | skills, scripts, templates |

Writing a file into the plugin directory at run time works exactly until the
first update, then loses it silently. That is the one genuinely destructive
mistake of the three.

## Why a single root

`~/.claude-user-data/` is one private git repo. Each tool takes a subfolder;
mirroring to a second machine is one clone, and adding a tool adds a directory
rather than a new repo, a new remote and a new thing to remember to push.

It also keeps the user data out of `~/.claude/`, which is routinely backed up,
synced or committed wholesale by other tooling. A git repo nested inside a
synced tree produces conflicts that are tedious to diagnose and easy to blame on
the wrong thing.

## What this means for tooling

**Never assume the library folder is the git repo root.** With this layout it is
not — the repo is `~/.claude-user-data/` and the library is a subfolder of it.
Resolve it:

```bash
python3 scripts/sop.py repo    # git rev-parse --show-toplevel, or exit 1
```

**Stage narrowly.** A sync that runs `git add -A` at the repo root sweeps in
whatever another tool left half-finished in its own folder. Stage the paths you
changed, and mention other pending changes rather than committing them.

**Reserved filenames.** Because the files sit directly in the folder,
`INDEX.md` and `README.md` are structural and can never be procedure ids. The
tooling enforces this; `sop.py new index` is refused rather than silently
shadowing the index.

**Human-first naming.** The filename is the id is the thing you would type. No
timestamps, no hashes, no `sop_0007.md` — the folder is meant to be browsable by
a person who has never used the plugin.
