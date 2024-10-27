# git-rebase - Guaranteeing Linear History

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [How to perform a rebase](#How-to-perform-a-rebase)
- [The drawbacks of rebasing](#The-drawbacks-of-rebasing)
- [References](#References)

## Introduction

`git rebase` is one of the most powerful Git tools in the toolkit, assuming the user knows how to use it correctly. There are a few different ways to use `git rebase` -- this section will strictly cover the "non-interactive" version. For information on how to use `git rebase -i`, check the section on [interactive rebasing](interactive-rebase.md).

The primary benefit of rebasing, over merging, is that rebasing always creates linear history. As the name implies, `git rebase` will change the base of a branch to match the current tip of another branch.

Consider the following example project history:

```
   B --- C      dev
 /
A --- D --- E   prod
```

After checking out branch `dev`, running `git rebase` on branch `prod` will result in the following history:

```
              B --- C   dev
             /
A --- D --- E           prod
```

What is happening under the hood is that commit `B` has had the parent commit changed to `E`, instead of commit `A`. Git checks out the latest commit on `prod`, and then plays back all the commits from `dev`, in order. Git walks the tree of commits forward from `E` until it reaches `C`, creating the history shown above.

The grand result of rebasing is that, when it comes time to merge `dev` into `prod`, the user can perform a "fast-forward merge", resulting in linear history and **no merge commits**!

## How to perform a rebase

Using `git rebase` is also relatively simply, assuming the user understands how rebasing works.

In the case of a rebase, the user will check out `dev` and rebase `prod` with `dev`, instead of checking out `prod` and merging `dev` into it. In this case, branch `prod` is now the "target branch", and branch `dev` is the "source (current) branch". Note that this is essentially the converse of a merge, where `dev` was the "target branch" and `prod` was the "source branch".

So, after checking out branch `dev`, the user would run the following command:

```
git rebase prod
```

Running `git log --decorate --oneline --graph` after performing a rebase produces the following output:

```
* <HASH_C> (HEAD -> dev) C
* <HASH_B> B
* <HASH_E> (prod) E
* <HASH_D> D
* <HASH_A> A
```

As has been stated before, the history is linear because `dev` no diverges from `prod` -- the tip of `prod` is now the base of `dev`. Hence, "rebase".

## The drawbacks of rebasing

The primary drawback to rebasing is that it alters the commit history of a branch. In the case of the above example, the history of branch `dev` was `A --- B --- C` before the rebase. After rebasing, the history is now `A --- D --- E --- B --- C`. This means that the commit SHAs for `B` and `C` will change. While this is intentional, and can provide many benefits, it can also create issues.

For example, when a remote repo is involved (such as with GitHub or GitLab), rebasing can result in a `git push --force` being necessary to update the remote.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Merge and Rebase](https://theprimeagen.github.io/fem-git/lessons/branches-merges-and-more/merge-and-rebase)
    - A deep dive into `git rebase`, with examples and "practice problems"
