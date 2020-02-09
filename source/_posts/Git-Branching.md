---
title: Git Branching
abbrlink: 3557a152
date: 2020-02-03 17:27:19
categories:
    - Git
---

# Commands About Branch

```zsh
# create a branch
# only create, don't switch
# if you don't type existbranch, use current branch
$ git branch <newbranch> [existbranch]

# switch your working directory
$ git checkout [branchname | tagname | commitsha1]

# create a branch and switch to it
# if you don't type existbranch, use current branch
$ git checkout -b <newbranch> [existbranch]

# move the branch pointer
$ git branch -f [branchname | tagname | commitsha1]

# merge the target branch into your current branch
$ git merge <targetbranch>

# no options => local branches list
# -r => remote branches list
# -a => both remote and local branches list
$ git branch [-r | -a]

# see the last commit on each branch
$ git branch -v

# filter the list to branches that you have or haven't merged into your current branch
$ git branch [--merged | --no-merged]

# fetch any data from the given remote that you don't yet have
# update your local database
# move your remote/branch pointer to its new, more up-to-data position.
# if you don't type remote, it will use origin as default
$ git fetch [remotename]

# push the local branch to the remote branch
$ git push <remotename> <localbranch:remotebranch>

# fetch the upstream branch and merge into the tracking branch
$ git pull

# set the current branch to track remotebranch
# you can use this command to change the relationship between a tracking branch and a upstream branch
$ git branch <-u | --set-upstream-to> <[[remotes/]origin/]remotebranch>

# if current branch is not a tracking branch, and you want to push it
# if remotebranch isn't exist, Git will automatically create it
$ git push --set-upstream <remotename> <remotebranch>

# see what tracking branches you have set up
$ git branch -vv

# delete a local branch
$ git branch -d <branchname>

# delete a remote branch
$ git push <remotename> --delete <branchname>

# take the patch(s) and replay it/them on top of somewhere else
$ git rebase <branchname | tagname | commitsha1>

# if there is any conflict while rebasing, use this command
$ git rebase <--continue | --abort | --skip>
```

# Branches in a Nutshell

To really understand the way Git does branching, we need to take a back and examine how Git stores its data. Git doesn't store data as a series of changesets or differents, but instead as a series of **snapshots**. When you make a commit, Git stores a commit object that contains a pointer to the snapshot of the content you staged. This object also contains the author's name and email address, the message that you typed, and the pointer to the commit or commits that directly came before this commit (its parent or parents): zero parents for the inital commit, one parent for a normal commit, and multiple parents for a commit that results from a merge of two of more branches.

{% note info %}
**A branch in Git is simply a lightweight movable pointer to one of these commits.**
{% endnote %}

When you create a new branch, Git creates a new pointer for you to move around. And How does Git know what branch you're currently on? Git keeps a special pointer called `HEAD`, it is the symbolic name for the currently checkout out commit -- it's essentially what commit you're working on top of. Normally `HEAD` attaches to a branch, when you commit, the status of that branch is altered. When `HEAD` attaches to a commit instead of a branch, this phenomenon is called **detach**.

# Basic Branching and Merging

![before-merge](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/before-merge.png)

```zsh
$ git checkout -b bugFix
$ git commit
$ git checkout master
$ git commit
$ git merge bugFix
```

![after-merge](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/after-merge.png)

# Remote Branches

Look at an example, let's say you have a Git server on your network at `git.ourcompany.com`. If you clone from this, Git's `clone` command automatically names it `origin` for you, pulls down all its data, creates some pointers to correspond to remote branches, the name of remote branchs in local will has a prefix like `remotes/` or `remotes/origin/`. Git also gives you your own local `master` branch starting at the same places as origin's `master` branch, so you have something to work from.

![Server and local repositories after cloning](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/Server%20and%20local%20repositories%20after%20cloning.png)

If you do some work on your local `master` branch, and, in the meantime, someone else pushes to `git.ourcompany.com` and updates its `master` branch, then your histories move forward differently. Also, as long as you stay out of contact with your `origin` server, your `origin/master` pointer doesn't move.

![Local and remote work can diverge](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/Local%20and%20remote%20work%20can%20diverge.png)

To synchronize your work with a given remote, you run a `git fetch <remote>` command. This command fetches any data from the given remote that you don't yet have, and updates your local database, moving your `origin/master` pointer to its new, more up-to-data position.

{% note warning %}
**It's important to note that when you do a fetch that brings down new remote-tracking branches, you don't automatically have local, editable copies of them. In other words, in this case, you don't have a new branch -- you have only an `origin/master` pointer that you can't modify.**
{% endnote %}

![git fetch updates your remote-tracking branches](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/git%20fetch%20updates%20your%20remote-tracking%20branches.png)

When you want to share a branch with the world, you need to push it up to a remote to which you have write access. Your local branches aren't automatically synchronized to the remotes you write to -- you have to explicitly push the branches you want to share. That way, you can use private branches for work you don't want to share, and push up only the topic branches you want to collaborate on.

If you have a branch named `serverfix` that you want to work on with others, you can push it up the same way you pushed your first branch. Run `git push <remote> <branch>`:

```zsh
$ git push origin serverfix
Counting objects: 24, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (24/24), 1.91 KiB | 0 bytes/s, done.
Total 24 (delta 2), reused 0 (delta 0)
To https://github.com/schacon/simplegit
 * [new branch]      serverfix -> serverfix
```

This is a bit of a shortcut. Git automatically expands the `serverfix` branchname out to `refs/heads/serverfix:refs/heads/serverfix`, which means, "Take my `serverfix` local branch and push it to update the remote's `serverfix` branch." You can also do `git push origin serverfix:serverfix`, which does the same thing -- it says, "Take my serverfix and make it the remote's serverfix." You can use this format to push a local branch into a remote branch that is named differently. If you didn't want it to be called `serverfix` on the remote, you could instead run `git push origin serverfix:awesomebranch` to push your local `serverfix` branch to the `awesomebranch` branch on the remote project.

Checking out a local branch from a remote-tracking branch automatically creates what is called a "tracking branch", and the branch it tracks is called an "upstream branch". Tracking branches are local branches that have a direct relationship to a remote branch. If you're on a tracking branch and type `git pull`, Git automatically know which server to fetch from and which branch to merge in. If you want to change the upstream branch you're tracking, you can use the `-u` or `--set-upstream-to` option to `git branch` to explicitly set it at any time.

# Rebasing

{% note warning %}
**It turns out that in addition to the commit SHA-1 checksum, Git also calculates a checksum that is based just on the patch introduced with the commit, this is called a "patch-id".**
{% endnote %}

In addition to `merge`, if you want to integrate changes from `experiment` into `master`, there is another way.

![Simple divergent history](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/Simple%20divergent%20history.png)

You can take the patch of the change that was introduce in `C4` and reapply it on top of `C3`. In Git, this is called **rebasing**. With the `rebase` command, you can take all the changes that were committed on one branch and replay them on a different branch.

```zsh
$ git checkout experiment
$ git rebase master
```

![Rebasing the change introduced in C4 onto C3](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/Rebasing%20the%20change%20introduced%20in%20C4%20onto%20C3.png)

At this point, you can go back to the `master` branch and do a fast-forward merge.

```zsh
$ git checkout master
$ git merge experiment
```

![Fast-forwarding the master branch](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Git%20Branching/Fast-forwarding%20the%20master%20branch.png)

{% note info %}
**Rebasing replays changes from one line of work onto another in the order they were introduced, whereas merging takes the endpoints and merges them together. There is no difference in the end product of the integration, but rebasing makes for a cleaner history. In addition, when you rebase stuff, you're abanding existing commits and creating new ones that are similar but different.**
{% endnote %}

{% note warning %}
**If there is any conflict while you are rebasing, you should fix the confilct, and then use `git add` to stage the files, use `git rebase --continue` to finish the rebasing process. In this process, do not use `git commit`, it will create a commit and detach the HEAD!**
{% endnote %}