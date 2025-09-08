git-rm - Removing Files from the Working Tree and Index
=======================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Using "git-rm" to completely remove a file](#using-git-rm-to-completely-remove-a-file)
- [Using "git-rm-cached" to remove a file only from the working tree and index](#using-git-rm-cached-to-remove-a-file-only-from-the-working-tree-and-index)

Introduction
------------

`git rm` is a command that allows the user to remove a file. Depending on the usage, `git rm` will operate in different ways.

As with `git restore`, `git rm` is a command that should be used with caution, as a misuse of the command may result in an untracked file being deleted.

Using "git-rm" to completely remove a file
------------------------------------------

To remove a path that is currently being tracked by Git **AND** remove the path from the Git repo entirely (aka, delete the path), run the following command:

```bash
git rm <PATH>
```

As with `git restore`, `git rm` can be used on any path that is passed as an argument, whether it be a path to a file or a directory.

`git rm` will only operate on files that do not currently have "local modifications", regardless of whether the changes are staged or unstaged. To remove a file with changes, there are a few options:

1. Run `git restore <PATH>` to restore the file to the state in the working tree, then run `git rm <PATH>`.
1. Run `git rm --cached <PATH>` to keep the file in the directory, but to direct Git to stop tracking the file.
1. Run `git rm -f <PATH>` to force the removal of the file from the working tree.

Using "git-rm-cached" to remove a file only from the working tree and index
---------------------------------------------------------------------------

While `git rm` removes files, both from the working tree (and index) **AND** the actual file, `git rm --cached` will remove the file *only* from the working tree and index.

To remove a path **only** from the working tree and index, run the following command:

```bash
git rm --cached <PATH>
```

When a new file is created and added to the index using `git add <PATH>`, but then needs to be removed from the index, `git rm --cached` is the best method to remove the file from the index while leaving the file intact.

`git rm --cached` can be particularly useful in a situation where a file is added and committed, but shouldn't have been. If the user wants to preserve the file and its contents, while also removing the file from the commit, then an interactive rebase (`git rebase -i`) in combination with `git rm --cached <PATH>` is the best way to achieve those results. For more information on interactive rebases, refer to the [section on "git rebase -i"](interactive-rebase.md#editing-the-contents-of-a-commit).
