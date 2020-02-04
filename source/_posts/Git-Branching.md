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
$ git branch <branchname>

# switch branch
$ git checkout <branchname>

# create a branch and switch to it
$ git checkout -b <branchname>

# merge the target branch into your current branch
$ git merge <targetbranchname>

# delete a branch
$ git branch -d <branchname>

# no options => local branches list
# -r => remote branches list
# -a => both remote and local branches list
$ git branch [-r | -a]

# see the last commit on each branch
$ git branch -v

# filter the list to branches that you have or haven't merged into your current branch
$ git branch [--merged | --no-merged]
```

# Branches in a Nutshell

To really understand the way Git does branching, we need to take a back and examine how Git stores its data. Git doesn't store data as a series of changesets or differents, but instead as a series of **snapshots**. When you make a commit, Git stores a commit object that contains a pointer to the snapshot of the content you staged. This object also contains the author's name and email address, the message that you typed, and the pointer to the commit or commits that directly came before this commit (its parent or parents): zero parents for the inital commit, one parent for a normal commit, and multiple parents for a commit that results from a merge of two of more branches.

{% note info %}
**A branch in Git is simply a lightweight movable pointer to one of these commits.**
{% endnote %}

When you create a new branch, Git creates a new pointer for you to move around. And How does Git know what branch you're currently on? It keeps a special pointer called `HEAD`. In Git, this is a pointer to the local branch you're currently on.

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




