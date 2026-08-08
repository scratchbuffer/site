+++
type = "article"
title = "Sign Git Commits With SSH Keys"
description = "Git & GitHub Setup for Signed Commits & Verification"

slug = "git-commit-sign-ssh"

tags = ['Git', 'GitHub']

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
so the other backend options were shoved into the existing config namespace.

### 1.3 Set the Default Signing Key

Point Git at the local SSH private key:
```shell
git config --global user.signingkey ~/.ssh/id_ed25519
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
an artifact of when Git was primarily used without centralized hosted "forges" like GitHub or GitLab.

For now we can ignore this.
The error messages are enough to tell that the commit is signed with _some_ key.

In addition to the allowed signers error, Git will insert a `No signature` error:
```shell
error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification
commit 6784531c0a6a8e7c6b49161115ed19c6504af95f (HEAD -> main)
No signature
Author: Foo Bar <foo@bar.net>
Date:   Fri Aug 7 19:21:22 2026 -0700

    Signed with SSH key: no signoff trailer
```

#### Signed Commits With an Unknown GPG Key

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
5. Under **Branch targeting criteria**, click the **Add target** and select a branch target
6. Under the **Branch rules**, select the **Require signed commits** checkbox
7. Scroll down to the bottom and click **Create**, or **Save changes** if editing an existing ruleset

### 3.2 Verify GitHub Rejects a Signed Commit With an Unregistered Key

At this point GitHub should reject the commit,
assuming SSH key has not been registered as a _signing_ key.
Registering it as an authentication key is a step almost everyone completes for new SSH key,
but authentication keys and signing keys are distinct concepts.

We can verify this and see what the error looks like.
To simplify, we just want to push a single commit signed with the SSH key.
We can delete the previous empty example commits and recreate the single signed one:

```shell
git reset --hard HEAD~3
git commit --allow-empty -m "Signed with SSH key: no signoff trailer"
```

Make sure the new commit is on the same branch which is protected by the ruleset on GitHub.

Now try to push:
```shell
% git push
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 424 bytes | 424.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: error
: GH013: Repository rule violations found for refs/heads/main.
remote: Review all repository rules at https://github.com/scratchbuffer/git-test/rules?ref=refs%2Fheads%2Fmain
remote:
remote: - Commits must have verified signatures.
remote:   Found 1 violation:
remote:
remote:   78d74bf3876e0c8f749a0be785a8df0aab486118
remote:
To github.com:scratchbuffer/git-test.git
 ! [remote rejected]
 main -> main (push declined due to repository rule violations)

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

## Q.E.D

We are good to push!
This SSH-based setup keeps it nice and simple for anyone who does not want to bother with GPG keys.

If you are interested in setting up the Allowed Signers file,
it is documented in the `ssh-keygen(1)` man page,
but the [GitLab doc on signing commits](https://docs.gitlab.com/user/project/repository/signed_commits/ssh/#verify-commits-locally)
has a more friendly explanation.
