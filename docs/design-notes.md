# Design notes

Decisions taken while building this, and the constraints that settled them.
Written 2026-08-12.

## Why a library and not skills

Skills are the obvious home for "a procedure Claude should follow", and were
rejected for two reasons.

**Context cost is per-session and unavoidable.** Every skill's `name` and
`description` is loaded into every session, whether or not it is used. That is
fine for a capability that might apply anywhere and terrible for a procedure
that applies four times a year. A library of forty procedures as forty skills is
forty permanently resident descriptions; as an index it is one file that is read
only when something plausibly matches.

**Skills are shareable; procedures are not.** A real SOP names hosts, repos,
account identifiers and people. It belongs in a private repo, not in a tree that
gets published, synced or copied between contexts.

The nesting — one dispatcher skill, one index, one file read — is the standard
answer to both. It is the same reason `references/` exists in a plugin rather
than everything being inlined into `SKILL.md`.

## Why `trigger` is separate from `title`

The dispatcher matches a situation, not a name. Early drafts had only a title,
and the failure mode was immediate and consistent: titles are written from the
author's point of view ("Deploy the site") and the reader arrives with a
circumstance ("I've merged and it needs to go out"). Two SOPs whose titles are
near-synonyms are indistinguishable; their triggers rarely are.

The schema description says "write the circumstance, not the action" because
without that instruction roughly every first draft of a `trigger` restates the
title and matches nothing new.

## Why `last_verified` and not `updated`

A file's modification date is free and useless. The question that matters when
following a procedure is *was this seen to work*, and editing a file is not
running it. Collapsing the two produces a library where everything looks fresh
and some of it is wrong — the worst state, because staleness is at least visible.

The cost is that it must be re-dated deliberately, so `sop-run` closes its loop
by re-dating on success and explicitly *not* re-dating on failure, and `sop-list`
refuses to re-date during a review pass.

`frequency` exists to calibrate this. Age alone is a weak signal: a monthly
procedure unverified for eight months is suspicious, a rare one unverified for
three years is exactly what the library is for.

## Why `autonomy` is a field and not a policy

The alternative was a single plugin-wide setting. It does not survive contact
with a real library, which contains both "regenerate the index" and "issue
refunds". Per-SOP is the only granularity that matches how the procedures
actually differ.

It is deliberately framed everywhere as a *default, not a licence*: it records
how much interruption a procedure normally warrants, and never widens what the
user has permitted in the session. An SOP is a note the user wrote to themselves;
it should not be able to grant permissions by being written confidently.

## Why the index is generated

Two failure modes were worth engineering out. A hand-maintained index drifts
from the files beside it and the drift is invisible — the dispatcher stops finding
a procedure that is right there. And a hand-maintained index conflicts on every
merge in a repo explicitly designed to be edited from several machines.

Generated means the conflict resolution is "take either side and regenerate",
which is stated in `sop-sync` because it is not obvious in the moment.

## Why local scaffolding instead of a GitHub template repo

This plugin follows the
[Plugin–Workspace–Vault](https://github.com/danielrosehill/Claude-Plugin-Workspace-Vault)
split, with one deliberate deviation: PWV instantiates the workspace from a
public **template repo**, and this plugin scaffolds it locally instead
(`sop.py init`, then `git init`, then optionally `gh repo create --source`).

The reason is that the workspace here is three files and a directory. A template
repo would add a second repo to maintain, a network dependency to first run, and
a `--template` default that a fork would have to override — all to avoid writing
a README and two `mkdir`s. The PWV rationale for a template repo is a workspace
substantial enough to diverge meaningfully from its starting point; that is not
this. The other two legs are unchanged.

Consequence worth knowing: there is no upstream to sync a library from, so
improvements to `templates/library-README.md` reach existing libraries only if
someone copies them over. `sop.py init` never overwrites an existing README.

## Why `~/.claude-user-data/sops/`, flat

Three constraints, and only one layout satisfies all of them.

**Not `~/.claude/sops`.** That tree is routinely backed up, synced or committed
wholesale by other tooling. A git repo nested inside a synced tree produces
conflicts that are tedious to diagnose and easy to blame on the wrong thing.

**Not a bare `~/sops`.** It collides with
[Mozilla SOPS](https://github.com/getsops/sops) in muscle memory and
tab-completion.

**Flat, not `sops/sops/`.** The first draft had the library root hold `INDEX.md`,
`README.md` and a `sops/` subdirectory. That is tidier in the abstract and worse
in practice: the point of markdown files over a database is that a person can
open the folder and see their procedures, and an extra level of nesting between
the folder you `cd` into and the files you want is friction for no gain at this
size. The cost is two reserved filenames — `INDEX.md` and `README.md` cannot be
SOP ids — which `sop.py` enforces at `new` rather than discovering at `index`.

**One root for all user content.** `~/.claude-user-data/` is a single private
repo, one subfolder per tool, so mirroring to another machine is one clone rather
than a repo per tool with a remote each. Written up as a general convention in
`references/user-data-convention.md`, because the next tool that needs to save
something should not have to re-decide this.

The consequence that catches tooling out: **the library folder is not the git
repo root.** `git -C <library>` still works, but `git add -A` at the root would
sweep in another tool's half-finished folder under a commit message about SOPs.
Hence `sop.py repo`, and hence every skill staging `$LIB` explicitly.

## Why the CLAUDE.md block is delimited and backed up

Idempotence: a block between markers can be replaced, so re-running setup after
moving the library updates the path instead of appending a second, contradictory
block. The single-marker case is treated as "hand-edited, stop and ask" rather
than guessed at, because guessing the end of a block in someone's most carefully
tuned file is not a risk worth taking for the convenience saved.

The `.bak-<date>` copy is cheap insurance on the same file.

## The front-matter parser

`sop.py` parses YAML front matter itself rather than depending on PyYAML,
because a zero-install script is worth more here than full YAML support. It
handles scalars, quoted scalars, inline `[a, b]` lists and block `- item` lists —
the subset the format actually uses. Anything else is rejected by `validate`
rather than half-parsed, which is the important property: a silently
misunderstood field would produce a wrong index, and a wrong index is the one
failure this whole structure exists to prevent.

If the format ever needs nested structures, that is the moment to take the
dependency, not before.
