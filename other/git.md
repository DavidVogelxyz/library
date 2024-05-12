# Git

NB: For any `git config` command, omitting the "--global" or "--local" option will set the config local to the current working directory (repo).

## Introduction

`Git` is a version control system (VCS), also known as a source-control management (SCM) system. Knowing how to use `git` properly is essential to being a developer and IT professional. This guide will summarize how to perform many useful tasks in `git`.

## Table of contents

- [Introduction](#introduction)
- [Setting up a new Git repository](#setting-up-a-new-git-repository)
- [How to set up a Git repo with GPG signing](#how-to-set-up-a-git-repo-with-gpg-signing)
- [Rolling back commits](#rolling-back-commits)
    - [The correct way to do a rollback](#the-correct-way-to-do-a-rollback)
    - [A more correct way to do a rollback](#a-more-correct-way-to-do-a-rollback)
        - [Fixing commit messages](#fixing-commit-messages)
        - [Fixing files that were previously committed](#fixing-files-that-were-previously-committed)
- [Patch adding and patch committing](#patch-adding-and-patch-committing)
    - [Patch committing](#patch-committing)
    - [Patch adding](#patch-adding)
- [Using diff](#using-diff)
- [Using restore](#using-restore)
- [Using stash](#using-stash)
- [References](#references)

## Setting up a new Git repository

There are a couple of steps to follow to set up a new Git repo, especially if using a new server/computer/VM.

First, set the user's name and the user's e-mail. To do this system-wide (globally), use the following commands:

```
git config --global user.name $NAME
```

```
git config --global user.email $EMAIL_ADDRESS
```

To do the same for only the current repo, use the following commands:

```
git config --local user.name $NAME
```

```
git config --local user.email $EMAIL_ADDRESS
```

If the repo is going to be pushed to GitHub, as an example, the "origin" URL must be set within the repo. To do this, use the following command as a guide, substituting in the correct URL for the repo.

```
git remote add origin https://github.com/DavidVogelxyz/library.git
```

Next, set the branch, using "-M" to force the name change:

```
git branch -M master
```

Finally, the repo can be pushed using the following command:

```
git push -u origin master
```

## How to set up a Git repo with GPG signing

To set GPG signing of Git commits system-wide, use the following command:

```
git config --global commit.gpgsign true
```

To set GPG signing locally (repo-specific), use the following command:

```
git config --local commit.gpgsign true
```

Once the GPG signing setting is turned on, Git needs to know which key to use to sign the commits. As with the "commit.gpgsign" option, this can be done either globally or locally. In addition, when giving Git a key ID, both short and long key IDs will work, so long as it's unique (as compared to other keys in the keyring).

To do this system-wide, use the following command:

```
git config --global user.signingkey $KEY_ID
```

To set a key local to a specific repo, use the following command:

```
git config --local user.signingkey $KEY_ID
```

## Rolling back commits

Sometimes, errors are made when committing to `git` and GitHub, and edits are required.

Originally, the section "[the correct way to do a rollback](#the-correct-way-to-do-a-rollback)" was written as a way to perform these changes. But, it's actually a clunky way of making these changes. A better way to rollback commits is described in "[a more correct way to do a rollback](#a-more-correct-way-to-do-a-rollback)".

For now, the original method will stay in the guide as a reference, as it does work. However, it may be removed in the future.

### The correct way to do a rollback

There have been times when I've made commits in a less-than-preferred manner.

An example of this is when I make a commit, and immediately notice a small typo. I find it irritating that the typo wasn't addressed in the original commit, and I'm reluctant to to push an additional commit just to correct a minor typo.

In these instances, I would vastly prefer to rollback to the previous "good" commit, and then push one correct commit.

I have found the best way to address this is the following.

```
git reset --hard $hash_of_desired_commit
```

The above command will rollback the repo to the state it was in before I pushed the commit with the minor error. It also reverts the HEAD back to that commit. Then, I perform the following command:

```
git clean -df
```

The above command cleans up the repo so that everything is exactly the way it was before the commit took place. I then make the changes I wanted to be included in the "one good commit."

Finally, I commit the changes and then push them using the following command:

```
git push -fu origin master
```

Normally, I use `git push -u origin master`. However, in this case, I require a --force flag to overwrite the commit that was already pushed to the public repo. While I could write the flags as `-uf`, I find the other way to be more amusing.

### A more correct way to do a rollback

Recently, I learned that the method described in the [previous section](#the-correct-way-to-do-a-rollback) is actually a sub-optimal way of performing these changes. A better method is described below.

#### Fixing commit messages

A prime example of a use case for this method is the following: wanting to change a commit message (to fix a typo) for a commit that is no longer the previous commit.

Use the following command:

```
git rebase -i $HASH_OF_COMMIT
```

Note: `$HASH_OF_COMMIT` can either be a hash, or it can be a reference to the commit, such as `HEAD^^` or `HEAD~2`.

This will open up an interactive window using the default terminal text editor. By choosing the relevant commit, changing "pick" to "reword", and saving, a second interactive window will open. Now, it is possible to edit the typos and fix the commit message, and `git` will rebase the commits.

If the commits were pushed to a remote location, it is possible to overwrite the published commits with the new commits using the following command:

```
git push -fu origin master
```

#### Fixing files that were previously committed

What if the change isn't a commit message, but a typo in a committed file?

Again, use the following `rebase` command:

```
git rebase -i $HASH_OF_COMMIT
```

Note: `$HASH_OF_COMMIT` can either be a hash, or it can be a reference to the commit, such as `HEAD^^` or `HEAD~2`.

This will open up an interactive window using the default terminal text editor. By choosing the relevant commit, changing "pick" to "edit", and saving, a second interactive window will open. Now, it is possible to edit the typos and fix the committed files, and `git` will rebase the commits.

If the commits were pushed to a remote location, it is possible to overwrite the published commits with the new commits using the following command:

```
git push -fu origin master
```

## Patch adding and patch committing

As with the rollback section, I have learned better how to use patch adding and patch committing. The original method is described in "[patch committing](#patch-committing)", and the improved way is outlined in "[patch adding](#patch-adding)".

The simple explanation is this: it's better to stage files for commit using `git add -p` before committing, rather than using `git commit -pm`. Potentially, and likely, it will lead to less errors. It's not "wrong" to skip a step and use `git commit -pm`; it's just sub-optimal.

### Patch committing

Sometimes, I don't want to add or commit all the changes in a file. Rather, I only want to commit *some* of the changes made to a file.

There's a really great flag for this: "-p"

The "p" stands for "patch," and it's the solution to this problem.

An example use case is making some changes to a vimrc file. There have been times when I have some changes to that vimrc file that I don't want to commit. However, one day, I make a small change that I *do* want to commit. Instead of trying to fix the vimrc file so that the only changes are the ones I want to commit, I can use a patch commit instead:

```
git commit -pm "commit message"
```

NB: this should be obvious, but if the example vimrc file was staged using `git add`, then the "-p" won't work as intended. This is because all the changes to that file have already been staged. In order for the "-p" to work as intended, the file has to be a tracked file without being a staged file.

By running `git commit -p`, git will look at all staged files that have changes, and will go through hunks of the file to ask which hunks should be staged. This gives me some discretion about what changes should be included in the commit, and which should not.

### Patch adding

As described above, there's nothing wrong with using `git commit -pm`; but, it's probably more optimal to use `git add -p` to stage the files first, and then use `git commit -m` to commit the files.

The main reason is that `git add -p` can be used repeatedly before making a commit. If `git commit -pm` is used, then an error in staging hunks may inadvertently result in the wrong hunks being committed.

Another worthwhile note is this: there are more options to `-p` than just "y" and "n". This is true for both `git commit -pm` and `git add -p`.

Sometimes, if the changes are close enough, `git` will include in "one hunk" changes that should be split into two hunks. This way, one hunk can be added, while other is not added.

- In this case, and there are lines between the two changes, simply use the "s" option to split the hunk into two smaller hunks.

In other situations, there's a bunch of changes in a file, but only one needs to be staged. Instead of having to press "n" repeatedly, just use "d". All other changes will be skipped, saving time.

Patch adding, in addition to the "s" and "d" options, gives the user even more discretion in deciding what changes to commit, and which to leave behind for a future commit.

## Using diff

`git diff` is a great way to directly compare changes made in a repo.

To compare differences for all files against the most recent commit, use the following command:

```
git diff HEAD
```

To compare differences for **ONLY THE UNSTAGED FILES** against the most recent commit, use the following command:

```
git diff
```

To compare differences for **ONLY THE STAGED FILES** against the most recent commit, use the following command:

```
git diff --staged
```

Another way to compare differences for **ONLY THE STAGED FILES** against the most recent commit, use the following command:

```
git diff --cached
```

To compare differences for **a specific file** against the most recent commit, use the following command:

```
git diff HEAD $FILE
```

To compare differences **between two branches**, use the following command. Note that the branches can be switched; however, the changes will be read in reverse. Put another way, any additions made from an old branch to a new one will look like deletions.

```
git diff $BRANCH_OLD $BRANCH_NEW
```

## Using restore

`git restore` is a command that can sometimes be a bit tricky to use, as an incorrect usage can potentially mean getting rid of a file that was meant to only be unstaged.

To unstage a file, use the following command on a *staged* file:

```
git restore --staged $FILE
```

To restore a file to the version found in the most recent commit and discard the "working changes", use the following command:

```
git restore $FILE
```

## Using stash

Sometimes, changes to the working directory need to be saved and hidden away, so that other work can be done.

An example use case is the following: a rebase to a previous commit is required, but new changes have already been added to the working directory. Git will mention that those changes cannot take place while those differences exist. In this case, the new changes need to either be committed, or they can be stashed.

To stash changes to the working directory, use the following command:

```
git stash
```

Note that `git stash` is a shorthand for `git stash push`: both commands will push the current changes into a stashed entry.

To recover the changes that have been stashed, use the following command:

```
git stash pop
```

This command will "pop" the most recently stashed entry back onto the current commit.

## References

- [Git SCM documentation - Getting Started - First-Time Git Setup](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)
    - Reference for `user.name` and `user.email` commands
- [FreeCodeCamp - A Beginner's Guide to Git](https://www.freecodecamp.org/news/a-beginners-guide-to-git-how-to-create-your-first-github-project-c3ff53f56861/)
    - Reference for setting up a `git` repository with GitHub
    - Specifically, [this image](https://cdn-media-1.freecodecamp.org/images/cxRrZUe-tW2Wkn0WUg-MsN1m1WesvGPlJT7V) for the command: `git remote add origin $URL`
- [Atlassian - Setting up a repository](https://www.atlassian.com/git/tutorials/setting-up-a-repository)
    - Another reference for setting up a `git` repository with GitHub
    - Again, for the command: `git remote add origin $URL`
- [GitHub Documentation - Signing Commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
    - Reference for `commit.gpgsign` command
- [GitHub Documentation - Telling Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
    - Reference for `user.signingkey` command
- [StackOverflow - How do I revert a Git repository to a previous commit?](https://stackoverflow.com/questions/4114095/how-do-i-revert-a-git-repository-to-a-previous-commit)
    - Reference for [the correct way to do a rollback](#the-correct-way-to-do-a-rollback)
- [YouTube - The Modern Coder - Learn how to rewrite Git history - Amend, Reword, Delete, Reorder, Squash and Split](https://www.youtube.com/watch?v=ElRzTuYln0M)
    - Reference for using `git rebase -i` ([a more correct way to do a rollback](#a-more-correct-way-to-do-a-rollback))
- [YouTube - ChaelCodes - Stop Using git add .](https://www.youtube.com/watch?v=u3NG1966zso)
    - Reference for [patch adding and patch committing](#patch-adding-and-patch-committing)
- [NuclearSquid - git add --patch and --interactive](https://nuclearsquid.com/writings/git-add/)
    - Reference for other options for `git add -p`
- [Atlassian - Git diff](https://www.atlassian.com/git/tutorials/saving-changes/git-diff)
    - Reference for `git diff`
- [Git SCM documentation - git restore](https://git-scm.com/docs/git-restore)
    - Reference for `git restore`
- [StackOverflow - What is `git restore` and how is it different from `git reset`?](https://stackoverflow.com/questions/58003030/what-is-git-restore-and-how-is-it-different-from-git-reset)
    - Another reference for `git restore`
- [Git SCM documentation - git stash](https://git-scm.com/docs/git-stash)
    - Reference for `git stash`
- [YouTube - Chris Titus Tech - How to Use GitHub](https://www.youtube.com/watch?v=v_1iqtOnUMg)
    - One of the first videos viewed about how to use GitHub
- [YouTube - Fireship - 13 Advanced (but useful) Git Techniques and Shortcuts](https://www.youtube.com/watch?v=ecK3EnyGD8o)
    - Another one of the first videos viewed about how to use Git
