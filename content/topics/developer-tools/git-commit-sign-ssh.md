+++
type = "article"
title = "Sign Git Commits With SSH Keys"
description = "Git & GitHub Setup for Signed Commits & Verification"

slug = "git-commit-sign-ssh"

tags = ['Git', 'GitHub']

date = 2026-08-07
+++

## Background

As companies and organizations harden their security posture,
it is common to require Git commits to be signed and verified with a cryptographic key function.

Git was originally designed only to support PGP/GPG\* key signing,
but GPG has fallen out of favor due to [numerous security and usability flaws](https://news.ycombinator.com/item?id=46403200).
On the flipside, most engineers already have a secure and easy-to-use cryptographic auth workflow: SSH keys.

SSH has a true "set it and forget it" configuration -
do it once (per machine) and then the mechanism disappears into the background.
Signed Git commits with SSH can be the same way -
but first we have to work through the awkward way in which Git has added support for SSH key cryptography.

_\*PGP is the specification, GPG is the de facto standard open-source implementation.
The terms are used interchangeably._

## Goals

We will:
1. Configure Git to sign commits automatically with an SSH key
2. Check local commit signatures and understand commit verification messages
3. Configure GitHub to verify commits signed with an SSH key

## 0. Prerequisites

### 0.1 Install Git

Most machines will already have `git` installed,
but the Git official docs provide [install instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) as well.

### 0.2 Generate an SSH Key

The GitHub docs provide a [complete guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
for generating an SSH key and adding it to the SSH agent.

## 1. Configure Git Signing for Commits & Tags

### 1.1 Set Git User Name and Email

Check `git config list` - if the user name and email are not set, add them now:
```shell
git config --global user.name 'Foo Bar'
git config --global user.email 'foo@bar.net'
```

### 1.2 Configure Git to Sign With SSH

```shell
git config --global gpg.format ssh
```
SSH is _not_ a format of PGP/GPG, despite the name of the config option.
When Git added support for additional signing "backends",
the new options were shoved into the existing config namespace.

### 1.3 Set the Default Signing Key

Point Git at the local SSH private key:
```shell
git config --global user.signingkey ~/.ssh/id_ed25519
```

### 1.4 Configure Git to Sign Commits by Default

Without this, we have to use the flag each time via `git commit -S/--gpg-sign`:
```shell
git config --global commit.gpgsign true
```

This `commit.gpgsign` config or `-S/--gpg-sign` flag adds the cryptographic signature
which is not visible in the commit message.

Note that this does *not* add the standard text signoff trailer to the commit message,
which looks like `Signed-off-by: Foo Bar <foo@bar.net>`.
That trailer can be added with the `-s/--signoff` flag.
While some repositories may require the trailer, it is only text - not cryptographic.

## 2. Sign & Verify a Commit Locally

Before we start, we must temper expectations -
we will not _successfully_ verify the signatures on our local machine.

Local commit verification requires an "allowed signers" config file,
which is an artifact of when Git was primarily used without centralized hosted “forges” like GitHub or GitLab.
The [Git documentation](https://git-scm.com/docs/git-config#Documentation/git-config.txt-gpgsshallowedSignersFile)
and the `ssh-keygen(1)` man page for the "allowed signers" file format are characteristically opaque and unhelpful,
and we have no need for full local key verification, so we can skip it for now.

Do not fear!
We can still use the error messages for different types of local commit verification to validate our configuration.

### 2.1 Create Different Types of Signed Commits

We will create a few commits signed a few different ways to see the different verifications,
with `--allow-empty` flag to avoid needing to create and stage actual changes each time.

First, use the `--no-gpg-sign` flag to create a completely unsigned commit,
even with the `commit.gpgsign` config set to `true`:
```shell
git commit --allow-empty --no-gpg-sign -m "No signature"
```

Next, demonstrate the text-only signoff trailer with `-s/--signoff`:
```shell
git commit --allow-empty --no-gpg-sign --signoff -m "Signoff trailer, no crypto signature"
```

Finally, create a cryptographically signed commit -
the default behavior with `commit.gpgsign` set to `true`:
```shell
git commit -m "Signed with SSH key: no signoff trailer"
```

### 2.2 Verify the Commit Signature Locally

Apply local commit verification to the git log with the `--show-signature` option:
```shell
git log --show-signature
```

The varying levels of error messages (or lack thereof) may be surprising.

#### Unsigned Commits
Commits which are _not_ signed with a cryptographic key will not show any error -
there was no attempt at a signature, so there is nothing to verify.
This will be the case in the `git log --show-signature` output
for the `No signature` and `Signoff trailer, no crypto signature` commits:
```shell
commit 87ff31fac294943a8c8ffc838a5a7fbc647cac43
Author: Foo Bar <foo@bar.net>
Date:   Fri Aug 7 20:49:15 2026 -0700

    Signoff trailer, no crypto signature

    Signed-off-by: Foo Bar <foo@bar.net>

commit 1e5d69bc336d7777b34408cf020ab0c64d3ab526
Author: Foo Bar <foo@bar.net>
Date:   Fri Aug 7 20:47:48 2026 -0700

    No signature
```

#### Signed Commits Without Allowed Signers Config

The commit which _was_ signed with the SSH key will actually show an error,
as we have skipped configuring the local "allowed signers" file as [mentioned above](#2-sign--verify-a-commit-locally).

We can ignore the fact that this verification failed.
Because unsigned commits do not show any error,
just the presence of these error messages is enough to determine that the commit is signed with _some_ key.

In addition to logging an `gpg.ssh.allowedSignersFile` error,
`git log --show-signature` will display a `No signature` error with the commit:
```shell
error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification
commit 6784531c0a6a8e7c6b49161115ed19c6504af95f (HEAD -> main)
No signature
Author: Foo Bar <foo@bar.net>
Date:   Fri Aug 7 19:21:22 2026 -0700

    Signed with SSH key: no signoff trailer
```

#### Bonus: Signed Commits With an Unknown GPG Key

One final case we may see while experimenting is the failed verification of a GPG-signed commit.
This is a much more visible error, prefixed with `gpg`:

```shell
commit 5a13dda67abeaa0b036b874fd53c6e8117cf8c39 (origin/main, origin/HEAD)
gpg: Signature made Tue 14 Jul 2026 11:24:18 AM PDT
gpg:                using RSA key B5690EEEBB952194
gpg: Can't check signature: No public key
Author: Foo Bar <foo@bar.net>
Date:   Tue Jul 14 11:24:18 2026 -0700

    Initial commit
```

GPG is the original signing mechanism for Git, and it will fail with this message
rather than the `allowedSignersFile` and `No signature` messages for failed SSH verification.

The above case is from the standard "Initial commit" created when initializing a repo in the GitHub UI.
It was signed by GitHub's own GPG key, but the same effect will appear for any commit
for which the signer's GPG public key is not registered with the local keyring.

## 3. Configure GitHub to Accept SSH Key Signatures

### 3.1 Require Signed Commits
In order to verify GitHub's acceptance of the signed commits,
we must have a branch on a repository protected by a "Require signed commits" rule.

1. Go to the **Settings** tab on a repository
2. Navigate to **Rulesets**
3. Click the **New ruleset** dropdown and choose **New branch ruleset**
4. Give the ruleset a name
5. Under **Branch targeting criteria**, click **Add target** and select a branch target
6. Under **Branch rules**, select the **Require signed commits** checkbox
7. Scroll down to the bottom and click **Create**, or **Save changes** if editing an existing ruleset

### 3.2 Verify GitHub Rejects a Signed Commit With an Unregistered Key

At this point GitHub should reject the commit,
assuming the SSH key has not been registered as a _signing_ key.
Registering it as an authentication key is a step almost everyone completes for a new SSH key,
but authentication keys and signing keys are distinct concepts.

We can verify this and see what the error looks like.
To simplify, we just want to push a single commit signed with the SSH key.
We can delete the previous empty example commits and recreate the single signed one:

```shell
git reset --hard HEAD~3
git commit --allow-empty -m "Signed with SSH key: no signoff trailer"
```

Make sure the new commit is on the same branch that is protected by the ruleset on GitHub.

Now try to push:
```shell
% git push
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 424 bytes | 424.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: Review all repository rules at https://github.com/scratchbuffer/git-test/rules?ref=refs%2Fheads%2Fmain
remote:
remote: - Commits must have verified signatures.
remote:   Found 1 violation:
remote:
remote:   78d74bf3876e0c8f749a0be785a8df0aab486118
remote:
To github.com:scratchbuffer/git-test.git ! [remote rejected] main -> main (push declined due to repository rule violations)

error: failed to push some refs to 'github.com:scratchbuffer/git-test.git'
```

### 3.3 Register the SSH Key as a Signing Key

1. Click the GitHub profile picture in the top right corner
2. Choose **Settings** from the dropdown
3. Navigate to **SSH and GPG keys**
4. Click **New SSH key**
5. In the **Add new SSH Key** form:
   1. Give it a title
   2. Select **Key type** as **Signing Key**
   3. Paste the SSH public key
6. Click **Add SSH key** to finish

### 3.4 Verify GitHub Accepts the Signed Commit With the Registered Signing Key

Now we should be able to push:
```shell
% git push
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 424 bytes | 424.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:scratchbuffer/git-test.git
   5a13dda..78d74bf  main -> main
```

## Q.E.D.

We are good to push!

The SSH-based setup keeps Git commit signing nice and simple without the pain of GPG keys
so we can satisfy the security checklist and get back to engineering.

If you are interested in learning about the Allowed Signers file... best of luck.
The Git documentation points to the `ssh-keygen(1)` manual page,
which itself says to go read the `sshd(8)` and `ssh_config(5)` pages,
and so on and so forth down the rabbit hole.
You may even enjoy yourself!
After all, not all who wander are lost.
