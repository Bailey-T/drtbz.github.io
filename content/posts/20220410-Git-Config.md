---
draft: True
title: Git Config
description: Super useful git configs

date: 2022-04-10T20:00:00+00:00
tags: 
    - Git
    - Reference
categories:
    - Git
    - Reference
---
Been a while. Got a new laptop and realised it would be dead handy to have this written down somewhere. This will be a living page I update if anything else comes along.

## Useful Git Configs
```
# Standard stuff - set username and email for commits
git config --global user.name TomB
git config --global user.email 123456+user@users.noreply.github.com

# Setup GPG for verified commits and automatically assume -S on all commits
git config --global user.signingkey A1B2C3D4E5
git config --global commit.gpgSign true

# Don't need to set set-upstream to push branches!
git config --global push.default current
```
## References
[gpgKeys@GithubDocs](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)