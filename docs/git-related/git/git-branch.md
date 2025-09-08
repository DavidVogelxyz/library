git-branch - Separating Development from Production
===================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Creating, and switching, to new branches](#creating-and-switching-to-new-branches)
- [Listing branches](#listing-branches)
- [What is the current branch?](#what-is-the-current-branch)
- [Where is the branch information in the Git directory?](#where-is-the-branch-information-in-the-git-directory?)
- [Deleting branches](#deleting-branches)
- [Renaming branches](#renaming-branches)
- [What to do with a branch full of new commits](#what-to-do-with-a-branch-full-of-new-commits)
- [References](#references)

Introduction
------------

Git branches, simply put, are a way to create divergence in a repo. With a better understanding of Git's internals, as described in [Git's internal file structure](git-internal-file-structure.md), it is easy to understand how creating branches is virtually free, in terms of data. Rather than creating new copies of all the files and directories, Git simply creates a new refrence ("the branch"), and uses pointers to refer back to all the files.

Creating, and switching, to new branches
----------------------------------------

To create a new branch, run the following command:

```bash
git branch <BRANCH>
```

This creates a new branch with whatever name was passed to the command. This branch points to the same commit as the branch the user is currently on. However, the user stays on the original branch after creating the new branch.

To get to the new branch, run the following command:

```bash
git checkout <BRANCH>
```

The command `git switch <BRANCH>` can also be used -- however, most users use `git checkout` in place of `git switch`. This is due to `git checkout`'s versatility -- it can be used to checkout commits and tags, in addition to branches. However, for future reference, a new branch can be commit with `git switch` by running the command `git switch -c <BRANCH>`.

Alternatively, "creating a branch and switching to it" can be performed within a single step, by using the following command:

```bash
git checkout -b <BRANCH>
```

Note the following: `git checkout -b` will attempt to create a new branch, and then switch to it. If a branch already exists with the provided name, the command will fail with the error: `fatal: a branch named '<BRANCH>' already exists`. A switch similar to `-b` exists: `-B`. In a situation where `<BRANCH>` already exists, `-B` will succeed -- Git will reset ("delete") the branch and check it out. Therefore, it's highly advisable to use only `-b`, or to use `git branch <BRANCH>` to create the new branch, and then check it out.

Listing branches
----------------

To list all the available branches, run the following command:

```bash
git branch
```

The word "available" is used, because there are situations where branches may exist elsewhere, but are not present locally. Therefore, `git branch` only lists the branches that exist locally. Put another way, `git branch` only lists the branches that have refs in the `.git` directory.

Note that `git branch` will only show the branches that are local to the repo. If a Git project has remotes configured, then `git branch` won't show any information about them. In order to see all the branches, including the ones that exist on remotes, run the following command:

```bash
git branch -a
```

For more information on remote repos, check out the section on [working with remote repos in Git](git-remote.md#using-git-fetch-to-grab-the-current-state-of-a-remote-repo).

What is the current branch?
---------------------------

There are a few different ways to see what branch the repo is currently on.

One way to do this is to run `git branch` -- a `*` will appear next to the current branch.

Another method is to run `git status` -- the first line of the output will say `on branch <BRANCH>`.

A third way is to run `git log`, or `git log --decorate`. The benefit of using `git log` is that it will tag all the branches to the commit to which they point.

Where is the branch information in the Git directory?
-----------------------------------------------------

The files for different branches are stored in `.git/refs/head`.

Deleting branches
-----------------

To delete a branch, run the following command:

```bash
git branch -D <BRANCH>
```

This will force the deletion of the branch, similar to how `git checkout -B <BRANCH>` forces the creation of a branch (even if it already exists). Using `git branch -d <BRANCH>` works as well, but only if the branch has been fully merged into the upstream branch, or it will fail with error: `error: the branch '<BRANCH>' is not fully merged`.

Renaming branches
-----------------

To rename a branch, run the following command:

```bash
git branch -m <BRANCH>
```

This command will rename the branch, and will migrate the branch to the new name, together with its config and its reflog. To learn more about reflogs, check out the section on ["git-reflog"](git-reflog.md).

Just like with other commands, `-M` will also perform the same action, but will force the name change. In some cases, `-m` will fail -- for example, if there's already a branch named `<BRANCH>`. When using `-M`, Git will reset ("delete") the branch with the name `<BRANCH>` and rename the current branch to `<BRANCH>`. Use `-M` with caution, as `-m` will warn the user that `a branch named '<BRANCH>' already exists`, while `-M` will not.

Note that `-m` and `-M` are abbreviations for "move". Just like in Linux, there is no actual way to "rename" something. Instead, Git is "moving" the files related to the current branch to new paths under the "new name".

What to do with a branch full of new commits
--------------------------------------------

Please refer to the section on ["git-merge"](git-merge.md) or ["git-rebase"](git-rebase.md) for information on how the new commits over to a different branch.

References
----------

- [thePrimeagen - Everything You'll Need to Know About Git - Branching](https://theprimeagen.github.io/fem-git/lessons/branches-merges-and-more/branching)
    - A deep dive into `git branch`
