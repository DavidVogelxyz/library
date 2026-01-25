git-cherry-pick - Applying Commits from One Branch to Another
=============================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Using "git-cherry-pick" to apply a commit from one branch to another branch](#using-git-cherry-pick-to-apply-a-commit-from-one-branch-to-another-branch)
- [Using "git-cherry-pick" to apply a range of commits](#using-git-cherry-pick-to-apply-a-range-of-commits)
- [Troubleshooting conflicts with "git-cherry-pick"](#troubleshooting-conflicts-with-git-cherry-pick)
- [References](#references)

Introduction
------------

`git cherry-pick` allows a user to take a commit from one branch and commit it to another branch. The main difference between `git cherry-pick` and `git merge` is that `git cherry-pick` does not require a "common ancestor" in order to apply a commit.

To imagine an example where `git cherry-pick` could be useful, consider the following example:

"A user has a production branch (`prod`) and a development branch (`dev`), which share the same `HEAD`. A user then creates three new commits to `dev`: `A`, `B`, and `C`. `A` and `B` contains changes to code, but `C` only contains some changes to the `README` file. The user would like to get the changes to the `README` over to `prod`, without merging commits `A` and `B`."

In this case, the user can use `git cherry-pick` to apply `C` to the `HEAD` of `prod`, resulting in one new commit being added to `prod`. This commit will contain *only* the changes to the `README` file found in commit `C`, without including any code changes from commits `A` and `B`.

Using "git-cherry-pick" to apply a commit from one branch to another branch
---------------------------------------------------------------------------

To apply a commit from one branch to a different branch, simply check out the "receiving" branch and run the following command:

```bash
git cherry-pick <HASH_OF_COMMIT>
```

This will take the commit referenced by `<HASH_OF_COMMIT>` and apply it to the current branch. The commit will receive a new hash, but the commit's message and its changes will stay the same.

As described in the section on ["git-reflog"](git-reflog.md), `git reflog` can be utilized with both `git merge` and `git checkout` to recover lost history. In some situations, it will make more sense to run `git merge` or `git checkout` in order to reorient the tip of a branch. In other cases, `git cherry-pick` will give the user more flexibility by allowing them to choose which commits are applied, and in what order.

Using "git-cherry-pick" to apply a range of commits
---------------------------------------------------

To apply a range of commits (from `HASH_OF_COMMIT_A` to `HASH_OF_COMMIT_B`, ***including*** `A`), run the following command:

```bash
git cherry-pick <HASH_OF_COMMIT_A>^..<HASH_OF_COMMIT_B>
```

Note the syntax joining the range is `^..` when including `HASH_OF_COMMIT_A`.

To apply a range of commits ***excluding*** `HASH_OF_COMMIT_A`, run the following command:

```bash
git cherry-pick <HASH_OF_COMMIT_A>..<HASH_OF_COMMIT_B>
```

Note that the syntax joining the range is `..` when excluding `HASH_OF_COMMIT_A`.

As with other Git commands, `HASH_OF_COMMIT` can be a branch name -- it ***does not*** have to be the hash itself.

Troubleshooting conflicts with "git-cherry-pick"
------------------------------------------------

Occasionally, a `git cherry-pick` will result in a conflict. For more information on how to resolve a conflict, refer to the section on [resolving merge conflicts](resolving-merge-conflicts.md). More specifically, review the section on [resolving a conflict when rebasing](resolving-merge-conflicts.md#resolving-a-conflict-when-rebasing).

The section on "resolving conflicts when rebasing" is suggested because `git cherry-pick` and `git rebase` share many similarities -- both commands are essentially checking out a previous commit and then playing back a new commit on top. Because of this, the commands to handle a `git cherry-pick` conflict mimic the `git rebase` commands.

To abort a `git cherry-pick`, run the following command:

```bash
git cherry-pick --abort
```

When conflict resolution is complete, the user proceeds with the following command:

```bash
git cherry-pick --continue
```

References
----------

- [thePrimeagen - Everything You'll Need to Know About Git - HEAD](https://theprimeagen.github.io/fem-git/lessons/branches-merges-and-more/head)
    - Information on how `git cherry-pick` works
- [StackOverflow - How to cherry-pick multiple commits](https://stackoverflow.com/questions/1670970/how-to-cherry-pick-multiple-commits/3933416#3933416)
    - Information on running `git cherry-pick` to apply a range of commits
