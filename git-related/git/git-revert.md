# git-revert - Reverting Changes with a New Commit

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Using "git-revert" to remove a previous commit](#Using-git-revert-to-remove-a-previous-commit)
- [References](#References)

## Introduction

`git revert` is different from `git restore`. Where `git restore` will restore a file to a previous state, `git revert` works by creating an inverse commit to address the removal of a previous commit. Essentially, `git revert` is an "anti-commit".

## Using "git-revert" to remove a previous commit

To revert a commit, run the following command:

```
git revert <COMMIT_HASH>
```

Note that `git revert` can result in conflicts. If a conflict occurs, then the process is similar to a conflict with `git rebase` -- the user will choose the changes to keep and discard, add the changes to the index with `git add`, and then proceed with reversion by running `git revert --continue`. After running `git revert --continue`, the user will have an opportunity to write a commit message, explaining which previous commit was reverted.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Revert](https://theprimeagen.github.io/fem-git/lessons/git-gud/revert)
    - A quick summary of how to use `git revert`
