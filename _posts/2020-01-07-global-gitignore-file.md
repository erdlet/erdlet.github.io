---
layout: post
title:  "Setting a global .gitignore file"
date:   2020-01-07 09:00:00 +0200
categories: git
author: Tobias Erdle
---

Everyone working on multiple Git projects might have thought the same: *'Why do I need
to add the same [IDE and build tool] files to .gitignore for each single project?*
While reading the docs I found a very nice solution for this: `git config core.excludesfile`

Using this setting, someone creates a file named `.gitignore_global` and add its
common ignored files to it. Then you need to point the config to this file and
use additionally the `--global` flag to enable the new ignore file for every git project in your user space.

The listing shows all steps to be performed:

```bash
# 1. Create global .gitignore file
touch ~/.gitignore_global

# 2. Edit the file (in this case with vim)
vim ~/.gitignore_global

# 3. Set as global exclude file
git config --global core.excludesfile '~/.gitignore_global'

```
