# git-commit - Saving Changes to a Repo

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Writing commits in a text editor](#Writing-commits-in-a-text-editor)
- [Setting the date with "git-commit-date"](#Setting-the-date-with-git-commit-date)
    - [A note on setting the date](#A-note-on-setting-the-date)
- [Amending commits with "git-commit-amend"](#Amending-commits-with-git-commit-amend)
- [Patch committing](#Patch-committing)

## Introduction

Once changes have been added to the index with `git add`, they can be committed to the repo with `git commit`. Git commits must have some sort of a message to go along with the commit, and this is usually incorporated by running `git commit -m "<COMMIT_MESSAGE>"`. However, just like with `git add`, there are a handful of additional tricks to `git commit` that increase its usefulness.

## Writing commits in a text editor

Git commits are required to have a message. But, instead of running `git commit -m`, what happens if a user runs just `git commit`?

Without the `-m` switch, `git commit` will open up an interactive session in the system's default text editor so the user can create the commit message. `git commit` will open up whatever program is defined by `$EDITOR` -- for example, a user may have Vim or Neovim set as their system's text editor. Depending on the length and content of a commit message, this is often a far easier way to write and format a commit message, rather than trying to format the message within the command line.

## Setting the date with "git-commit-date"

When using `git commit`, the timestamp associated with the commit is the "timestamp at the moment that the message is applied to the commit". However, there are options that allow the user to set the date as they please.

To do this, run the following command:

```
git commit --date="Apr 20 16:20:42 2024"
```

The format of the date is `MMM D HH:MM:SS YYYY`. There are other options too, such as setting the timezone offset for the timestamp. But, to keep it simple, those are the only parameters needed to set the date.

Disclaimer: this guide is not recommending that the user use `git commit --date` maliciously. However, there can be valid reasons for setting the date and time as a value that's not the current date and time. This guide leaves it up to the user to use this option wisely.

### A note on setting the date

The date that is set with `git commit --date` is the commit's "committer date".

On a site like GitHub, the date that displays with a commit is known as the "author date". When using `git commit`, the "author date" is always the timestamp at the moment the commit is created.

However, it *is* possible to change the "author date" to be the same as the "committer date" -- it just requires the user run `git rebase --committer-date-is-author-date -i <HASH_OF_PREVIOUS_COMMIT>`. Note that `<HASH_OF_PREVIOUS_COMMIT>` will refer to the commit before the one for which the user wants to adjust the author date.

When `git rebase -i` operates on the commit, it will have its "author date" changed to the "committer date". For more information, check out the section on [interactive rebases](interactive-rebase.md#An-early-note-on-dates-when-rebasing-interactively).

## Amending commits with "git-commit-amend"

Sometimes, after making a commit, a user will notice an error. Fortunately, there is a way to edit the previous commit through the use of `git commit --amend`.

There's a bunch worth noting about `git commit --amend`:

- As stated, `git commit --amend` only operates on the previous commit.
- If there are changes staged with `git add`, `git commit --amend` allows the user to meld those staged changes into the previous commit. Then, the user will have the opportunity to edit the previous commit's message.
    - If no changes are staged, then `git commit --amend` will only allow the user to edit the previous commit's message.
- If a message is passed with `git commit --amend -m`, then `git commit` will accept that message as the new commit message.
    - Otherwise, without the `-m` switch, the command will open an interactive session in the system terminal.

There are also switches such as `--no-edit` that allow the user to skip editing the commit message. This is useful in a situation where the user wants to edit the author date of the commit, but not the message itself. In that case, the user would run `git commit --amend --no-edit --date`, and then specify the date (as explained in [setting the date with "git-commit-date"](#Setting-the-date-with-git-commit-date)). However, it may be advisable in this situation to run `git commit --amend --date`, as the interactive editor session can be closed without editing, and it allows an opportunity for the user to confirm that the date was set correctly.

Editing commits that are further back in history requires the use of an interactive rebase (`git rebase -i`). For more information on rewording commit messages, check out this section on [interactive rebases](interactive-rebase.md#Rewording-a-commit-message); for more information on editing commits, check out this other section on [interactive rebases](interactive-rebase.md#Editing-the-contents-of-a-commit).

## Patch committing

Just like with `git add`, `git commit` can be used with the `-p` switch. In both cases, `-p` represents patch, and allows the user to select what changes will be committed.

Unlike `git add`, it is not advisable to use `git commit -p` -- instead, use `git add -p` to achieve the same results. The reason is simple: it's better to stage files using `git add -p`, and then commit them.

When using `git add -p`, any error made in hunk selection can easily be remedied with a `git restore` or a `git rm --cached`. If `git commit -p` is used, then the only way to back out is by leaving the commit message blank. And, if `git commit -pm <COMMIT_MESSAGE>` is used, then the only hope is that the incorrectly selected hunk isn't the last hunk. There are options when patching to return to the previous hunk; but, if the error was made on the final hunk, then the commit message is applied and the commit will complete. For these reasons, `git add -p` is the better way to patch in changes for commit.
