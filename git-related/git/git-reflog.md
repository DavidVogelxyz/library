# git-reflog - Recovering Lost History

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [The basics of git-reflog](#the-basics-of-git-reflog)
- [Using "git-reflog" and "git-cat-file" to recover lost history](#using-git-reflog-and-git-cat-file-to-recover-lost-history)
- [Using "git-reflog" and "git-merge" to recover lost history](#using-git-reflog-and-git-merge-to-recover-lost-history)
- [Using "git-reflog" and "git-checkout" to recover lost history](#using-git-reflog-and-git-checkout-to-recover-lost-history)
- [Using "git-reflog" and "git-cherry-pick" to recover lost history](#using-git-reflog-and-git-cherry-pick-to-recover-lost-history)
- [References](#references)

## Introduction

As discussed in the section on [Git's internal file structure](git-internal-file-structure.md), a user with understanding of how Git works is able to navigate through Git's internal plumbing to find the files that reference commits, trees (directories), blobs (files), refs (references), and more. But, this knowledge can also allow the user to do some extraordinary feats, such as recovering information that was previously committed to a repo, but then deleted.

## The basics of git-reflog

By default, running `git reflog` on its own will produce a history, similar to a `git log --oneline`. "Reflog" is simply a "log of references", and specifically refers to the ref `HEAD`. The history returned by `git reflog` is a log of where `HEAD` has pointed. In other words, `git reflog` allows the user to see the history of every `git checkout` that has been performed, along with other activity that has resulted in changes to `HEAD`.

If `git reflog` is run with a numerical argument, then the "reflog" will only show the amount of lines specified. So, using `git reflog -4` as an example, the log will only show the 4 most recent changes to `HEAD`.

As with everything else in Git, this information is stored within the `.git` directory. Specifically, this file is located at the path `.git/logs/HEAD`.

This is great to know -- but, what's the practical application? How can this information be used to recover a deleted commit?

## Using "git-reflog" and "git-cat-file" to recover lost history

Consider an example repo with two branches, `prod` and `dev`. Perhaps, it has a history that looks similar to the following:

```
   B    dev
 /
A       prod
```

If the user has `prod` checked out, and they run `git branch -D dev`, the branch `dev` will be deleted. The user is now devastated -- all their hard work on commit `B` has been erased!

However, this user knows about Git's internals, and understands how to use `git reflog` to bring commit `B` back. To do this, the user would run `git reflog`, returning an output similar to the following:

```
<HASH_A> (HEAD -> prod) HEAD@{0}: checkout: moving from dev to prod
<HASH_B> HEAD@{1}: commit: <COMMIT_TITLE_OF_B>
<HASH_A> (HEAD -> prod) HEAD@{2}: checkout: moving from prod to dev
```

This would allow them to reference the SHA of commit `B`. As described in [Git's internal file structure](git-internal-file-structure.md#the-files-are-in-the-computer), the user can run `git cat-file -p` in combination with this hash to walk the tree back to the files that were changed in this commit. Then, again using `git cat-file -p`, they can return the entire contents of the file(s) that were deleted. The user is now able to recover all their lost work!

## Using "git-reflog" and "git-merge" to recover lost history

Another way to perform this same outcome is to use `git reflog` in combination with `git merge`.

As described in [recovering lost history with git-reflog and git-cat-file](#recovering-lost-history-with-git-reflog-and-git-cat-file), the user would get the SHA value for commit `B` by using `git reflog`. The user can this perform a `git merge` on that hash, as shown below:

```
git merge <HASH_B>
```

Assuming linear history, a fast-forward merge will be performed, and the commit will be merged into branch `prod`. As has been reviewed previously, a branch is just a pointer to a commit. So, by using `git merge`, branch `prod` is being updated to point at commit `B`, even though commit `B` was previously deleted!

When using `git merge` in this way, note that the merging process will pull in all the commits between the tip of `prod` and the commit passed as the argument. As an example, see the following case:

```
   B --- C      dev
 /
A               prod
```

If the command `git merge <HASH_C>` was used while branch `prod` was checked out, then both `B` and `C` would be merged onto `prod`.

## Using "git-reflog" and "git-checkout" to recover lost history

In a different situation, it may be necessary to recover lost history with a combination of `git reflog` and `git checkout`.

As an example, a user may attempt an interactive rebase, but make some unintended changes. Before the user is able to run `git rebase --abort`, the rebase completes. In this case, the user runs `git reflog` and finds the commit hash of the `HEAD` that the branch pointed to before the commit. How does this user get the branch to point back to that commit?

There are a few different ways to accomplish this.

1. The user could take that commit hash and change the `.git/refs/heads/<BRANCH>` file to point to that commit.
1. The user could checkout that commit with `git checkout <COMMIT_HASH>`. Once that commit is checked out, the user can run `git checkout -B <BRANCH>`. This will overwrite the `HEAD` of `<BRANCH>` so that it points to the commit that is currently checked out, effectively reversing the interactive rebase.

For more information, refer to the following section:

- ["git-checkout"](git-branch.md#creating-and-switching-to-new-branches)
- ["Interactive rebasing"](interactive-rebase.md#the-basics-of-an-interactive-rebase)

## Using "git-reflog" and "git-cherry-pick" to recover lost history

What happens in a situation where a user doesn't want to apply all commits between the current tip and another tip?

Consider the following example:

```
   B --- C      dev
 /
A               prod
```

If the user is on branch `prod` (at commit `A`), and only wants to apply commit `C` from `dev`, then neither `git merge <HASH_OF_C>` or `git checkout <HASH_OF_C>` will work, as they both will apply `B` to the history.

In this case, a user's preferred tool will be `git cherry-pick`. For more information, refer to the section on ["git-cherry-pick"](git-cherry-pick.md).

## References

- [thePrimeagen - Everything You'll Need to Know About Git - HEAD](https://theprimeagen.github.io/fem-git/lessons/branches-merges-and-more/head)
    - Information on how `git reflog` works
