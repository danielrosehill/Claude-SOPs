# Where everything lives

Three locations, three lifecycles. Putting a file in the wrong one fails in a
predictable way, so the split is worth knowing before writing anything.

| | Path | Lifecycle | Privacy |
| --- | --- | --- | --- |
| **Plugin** | wherever Claude Code installed it | replaced wholesale on `/plugin update` | public |
| **Library** | `~/claude-sops` (configurable) | a git repo you own; diverges forever | private |
| **Vault** | `~/.claude-plugins/claude-sops/` | survives updates, committed nowhere | local |

**Never write into the plugin directory at run time.** It works until the first
update and then loses the data silently.

## The library

```
~/claude-sops/
├── INDEX.md          generated — the only file read by default
├── README.md         from templates/library-README.md at init
├── sops/
│   ├── deploy-the-site.md
│   └── close-the-month.md
└── archive/
    └── 2026-03-04-old-deploy.md
```

Its own git repo, private, pushed to a remote so it can be cloned onto every
machine that runs Claude Code. It is deliberately **not** inside `~/.claude`:
that directory is frequently backed up or synced wholesale by other tooling, and
a git repo nested inside a synced tree is a recurring source of confusing
conflicts.

Resolution order for the path, highest first:

1. `$CLAUDE_SOPS_DIR`
2. `library` in `~/.claude-plugins/claude-sops/config.json`
3. `~/claude-sops`

`python3 scripts/sop.py path` prints the winner. Use it rather than assuming —
on a machine where the user keeps the library somewhere else, a hardcoded
`~/claude-sops` silently creates a second, empty library.

## The vault

```json
{
  "library": "~/claude-sops",
  "repo": "<owner>/claude-sops",
  "remote": "github",
  "claude_md": "~/.claude/CLAUDE.md",
  "seeded": "2026-08-12"
}
```

Machine-specific truth, nothing else. No SOP content, no credential values —
credentials are referenced by name or `op://` path and resolved at use.

Rules that apply to it:

- **Read it always, write it only when asked**, and say the path when you do.
- **Never fail because it is missing.** Absent vault means defaults; a first run
  works with no setup at all.

## The `CLAUDE.md` snippet

One delimited block in the user's user-level `CLAUDE.md`, so that a session
which never touches an SOP still knows the library exists. Exact text and the
rules for editing it: [`claude-md-snippet.md`](claude-md-snippet.md).

## Why the split exists

The library could have been skills — one skill per procedure, in `~/.claude/skills/`.
That fails on two counts.

**Context.** Every skill's name and description is loaded every session. Forty
procedures is forty descriptions permanently resident, and most sessions use
none of them. One index file read on demand costs nothing until it is read.

**Privacy.** Real procedures name real repos, hosts, accounts and people. They
are not shareable and should not live in the same tree as things that are. The
plugin is public because the machinery generalises; the library is private
because the procedures do not.
