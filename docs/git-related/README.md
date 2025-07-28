# Git Good at Git

[Back to "Docs and guides"](../README.md)

## Introduction

Git is a version control system (VCS), also known as a source-control management (SCM) system. Knowing how to use `git` properly is essential to being a developer and IT professional. This guide will summarize how to perform many useful tasks in Git.

This guide assumes that the user has some basic understanding of 7 basic Git commands: `init`, `status`, `add`, `commit`, `push`, `pull`, and `clone`.

An important note: there are man pages for `git`, as well as for all Git commands. The man page for `git` can be accessed by running the following:

```
man git
```

The syntax for pulling up the man page for any Git command is as follows:

```
man git-add
```

## Table of contents

- [Introduction](#introduction)

- Git

    - [Terms and Definitions](git/terms-and-definitions.md)

    - [git-config - Setting Up a New Git Environment](git/git-config.md)

    - [git-init - Initializing a New Git Repository](git/git-init.md)

    - [git-status - Viewing the Status of the Index](git/git-status.md)

    - [git-add - Staging Changes to be Committed](git/git-add.md)

    - [git-commit - Saving Changes to a Repo](git/git-commit.md)

    - [git-log - Viewing a History of Changes](git/git-log.md)

    - [Git's Internal File Structure](git/git-internal-file-structure.md)

    - [git-branch - Separating Development from Production](git/git-branch.md)

    - [git-merge - Bringing Changes Over to Other Branches](git/git-merge.md)

    - [git-rebase - Guaranteeing Linear History](git/git-rebase.md)

    - [git-reflog - Recovering Lost History](git/git-reflog.md)

    - [git-cherry-pick - Applying Commits from One Branch to Another](git/git-cherry-pick.md)

    - [Working with Remote Repositories in Git](git/git-remote.md)

    - [git-stash - Saving Changes Without Commiting](git/git-stash.md)

    - [git-restore - Restoring Files to the State of the Working Tree](git/git-restore.md)

    - [git-rm - Removing Files from the Working Tree and Index](git/git-rm.md)

    - [Resolving Merge Conflicts](git/resolving-merge-conflicts.md)

    - [Interactive rebases - Editing Commit History](git/interactive-rebase.md)

    - [git-bisect - Testing to Find a "Breaking Change" in a Branch](git/git-bisect.md)

    - [git-revert - Reverting Changes with a New Commit](git/git-revert.md)

    - [git-reset - Resetting the State of a Branch](git/git-reset.md)

    - [git-tag - Creating Tags](git/git-tag.md)

    - [git-diff - Viewing Differences in the Command Line](git/git-diff.md)

    - [git-difftool - Viewing Differences using a Text Editor](git/git-difftool.md)

- GitHub / GitLab

    - [Pull requests (PRs)](github/pull-requests.md)

    - [When dates differ between GitHub's commit history and "git-log"](github/different-dates.md)

    - [The "user commit counter"](github/user-commit-counter.md)

- [References](#references)

## References

- [Frontend Masters - thePrimeagen - Everything You'll Need to Know About Git](https://frontendmasters.com/courses/everything-git)
    - An excellent video course on Frontend Masters taught by thePrimeagen
        - 4 hours of comprehensive content
        - First watched on 2024 June 17, Monday
- [thePrimeagen - Everything You'll Need to Know About Git](https://theprimeagen.github.io/fem-git)
    - The "slide deck" from the Frontend Masters video course
- [YouTube - Chris Titus Tech - How to Use GitHub](https://www.youtube.com/watch?v=v_1iqtOnUMg)
    - One of the first videos referenced about how to use GitHub
- [YouTube - Fireship - 13 Advanced (but useful) Git Techniques and Shortcuts](https://www.youtube.com/watch?v=ecK3EnyGD8o)
    - Another one of the first videos referenced about how to use Git
