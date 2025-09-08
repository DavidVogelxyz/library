git-merge - Bringing Changes Over to Other Branches
===================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [How to perform a three way merge](#how-to-perform-a-three-way-merge)
- [How to perform a fast-forward merge](#how-to-perform-a-fast-forward-merge)
- [What about rebase?](#what-about-rebase)
- [References](#references)

Introduction
------------

As discussed in the section on ["git-branch"](git-branch.md), sometimes a user will want to bring changes from one branch over to another.

An example use case is the following: a user has a main production branch (prod), and another branch for development (dev). Branch `dev` diverges from `prod` at `A`, and some development work occurs, where commits `B` and `C` are created. However, in the meantime, `prod` is updated, and commits `D` and `E` are added to the branch. Thus, the state of the project history will look something like the following:

```
   B --- C      dev
 /
A --- D --- E   prod
```

What can the user do to get the commits in `dev` over to `prod`? One way to accomplish this is through the use of `git merge`.

A "merge" is an attempt to combine two commit histories together that share a common ancestor, and have since diverged. This "common ancestor" is sometimes known as the "merge base" -- the "best common ancestor" is simply the most recent of the common ancestors. By merging, Git will take the tip of `dev` (`C`) and the tip of `prod` (`E`), and create a new commit that brings the two tips together. The branch on which this new commit exists is known as the branch that is "being merged into".

How to perform a three way merge
--------------------------------

Using `git merge` is very simple. But, it's important to understand the fundamentals of how merging works.

A three way merge is a type of merge that produces a merge commit. Referencing the above example, to merge `dev` into `prod`, the user should have `prod` checked out (in other words, `prod` should be the active branch). Then, the user would run the following command:

```bash
git merge dev
```

In this case, `dev` would be the "source branch", and `prod` would be the "target branch". So, `git merge` works by bringing the diverging commits of the "source branch" into the the branch the user currently has checked out (the "target branch"). After running the command, an editor (ex. Vim) would open, allowing the user to add a commit message to the merge commit.

As discussed in the section on ["git-log"](git-log.md#git-log-parents), running `git log --decorate --oneline --graph --parents` would produce an output similar to the following:

```
*   <HASH_MERGE_COMMIT> <HASH_E> <HASH_C> (HEAD -> prod) Merge branch 'dev' into prod
|\
| * <HASH_C> <HASH_B> (dev) C
| * <HASH_B> <HASH_A> B
* | <HASH_E> <HASH_D> (prod) E
* | <HASH_D> <HASH_A> D
|/
* <HASH_A> A
```

Notice that, in the first line, `<HASH_MERGE_COMMIT>` is directly followed by `<HASH_E>`, and `<HASH_C>` follows after. This is one indication that `C` was merged into `E` to create the merge commit. Another indicator is the commit message -- `merge branch 'dev' into prod`.

This type of merge is commonly referred to as a "three way merge" -- this is reference having "two paths" in the past, and "one path" into the future.

How to perform a fast-forward merge
-----------------------------------

Instead of the above example, imagine that the project history looks like the following:

```
              X - Y     dev
             /
A --- D --- E           prod
```

If the user checks out `prod`, and runs `git merge dev` in this instance, the resulting commit history would look like the following:

```
* <HASH_Y> (HEAD -> prod, dev) Y
* <HASH_X> X
* <HASH_E> E
* <HASH_D> D
* <HASH_A> A
```

This is known as a "fast-forward merge".

Note the differences in this instance. First, there is no merge commit. Because the parent of `X` was `E`, instead of `A`, there is no diverging history, and thus no need for a merge commit. Instead, the user has "linear history". Also, the tips of both `prod` and `dev` now point to `Y`. In the case of a "three way merge" with a merge commit, the tip of `prod` would point to the merge commit (`E`), while the tip of `dev` remains pointing at `C`.

What about rebase?
------------------

With an understanding of "three way merges" and "fast-forward merges", a user can get commits that are on one branch over to another branch. In the case of a "three way merge", a merge commit is created, documenting the converging of two different branches of commit history. In contrast, a "fast-forward merge" has no need to create a merge commit, and can instead create linear history, because there was never any divergence.

So, what's the point of `git rebase`? Find out in the [next section](git-rebase.md)!

References
----------

- [thePrimeagen - Everything You'll Need to Know About Git - Merge and Rebase](https://theprimeagen.github.io/fem-git/lessons/branches-merges-and-more/merge-and-rebase)
    - A deep dive into `git merge`, with examples and "practice problems"
