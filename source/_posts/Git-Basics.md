---
title: Git Basics
abbrlink: 22d36175
date: 2020-02-02 18:40:30
categories:
    - Git
---

# What is Git

The major difference between Git and other VCS (Version Control System) is the way Git thinks about its data. Conceptually, most other systems store information as a list of file-based changes. These other systems think of the information they store as a set of files and the changes made to each file over time. Git dosen't think of or store its data this way, instead, Git thinks of its data more like a series of snapshots of a miniature filesystem. With Git, every time you commit, or save the state of your project, Git basically takes a picture of what all your files look like at that moment and stores a reference to that snapshot. To be efficient, if files have not changed, Git dosen't store the file again, just a link to the previous identical file it has already stored. Git thinks about its data more like a stream of snapshots. Because you have the entire history of the project tight there on your local disk, most operations seem almost instantaneous, and most operations in Git need only local files and resources to operate -- generally no information is needed from another computer on your network. 

{% note info %}
Git has three main states that your files can reside in:
- **modified**: you have changed the file but have not commited it to your database yet.
- **staged**: you have marked a modified file in its current version to go into your next commit snapshot.
- **committed**: the data is safely stored in your local database.

This leads us to the three main sections of a Git project:
- **the working directory**: a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.
- **the staging area**: a file, generally contained in your Git directory, that stores information about what will go into your next commit. Its technical name in Git parlance is the "index", but the phrase "staging area" works just so well.
- **the Git directory**: stores the metadata and project database for your project. This is the most important part of Git, and it is what is copied when you `clone` a repository from another computer.
{% endnote %}

![main sections of Git](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Tutorial%20of%20Git/main%20sections%20of%20Git.png)

Git comes with a tool called `git config` that lets you get and set configuration variables that control all aspects of how Git looks and operates. These variables can be stored in three different places:

{% note info %}
- `/etc/gitconfig` file: Contains value applied to every user on the system and all their repositories. If you pass the option `--system` to `git config`, it reads and writes from this file specifically.
- `~/.gitconfig` or `~/.config/git/config` file: Values specific personally to you, the user. You can make Git read and write to this file specifically by passing the `--global` option, and this affects all of the repositories you work with on your system.
- `config` file in the Git directory of whatever repository you're currently using: Specific to that single repository. You can force Git to read from and write to this file with the `--local` option, but that is in fact the default
{% endnote %}

# Getting a Git Repository

You can take a local directory that is currently not under version control, and turn it into a Git repository.

```zsh
$ cd /Users/user/my_project
$ git init
```

Or, you can `clone` an existing Git repository from elsewhere.

```zsh
$ git clone <url> [target-directory]
```

# Recording Changes to the Repository

{% note info %}
Each file in your working directory can be in one of two states:
- **tracked**: Tracked files are files that were in the last snapshot or in the Staging Area. In short, tracked files are files that Git knows about.
- **untracked**: Any files in your working directory that were not in your last snapshot and are not in your staging area. When you first clone a repository, all of your files will be tracked and unmodified because Git just checked them out and you haven't edited anything.
{% endnote %}

![lifecycle of status of files](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Tutorial%20of%20Git/lifecycle%20of%20the%20status%20of%20files.png)

```zsh
# determine which files are in which state
$ git status

# move a file from the working directory to the staging area
# "git add ." means move all files in current directory
$ git add <file>

# discard changes in working directory
# you can only use this command on tracked files
$ git checkout -- <file>

# move a file from the staging area to the working directory
$ git reset HEAD <file>

# see what you've changed but not yet staged
$ git diff
# see what you've staged that will go into your next commit
$ git diff --staged

# remove a file from Git, and also removes the file from the working directory (which means delete the file)
$ git rm <file>
# keep the file in the working directory but remove it from the staging area
# this is particulary useful if you forgot to add something to your .gitignore file and accidentally staged it
$ git rm --cached <file>

# commit your changes that are in the staging area
$ git commit [-m <msg>]
```

Often, you'll have a class of files that you don't want Git to automatically add or even show you as being untracked. In such cases, you can create a file listing patterns to match them named `.gitignore`

{% note info %}
The rules for the patterns you can put in the `gitignore` file are as follows:
- Blank lines or lines starting with `#` are ignored.
- Standard glob patterns work, and will be applied recursively throughout the entire working directory.
- You can start patterns with a forward slash (`/`) to avoid recursivity.
- You can end patterns with a forward slash (`/`) to specify a directory.
- you can negate a pattern by starting it with an exclamation point (`!`).
{% endnote %}

Here is an example `.gitignore` file:

```zsh
# ignore all .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in any directory named build
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory and any of its subdirectories
doc/**/*.pdf
```

# Working with Remotes

Remote repositories are versions of your project that are hosted on the Internet or network somewhere.

```zsh
# show the list of the shortnames of each remote handle you've specified
$ git remote

# show the URLs that Git has stored for the shortname
$ git remote -v

# add a new remote Git repository
$ git remote add <name> <url>

# get data from remote repository
# this command only downloads the data to your local repository
# it doesn't automatically merge it with any of your work of modify what you're currently working on
$ git fetch <remote>

# push any commits you've done back up to the server
$ git push <remote> <branch>

# inspect a remote
$ git remote show <name>

# change a remote's shortname
$ git remote rename <old> <new>

# remove a remote
$ git remote remove <name>
```

# Tagging

```zsh
# list the existing tags
$ git tag

# create an tag
$ git tag <tagname> [<commit>] [-a] [-m <msg>]

# the `git push` command doesn't transfer tags to remote servers
# you will have to explicitly push tags to a shared server after you have created them
$ git push <remote> <tagname>

# transfer all of your tags to the remote server that are not already there
$ git push <remote> --tags

# delete local tag
$ git tag -d <tagname>

# delete remote tag
$ git push <remote> :refs/tags/<tagname>
# or
$ git push <remote> --delete <tagname>

# checkout tag
$ git checkout <tagname>
```