+++
type = "article"
title = "Sign Git Commits With SSH Keys"
description = "Git & GitHub Setup for Signed Commits & Verification"

slug = "git-commit-sign-ssh"

tags = ['Git', 'GitHub', 'GitLab']

date = 2026-07-14
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

Note that this does *not* add the standard signoff trailer to the commit message,
which looks like `Signed-off-by: Foo Bar <foo@bar.net>`.

The `commit.gpgsign` config or `-S/--signoff` flag adds the cryptographic signature
which is not visible in the commit message.