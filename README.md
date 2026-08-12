# Claude-SOPs

A Claude Code plugin for keeping your own **standard operating procedures** — the
recurring things you do a particular way — as a small, versioned library of
markdown files that Claude consults on demand instead of working the sequence
out again every time.

The plugin is the machinery. **The procedures stay yours**: they live in a
private repo you own, they never enter this one, and they cost nothing in
context until one of them is actually needed.

## The problem

Some of what you want an assistant to know is not a capability, and not a
preference. It is a *procedure* — your way of doing a specific recurring job,
with the order that works, the step everyone forgets, and the failure that
looked like something else.

Two obvious places to put that, both of which fail:

**In `CLAUDE.md`.** It is loaded every session, so every procedure you add taxes
every conversation, including the several hundred that have nothing to do with
it. In practice this caps the collection at three or four before the file
becomes unusable.

**As skills, one per procedure.** Better, but every skill's name and description
is resident in every session too. Forty procedures is forty descriptions always
loaded. And skills sit in a tree you may share or sync; real procedures name
real hosts, repos, accounts and people, and are not shareable.

## The shape of the answer

One pointer, one index, one file read.

```
~/.claude/CLAUDE.md                one block: "recurring procedures live in ~/.claude-user-data/sops"
        │
        ▼
~/.claude-user-data/sops/INDEX.md  a generated table: id, title, "use when", verified date
        │
        ▼
~/.claude-user-data/sops/<id>.md   exactly one procedure, read only once it matched
```

Adding the fortieth SOP costs one row in a table. That is the whole design, and
everything else here follows from it.

The library is a **flat folder of readable markdown files**, one per procedure,
under `~/.claude-user-data/` — the convention this plugin uses for user content
that belongs to you rather than to the tool. Open it in any editor, read it
without Claude, grep it, edit it by hand. See
[`references/user-data-convention.md`](references/user-data-convention.md).

## Install

```
/plugin marketplace add danielrosehill/Claude-Code-Plugins
/plugin install claude-sops
```

Or clone this repo and add it as a plugin directory. No dependencies — the
tooling is one Python 3 standard-library script.

Then:

```
/sop setup
```

which creates `~/.claude-user-data/sops`, makes it a git repo, offers to push it to a
**private** GitHub repo so it mirrors across your machines, and adds a single
delimited block to your user-level `CLAUDE.md` — showing you the block and
asking first, because that file governs every future session.

## Use

Everything is reachable through `/sop`, or by just saying what you want.

| Say | What happens | Skill |
| --- | --- | --- |
| "run the SOP for publishing a plugin" | matches the index, opens one file, follows it | `sop-run` |
| "write this up as an SOP" | captures what just happened into a new procedure | `sop-write` |
| "the deploy SOP is out of date" | amends it, re-dates it | `sop-edit` |
| "what SOPs do I have?" | shows the index; flags the stale ones | `sop-list` |
| "we don't do that any more" | archives it, keeps the history | `sop-remove` |
| "sync my SOPs" | commits and pushes the library | `sop-sync` |

The useful habit is the second row. The best moment to write a procedure down is
immediately after working it out, while you still remember which of the four
things you tried was the one that worked.

## What an SOP looks like

```markdown
---
id: publish-a-package
title: Publish a package and register it in the index
trigger: a package repo is ready to be released publicly (not internal-only builds)
tags: [release, publishing]
autonomy: confirm
frequency: occasional
last_verified: 2026-08-12
---

## When this applies
## Preconditions
## Steps
## Verification
## If it goes wrong
## Notes
```

Two fields do most of the work:

**`trigger`** is matched by the dispatcher, so it decides whether a procedure is
ever found again. Write the *circumstance*, not a restatement of the title, and
name the near-miss case — "…ready to be released publicly (not internal-only
builds)" prevents a whole class of wrong match.

**`last_verified`** is set when the procedure was **run and seen to work**, not
when the file was edited. The index flags anything unverified for over a year.
Keeping those two dates distinct is what stops the library filling up with
confident, wrong instructions.

Full spec: [`references/sop-format.md`](references/sop-format.md). Worked
examples: [`examples/`](examples/).

## Where things live

| | Path | Lifecycle | Privacy |
| --- | --- | --- | --- |
| Plugin | wherever Claude Code installed it | replaced on `/plugin update` | public |
| Library | `~/.claude-user-data/sops/` (configurable) | your git repo, diverges forever | **private** |
| Vault | `~/.claude-plugins/claude-sops/` | survives updates, committed nowhere | local, never synced |

```
~/.claude-user-data/           private git repo, mirrored across your machines
└── sops/
    ├── INDEX.md               generated
    ├── README.md
    ├── deploy-the-site.md     one procedure, one file
    ├── close-the-month.md
    └── archive/               retired procedures
```

`INDEX.md` and `README.md` are reserved names; everything else ending in `.md`
is a procedure. Nothing is ever written into the plugin directory at run time —
that works until the first update and then loses data silently. Details, and how
to move the library: [`references/library-layout.md`](references/library-layout.md).

## Tooling

```bash
python3 scripts/sop.py path        # where is the library
python3 scripts/sop.py repo        # which git repo it sits in
python3 scripts/sop.py list        # what is in it
python3 scripts/sop.py new <id> --title "…"
python3 scripts/sop.py index       # regenerate INDEX.md
python3 scripts/sop.py validate    # front matter, ids, placeholders, duplicates
python3 scripts/sop.py retire <id>
```

`INDEX.md` is generated. Edit the SOPs and regenerate; hand edits are lost.

## A note on autonomy

Each SOP declares how much of itself may run without asking — `auto`, `confirm`
(the default), `confirm-each`, `manual`. That field records **a default, not a
licence**. It does not authorise something you have declined in the session, and
it does not exempt irreversible or outward-facing steps from confirmation. It
exists so that a wholly reversible procedure does not interrupt you six times,
and so that one touching money or production interrupts you at every step.

## Licence

MIT. See [LICENSE](LICENSE).
