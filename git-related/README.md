# Git Good at Git

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

- [Terms and Definitions](terms-and-definitions.md)

- [git-config - Setting Up a New Git Environment](git-config.md)

- [git-init - Initializing a New Git Repository](git-init.md)

- [git-status - Viewing the Status of the Index](git-status.md)

- [git-add - Staging Changes to be Committed](git-add.md)

- [git-commit - Saving Changes to a Repo](git-commit.md)

- [git-log - Viewing a History of Changes](git-log.md)

- [Git's Internal File Structure](git-internal-file-structure.md)

- [git-branch - Separating Development from Production](git-branch.md)

- [git-merge - Bringing Changes Over to Other Branches](git-merge.md)

- [git-rebase - Guaranteeing Linear History](git-rebase.md)

- [git-reflog - Recovering Lost History](git-reflog.md)

- [Working with Remote Repositories in Git](git-remote.md)

- [git-stash - Saving Changes Without Commiting](git-stash.md)

- [git-restore - Restoring Files to the State of the Working Tree](git-restore.md)

- [git-rm - Removing Files from the Working Tree and Index](git-rm.md)

- [Resolving Merge Conflicts](resolving-merge-conflicts.md)

- [Interactive rebases - Editing Commit History](interactive-rebase.md)

- [git-bisect - Testing to Find a "Breaking Change" in a Branch](git-bisect.md)

- [git-revert - Reverting Changes with a New Commit](git-revert.md)

- [git-reset - Resetting the State of a Branch](git-reset.md)

- [git-diff - Viewing Differences in the Command Line](git-diff.md)

- [git-difftool - Viewing Differences using a Text Editor](git-difftool.md)

- [References](#References)

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
