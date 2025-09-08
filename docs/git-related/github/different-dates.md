When dates differ between GitHub's commit history and "git-log"
===============================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [How to revise an author date](#how-to-revise-an-author-date)
- [References](#references)

Introduction
------------

When using tools such as ["git-commit-date"](../git/git-commit.md#setting-the-date-with-git-commit-date) or [interactive rebases](../git/interactive-rebase.md), and pushing those changes to a GitHub or GitLab repo, a user may encounter a situation where the date shown by `git log` is not the same as the date shown in GitHub's commit history.

Discussed more in [this section](../git/git-commit.md#a-note-on-setting-the-date), there are two different dates that are associated with a commit: the "committer date" and the "author date". The "committer date" is the date that the commit was made, while the "author date" is the date the commit was authored. While `git log` displays the committer date, GitHub's commit history displays the author date.

This distinction can present issues when a user attempts to change the committer date in an effort to change the date displayed on GitHub.

How to revise an author date
----------------------------

The best way to revise an author date on GitHub is to perform the following steps:

- First, change the committer date of the commit by running `git commit --amend --date`, as described in [this section](../git/git-commit.md#setting-the-date-with-git-commit-date).
    - In order to amend a commit that is not the previous commit, an interactive rebase is required.
- Next, change the author date to the committer date by running `git rebase --committer-date-is-author-date -i`, as described in [this section](../git/interactive-rebase.md#an-early-note-on-dates-when-rebasing-interactively).

References
----------

- [YouTube - The Modern Coder - Learn how to rewrite Git history - Amend, Reword, Delete, Reorder, Squash and Split](https://www.youtube.com/watch?v=ElRzTuYln0M)
    - Reference for using `git rebase -i`
- [StackOverflow - git rebase without changing commit timestamps](https://stackoverflow.com/questions/2973996/git-rebase-without-changing-commit-timestamps)
    - Reference for using `git rebase --committer-date-is-author-date -i`
- [StackOverflow - How can one change the timestamp of an old commit in Git?](https://stackoverflow.com/questions/454734/how-can-one-change-the-timestamp-of-an-old-commit-in-git/5017265#5017265)
    - Reference for "changing the committer date and author date"
