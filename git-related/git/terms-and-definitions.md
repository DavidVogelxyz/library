# Terms and Definitions

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Terms, and their definitions](#Terms-and-their-definitions)
- [References](#References)

## Introduction

This section of the guide is dedicated to listing the terms and definitions that get used throughout this guide.

## Terms, and their definitions

- **Repository ("repo")**: A project tracked by Git.

- **Commit**: A state that represents the entire project.

- **Working tree**: The current state of the Git repo, without any new changes. When `git init` is run on a directory, the working tree is bare. When `git clone` is run, the working tree is the most recent commit. The working tree is compared against the index to differentiate changes.

- **Index ("staging area")**: A data structure that defines the differences between the working tree and the new state. When `git add` is run on a file, that file is added to the index, and the change becomes "staged". Files are not tracked by Git until they are either in the working tree, or added to the index with `git add`.

- **Untracked file**: A file that has never been staged (indexed), and is not tracked by Git. Git does not have any information on untracked files. Deleting, or removing, an untracked file will result in lost information. A new file is always untracked at first, and then is added to the index with `git add` and becomes an "indexed file".

- **Indexed (staged) file**: A file that has been added to the index (staging area), and can be committed. A file ***must*** be staged in order to be committed. When a file is committed with `git commit`, it becomes a tracked file.

- **Tracked file**: A file that has already been committed to the repo. A tracked file without changes cannot be staged again; but, a tracked file with a change can have the new state staged so it can be commited. While Git has information on a tracked file, it will ***not*** have information on the change until the changes are staged.

- **Branch**: A named object that points to a commit. Branches allow divergence in a repo. For more information, check out the section on ["git-branch"](git-branch.md).

- **Reference ("ref")**: A pointer to a commit. A "ref" may point to a branch or a tag, but the reference ultimately points to a commit.

- **HEAD**: A specific pointer that points to the tip of a branch. For more information, check out the section on [Git's Internal File Structure](git-internal-file-structure.md#What-is-HEAD).

- **A "remote"**: In the reference frame of any Git repo, a "remote" is the same Git repo that exists at a different path. While this often refers to a version of the repo on GitHub or GitLab, the path can be anywhere that's not the local path. For more information, check out the section on ["git-remote"](git-remote.md).

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Introduction to Git](https://theprimeagen.github.io/fem-git/lessons/intro/intro)
    - Basic Git introduction
