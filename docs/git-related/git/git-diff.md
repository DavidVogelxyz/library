git-diff - Viewing Differences in the Command Line
==================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Using "git-diff" to view changes](#using-git-diff-to-view-changes)
- [References](#references)

Introduction
------------

As reviewed in the [section on "git log"](git-log.md#reviewing-changes-with-git-log-and-git-show), both `git log -p` and `git show` can be used to view the changes contained in commits. But, a user may want to view the changes between the current working tree and the current state of their repo. Sometimes, the user wants to see differences between staged files (the index) and the working tree; other times, the user wants to view changes of unstaged files against the working tree. In these cases, the user can run `git diff`.

Note that `git diff`, while widely used, is often considered a sub-par way to view differences. An alternate method, using `git difftool`, is outlined in the [section on "git difftool"](git-difftool.md).

Using "git-diff" to view changes
--------------------------------

First, note that `git diff` will only show differences for unstaged files for which a version exists in the working tree. If the file is not part of the working tree, then the whole file is different, but `git diff` will not show the difference.

To compare differences for **ALL FILES that exist in the working tree** (both staged and unstaged) against the most recent commit, run the following command:

```bash
git diff HEAD
```

To compare differences for **ONLY THE UNSTAGED FILES that exist in the working tree** against the most recent commit, run the following command:

```bash
git diff
```

To compare differences for **ONLY THE STAGED FILES** (even if no version exists in the working tree) against the most recent commit, run the following command:

```bash
git diff --staged
```

Another way to compare differences for **ONLY THE STAGED FILES** against the most recent commit, run the following command:

```bash
git diff --cached
```

To compare differences for **a specific file that exists in the working tree** against the most recent commit, run the following command:

```bash
git diff HEAD <PATH>
```

To compare differences **between two branches**, run the following command. Note that the branches can be switched; however, the changes will be read in reverse. Put another way, any additions made from an old branch to a new one will look like deletions.

```bash
git diff <BRANCH_OLD> <BRANCH_NEW>
```

References
----------

- [Atlassian - Git diff](https://www.atlassian.com/git/tutorials/saving-changes/git-diff)
    - Reference for `git diff`
