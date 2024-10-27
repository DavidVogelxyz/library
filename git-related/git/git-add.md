# git-add - Staging Changes to be Committed

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Staging changes with "git-add"](#Staging-changes-with-git-add)
- [Patch adding](#Patch-adding)
- [References](#References)

## Introduction

To add changes to a repo so they can be committed, run `git add`. While `git add` is a very simple command, there are a few tricks to be aware of that can enhance its use.

## Staging changes with "git-add"

The basic syntax for `git add` is as follows:

```
git add <PATH>
```

Most users are aware that substituting `<PATH>` with `.` will add all files in the current working directory (and its subdirectories). However, `<PATH>` can also be a path to a file, a directory, or a pattern -- `git add` will add to the index every matching file that can be staged.

## Patch adding

A very useful switch for `git add` is `-p`, which represents "patch". `git add -p` allows a user to make decisions on *what parts* of a file should be staged.

Consider the following situation: a user makes some edits to two sections of a file. For many reasons, the user may want those changes to be part of separate commits. By running `git add -p <PATH>`, the file will be broken down into sections called "hunks", and the user will be given an interactive session where they can choose to add a hunk with `y` or `n`.

When "patch adding", the user has more options available than just `y` and `n`.

For example, there is also the `s` option. When `git add -p` is run on a file, Git will group changes based on proximity. But, sometimes, the changes are close enough that they're put into the same hunk, even though the user wants to select `y` for one part and `n` for the other. This is where the `s` option becomes useful. `s` represents "split", and it allows the user to take a hunk that contains two changes and split them into smaller hunks. Note that `s` is only available for changes that have at least one unedited line between them.

Another available option is `d`, representing "drop". In some situations, a file may contain many changes, and the user has already selected `y` for all the hunks that should have been added to the index. Instead of having to press `n` until the end of the `git add -p` command, the user can simply select `d`, and all other changes will be dropped from the selection process.

Yet another option available is `q`, representing "quit". The difference between `q` and `d` is quite subtle, and involves other options that allow the user to skip a hunk until later. `q` will quit `git add`, and will drop everything, including skipped hunks; `d` drops any hunk that comes after the dropped hunk, returning to any skipped hunks.

## References

- [YouTube - ChaelCodes - Stop Using git add .](https://www.youtube.com/watch?v=u3NG1966zso)
    - Reference for `git add -p`
- [NuclearSquid - git add --patch and --interactive](https://nuclearsquid.com/writings/git-add/)
    - Reference for other options for `git add -p`
