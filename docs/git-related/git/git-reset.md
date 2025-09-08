git-reset - Resetting the State of a Branch
===========================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Using "git-reset-soft" to keep changed files in the index](#using-git-reset-soft-to-keep-changed-files-in-the-index)
- [Using "git-reset-hard" to drop changes from the index](#using-git-reset-hard-to-drop-changes-from-the-index)
- [Using "git-reset" and "git-clean" to roll back commits](#using-git-reset-and-git-clean-to-roll-back-commits)
- [References](#references)

Introduction
------------

As mentioned in the [section on "git restore"](git-restore.md#using-git-restore-staged-to-unstage-a-tracked-file), `git reset` and `git restore` share some similarities. The best way to explain `git reset` is that it is effectively a branch-wide `git restore`.

Using "git-reset-soft" to keep changed files in the index
---------------------------------------------------------

`git reset --soft` operates similarly to `git restore --staged`, in that it will reset the state of the branch to a previous state, while keeping the changed files intact. In other words, a "soft reset" will reset the history to the point specified, but the index will contain all the changes from prior to the soft reset.

To perform a soft reset, run the following command:

```bash
git reset --soft <COMMIT_HASH>
```

As with other instances of `<COMMIT_HASH>`, the user can pass a commitish instead, such as `HEAD~1` (one commit back from `HEAD`).

In the case of `git reset --soft HEAD~1`, Git will reset the working tree to the state of the previous commit. Running `git status` will show that the changes from before the reversion still exist, and are currently staged for commit.

As mentioned in the [section on git commit](git-commit.md#amending-commits-with-git-commit-amend), this is another way to perform a `git commit --amend` in order to amend the contents of a commit. In the case of a `git commit --amend`, the changes to be melded into the previous needed to be staged before running `git commit --amend`. In the case of `git reset --soft HEAD~1`, the user is reset to the previous commit just prior to `git commit`, and they can now add in those changes and stage them before performing another `git commit`.

As with other commands such as `git commit --amend` and `git rebase -i`, `git reset` will change the commit history.

Using "git-reset-hard" to drop changes from the index
-----------------------------------------------------

**WARNING**: use `git reset --hard` with caution -- it's a command known for deleting users' work.

In contrast to `git reset --soft`, `git reset --hard` will reset the state of the branch to a previous state, while *also removing* the changes from the index and working tree. This means that, as opposed to a soft reset, a "hard reset" will delete those files. Note that, using tools such as `git reflog`, a hard reset can be reversed.

Another item worth noting is that `git reset --hard` only affects tracked files. If a file is untracked by Git, it will be untouched by `git reset --hard`. Therefore, an untracked file will not be deleted by a hard reset; but, an untracked file that is added to the index with `git add`, *but isn't yet committed*, **will be deleted**.

This means that there can be situations where using a hard reset can delete work that is staged but not yet committed. In cases such as these, it's much more difficult to recover the lost files; though, it is *not* impossible.
A clever perusing of the `.git/objects` directory may be able to reveal the file in question, based on the timestamp of when it was created. However, as opposed to other use cases for `git cat-file -p`, it's more labor-intensive to find the right file. If the file was added to the index some time in the past, and attempting to run `git cat-file -p` on the most recently created files doesn't yield results, it will be a difficult task to find the missing file.

Using "git reset" and "git clean" to roll back commits
------------------------------------------------------

While it is far more advisable to use [interactive rebases](interactive-rebase.md) to roll back commits, it is possible to do so with `git reset`.

As explained in the section about [using "git-reset-soft" to keep changed files in the index](#using-git-reset-soft-to-keep-changed-files-in-the-index), `git reset --soft` can achieve this outcome. It is probably advisable to only use `git reset --soft HEAD~1` for this purpose, so a commit can be adjusted and committed again. But, `git reset --soft` can be used to roll back any number of commits.

`git reset --hard` can also be used to roll back a commit. But, as was explained in [using "git-reset-hard" to drop changes from the index](#using-git-reset-hard-to-drop-changes-from-the-index), it is far more destructive. As mentioned in that section, `git reset --hard` may need to be used twice, in the event that there are untracked files in the repo. Alternatively, a user can run `git clean -df` to remove any untracked files and clean up the repo so that it's in the exact same state as the working tree of the commit.

References
----------

- [thePrimeagen - Everything You'll Need to Know About Git - Reset](https://theprimeagen.github.io/fem-git/lessons/git-gud/reset)
    - A quick summary of how to use `git reset`
- [StackOverflow - How do I revert a Git repository to a previous commit?](https://stackoverflow.com/questions/4114095/how-do-i-revert-a-git-repository-to-a-previous-commit)
    - Reference for using `git reset` and `git clean` to roll back a commit
