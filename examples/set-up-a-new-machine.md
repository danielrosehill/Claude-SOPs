---
id: set-up-a-new-machine
title: Bring a new workstation up to a working state
trigger: a fresh machine, a reinstall, or a rebuild after a disk failure
tags: [workstation, setup]
autonomy: confirm-each
frequency: rare
last_verified: 2026-08-12
requires: [git, gh, op]
---

> Example SOP, shipped with the plugin. Not real — it shows what a rare,
> high-consequence procedure looks like in this format.

## When this applies

Any machine that will do real work: a new laptop, a reinstall, a rebuild after
hardware failure.

Not this: a throwaway VM or a container. Those get the container image, and
following this procedure on one wastes an hour and installs a credential agent
that has no business being there.

## Preconditions

- The recovery phrase for the password manager is to hand, **on paper**. Nothing
  below works without it, and there is no reset.
- The backup destination is reachable and its last snapshot is recent — check
  before wiping anything, not after.
- A second device is available to approve the MFA prompts.

## Steps

Order matters up to step 4; after that it does not.

1. Sign in to the password manager and unlock the vault. Everything downstream
   reads credentials from it.
2. Generate a new SSH key **on this machine** and add it to the account. Do not
   copy a key across from the old one.
3. Install the CLI tooling. Authenticate each tool as it is installed, so a
   failure is attributable.
4. Clone the dotfiles repo and run its install script.
5. Clone the SOP library, and everything else, in any order.
6. Restore data from the most recent snapshot. Verify a restored file opens
   before deleting anything on the source.

## Verification

- A commit pushed from this machine appears with the new key's signature.
- The password manager CLI reads a test secret without prompting for the phrase
  a second time.
- A file restored from backup opens and matches its checksum.

## If it goes wrong

**The SSH key is added but pushes are still rejected.** The key was added to the
wrong account — personal instead of work, or the reverse. `ssh -T git@<host>`
names the account it authenticated as; that is the fastest way to see it.

**The password manager CLI asks for the recovery phrase repeatedly.** The
desktop app is not running, so the CLI cannot use the system integration. Start
the app once and it stops.

**The dotfiles install script half-runs and exits.** It is not idempotent past
the shell change. Re-running from the top will not recover it; fix the failing
step and run the remaining sections by hand.

Step 6 is the irreversible one. Nothing is deleted from the old machine or the
backup source until a restored file has been opened and checked.

## Notes

`autonomy: confirm-each` on purpose. Every step here either creates a
credential, changes an account, or moves data, and the whole procedure runs
maybe twice a year — there is no efficiency to be gained by batching the
confirmations, and a wrong step is expensive.

The key is generated fresh rather than copied so that decommissioning the old
machine is a single revocation with no shared material left behind.
