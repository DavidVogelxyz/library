# git-restore - Restoring Files to the State of the Working Tree

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Using "git-restore" to restore the state of the working tree](#Using-git-restore-to-restore-the-state-of-the-working-tree)
    - [Patch restoring](#Patch-restoring)
- [Using "git-restore-staged" to unstage a tracked file](#Using-git-restore-staged-to-unstage-a-tracked-file)
- [References](#References)

## Introduction

`git restore` is command that allows a user to restore files to the state of the working tree. This command can sometimes be tricky to use -- if run incorrectly, a user may get rid of a file that they only meant to unstage.

## Using "git-restore" to restore the state of the working tree

To restore a file to the state of the working tree, run the following command:

```
git restore <PATH>
```

`git restore` works for any path specified, whether it's a path to a file, a directory, or the entire Git repo.

### Patch restoring

Just like with `git add`, `git restore` can also be used with the `-p` switch, representing "patch". If `git restore -p` is run on a path, then Git will open an interactive session where the user can choose hunks to restore to the state of the working tree.

When using `git restore -p`, a yes (`y`) will restore a hunk to the state found in the working tree, and a no (`n`) will leave the changes as they are.

## Using "git-restore-staged" to unstage a tracked file

Running `git restore` with the `--staged` switch allows a user to unstage a file, while keeping the changes. In this way, `git restore --staged` operates very similarly to `git reset --soft` -- in both cases, Git will remove the changes from the index without restoring the files to the state found in the working tree. In contrast, `git restore` operates similar to `git reset --hard`, where Git will restore the file and remove the changes. For more information on `git reset`, refer to the [section on "git-reset"](git-reset.md#Introduction).

To unstage a file, run the following command on a staged file:

```
git restore --staged <PATH>
```

This is a situation where `git restore` can be tricky. If `git restore` is run on a staged file without the `--staged` switch, then Git will restore the file to the state found in the working tree.

Note that `git restore --staged` only works on files that already exist in the working tree. If a file is new, and hasn't yet been added to the working tree, it should be removed by running the following command:

```
git rm --cached <PATH>
```

For more information on `git rm`, refer to the [section on "git-rm"](git-rm.md#Using-git-rm-cached-to-remove-a-file-only-from-the-working-tree-and-index).

## References

- [Git SCM documentation - git restore](https://git-scm.com/docs/git-restore)
    - Reference for `git restore`
- [StackOverflow - What is `git restore` and how is it different from `git reset`?](https://stackoverflow.com/questions/58003030/what-is-git-restore-and-how-is-it-different-from-git-reset)
    - Another reference for `git restore`
