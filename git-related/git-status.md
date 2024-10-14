# git-status - Viewing the Status of the Index

[Back to the home page](README.md)

## Table of contents

- [Introduction](#Introduction)
- [Using an alias with "git-status"](#Using-an-alias-with-git-status)
- [References](#References)

## Introduction

`git status` is a relatively simple and well-known command. It allows the user to check the status of the index against the working tree, and will display all untracked and tracked files, and all staged and unstaged changes.

## Using an alias with "git-status"

For how often `git status` is used, it can be useful to set a global alias in the `~/.gitconfig` file to reduce the amount of times the command is typed. For more information on how to set a global alias for `git status`, refer to the section on ["git-config"](git-config.md#Global-configs-for-Git-aliases).

In short, it's possible to set `s` as an alias for status. Therefore, the user would only need to type `git s` to run the `git status` command.

## Getting similar information for other remotes

For more information on remotes, refer to the section on [working with Remote Repositories in Git](git-remote.md).

`git status` will display information on the `remote` that is currently being tracked for that branch. But, what if the user has multiple `remotes`, and wants to check how many commits forward or behind the branch is against a different `remote` than the one being tracked?

In this case, it's possible to do this with the following command:

```
git rev-list --left-right --count <BRANCH>...<REMOTE>/<BRANCH>
```

This will return an output similar to the following:

```
5      0
```

What this tells the user is that the current branch is 5 commits ahead of `<REMOTE>/<BRANCH>` and 0 commits behind.

While not as verbose as `git status`, this command will allow the user to get this information, which is not possible otherwise (unless the user changes the `remote` being tracked, just to get this information from `git status`). Since it's a decently lengthy command (even if less frequently used), it also benefits from creating a Git alias for it, which is described in the section on ["git-config"](git-config.md#Global-configs-for-Git-aliases).

## References

- [StackOverflow - git ahead/behind info between master and branch?](https://stackoverflow.com/questions/20433867/git-ahead-behind-info-between-master-and-branch)
    - Reference for using `git rev-list`
