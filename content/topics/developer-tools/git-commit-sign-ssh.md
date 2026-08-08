+++
type = "article"
title = "Sign Git Commits With SSH Keys"
description = "Git & GitHub Setup for Signed Commits & Verification"

slug = "git-commit-sign-ssh"

tags = ['Git', 'GitHub', 'GitLab']

date = 2026-08-07
+++

## 0. Prerequisites

### 0.1 Install Git

Most machines will already have `git` installed,
but the Git official docs provide [install instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) as well.

### 0.2 Generate an SSH Key

The GitHub docs provide a [complete guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
for generating an SSH key and adding it to the SSH agent.

## 1. Configure Git Signing for Commits & Tags

### 1.1 Set Git User Name and Email

Check `git config list` - if user name and email are not set, add them now:
```shell
git config --global user.name 'Foo Bar'
git config --global user.email 'foo@bar.net'
```

### 1.2 Configure Git to Sign With SSH

```shell
git config --global gpg.format ssh
```
SSH is _not_ a format of GPG, despite the name of the config option.
Git added support for further signing "backends" after initially only designing for GPG,
so the other backend options got shoved into the existing config namespace.

### 1.3 Set the Default Signing Key

Point git at the SSH public key:
```shell
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```


### 1.4 Configure Git to Sign Commits by Default

Without this, we have to use the flag each time via `git commit -S/--signoff`:
```shell
git config --global commit.gpgsign true
```

This `commit.gpgsign` config or `-S/--gpg-sign` flag adds the cryptographic signature
which is not visible in the commit message.

Note that this does *not* add the standard text signoff trailer to the commit message,
which looks like `Signed-off-by: Foo Bar <foo@bar.net>`.
That trailer can be added with the `-s/--signoff` flag.
While some repositories may require the trailer, it is only text - not cryptographic

## 2. Sign & Verify a Commit Locally

### 2.1 Create Different Types of Signed Commits

We will create a few commits signed a different ways to see the different verifications.
Use the `--allow-empty` flag to avoid need to stage changes each time.

First, we can us the `--no-gpg-sign` flag to create a completely unsigned commit,
even with the `commit.gpgsign` config set to `true`:
```shell
git commit --allow-empty --no-gpg-sign -m "No signature"
```

Then we can demonstrate the text-only signoff trailer with `-s/--signoff`:
```shell
git commit --allow-empty --no-gpg-sign --signoff -m "Signoff trailer, no crypto signature"
```

Finally create a cryptographically-signed commit -
the default behavior with `commit.gpgsign` set to `true`:
```shell
git commit -m "Signed with SSH key: no signoff trailer"
```

### 2.2 Verify the Commit Signature Locally

```shell
git log --show-signature
```

The varying levels of signature verification messages visible may be unexpected.

#### Unsigned Commits
Commits which are _not_ signed with an SSH will not show any error -
there was no attempt at a cryptographic signature so there is nothing to verify.
This is the case for the `No signature` and `Signoff trailer, no crypto signature` commits:
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

The commit which _was_ signed with the SSH key will actually show an error.
Local commit signature verification requires an "allowed signers" config file -
an artifact of when git was primarily used without centralized hosted "forges" like GitHub or GitLab.

For now we can ignore this.
The error messages are enough to tell that the commit is signed with _some_ key.

In addition to the allowed signers error, git will insert a `No signature` error:
```shell
error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification
commit 6784531c0a6a8e7c6b49161115ed19c6504af95f (HEAD -> main)
No signature
Author: Foo Bar <foo@bar.net>
Date:   Fri Aug 7 19:21:22 2026 -0700

    Signed with SSH key: no signoff trailer
```

#### Signed Commits With an Unknown SSH Key

The errors in the above case actually indicate more than that the commit is signed with _some_ key.
Those errors - or the lack of a much louder error -
show that it is signed with a key registered to the local SSH agent.

A commit signed with a key _not_ registered with the local agent
will show a longer error, prefixed with `gpg`:

```shell
commit 5a13dda67abeaa0b036b874fd53c6e8117cf8c39 (origin/main, origin/HEAD)
gpg: Signature made Tue 14 Jul 2026 11:24:18 AM PDT
gpg:                using RSA key B5690EEEBB952194
gpg: Can't check signature: No public key
Author: Foo Bar <foo@bar.net>
Date:   Tue Jul 14 11:24:18 2026 -0700

    Initial commit
```

This is the standard "Initial commit" created when initializing a repo in the GitHub UI.