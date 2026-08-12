---
id: publish-a-package
title: Publish a package and register it in the index
trigger: a package repo is finished and ready to be released publicly (not for internal-only builds, which stop after the tag)
tags: [release, publishing]
autonomy: confirm
frequency: occasional
last_verified: 2026-08-12
requires: [gh, npm]
---

> Example SOP, shipped with the plugin. It is not real — it exists to show the
> shape of the format. A real library's SOPs name real repos and hosts, which is
> why the library is private and this file is not in it.

## When this applies

A package in its own repo, tests green, ready for other people to install.

Not this: an internal build that only ever gets consumed from the registry cache
by our own services. Those stop at the tag and never touch the public index.

## Preconditions

- Working tree clean: `git status --porcelain` returns nothing.
- Logged in: `gh auth status` and `npm whoami` both succeed.
- The version in `package.json` is not already published:
  `npm view <name> versions --json`.
- `CHANGELOG.md` has an entry for this version. The release notes are generated
  from it, so an empty entry ships an empty release.

## Steps

1. Bump and tag — one command, so the tag can never disagree with the manifest:

   ```bash
   npm version minor      # or patch / major
   ```

2. Push the commit and the tag:

   ```bash
   git push --follow-tags
   ```

3. Wait for CI to go green before publishing. Publishing a version CI later
   rejects leaves a broken version that cannot be unpublished after 72 hours.

   ```bash
   gh run watch
   ```

4. Publish:

   ```bash
   npm publish --access public
   ```

5. Register it in the index repo — a new row in `registry.json`, then a PR.
   This is the step that is always forgotten, because the package works without
   it; only discovery breaks.

6. Refresh the search cache so the new version is findable today rather than at
   the next nightly crawl.

## Verification

- `npm view <name> version` returns the new version from a clean shell.
- The package page shows the new README, not the previous one.
- Searching the index for the package name returns it.

Step 6 is the one to actually check: it fails silently and everything else still
looks correct.

## If it goes wrong

**`npm publish` returns 403 with an otherwise valid login.** Almost always a
scope permission, not an auth failure. Check `npm access list packages` — the
message names the registry, which sends you looking in the wrong place.

**CI green, publish rejected as a duplicate version.** A previous run published
and then failed afterwards. `npm view <name> versions --json` and pick up from
step 5; do not bump again.

**The index PR merges but the entry does not appear.** The registry file is
cached for an hour. Wait, then refresh — do not open a second PR, which is what
it looks like it needs.

Steps 4 onward are **not reversible**. Unpublish is only available for 72 hours
and breaks anyone who installed in the meantime.

## Notes

The tag is created by `npm version` rather than by hand so that the manifest and
the tag cannot drift — they did drift once, and the resulting release pointed at
the wrong commit for a week without anyone noticing.

The index registration is deliberately a PR rather than a direct push: it is the
only step with a reviewer, and it is the one where a typo is invisible until
someone cannot find the package.
