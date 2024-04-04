---
title: Git merge to master
date: 2024-04-03 11:31:00 -0700
categories: [Git]
tags: 
Pin:
---

## Git merge to master

### Merge a branch to master

```bash
# Switch to a new branch
git checkout -b new_branch
# Confirm the git status
git status
# Fetch the latest changes from the upstream repository
git fetch upstream master
# Merge the changes from the upstream repository
git merge upstream/master
# Push the new branch to the remote repository
git push -u origin new_branch
# Switch to the master branch
git checkout master
# Merge the new branch to the master branch
git merge new_branch
# Push the changes to the remote repository
git push origin master
```