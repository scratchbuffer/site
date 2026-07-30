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

Ensure git user name and email are set
