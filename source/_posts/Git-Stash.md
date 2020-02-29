---
title: Git Stash
abbrlink: 1955d066
date: 2020-02-23 17:06:56
categories:
    - Git
---

```zsh
# save your local modifications to a new stash entry
# and roll them back to HEAD
$ git stash push [-m <message>]

# list the stash entries that you currently have
# stash@{0} is the latest entry, stash@{1} is the one before
$ git stash list

# show the changes recorded in the stash entry as a diff 
# between the stashed contents and 
# the commit back when the stash entry was first created
$ git stash show [<stash>]

# remove a single stashed state from the stash list
# and apply it on top of the current working tree state
$ git stash pop [<stash>]

# like pop, but do not remove the state from the stash list
$ git stash apply [<stash>]

# remove all the stash entries
$ git stash clear

# remove a single stash entry from the list of entries
$ git stash drop [<stash>]
```

When you want to record the current state of the working directory and the index, but want to go back to a clean working directory. The command `git stash` saves your local modifications away and reverts the working directory to match the `HEAD` commit.

The modifications stashed away by this command can be listed with `git stash list`, inspected with `git stash show`, and restored with `git stash apply`. Calling `git stash` without any arguments is equivalent to `git stash push`. A stash is by default listed as "WIP on **branchname**...", but you can give a more descriptive message on the command line when you create one.

The latest stash you created is stored in `refs/stash`; older stashes are found in the reflog of this reference and can be named using the usual reflog syntax (e.g. `stash@{0}` is the most recently created stash, `stash@{1}` is the one before it, `stash@{2.hours.ago}` is also possible). Stashes may also be referenced by specifying just the stash index (e.g. the integer `n` is equivalent to `stash@{n}`).
