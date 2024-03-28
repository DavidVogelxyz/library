# Git

NB: For any `git config` command, omitting the "--global" or "--local" option will set the config local to the current working directory (repo).

## Table of contents

- [Setting up a new Git repository](#Setting-up-a-new-Git-repository)
- [How to set up a Git repo with GPG signing](#How-to-set-up-a-Git-repo-with-GPG-signing)
- [The correct way to do a rollback](#The-correct-way-to-do-a-rollback)
- [Patch adding and patch committing](#Patch-adding-and-patch-committing)
- [Using diff](#Using-diff)
- [Using restore](#Using-restore)

## Setting up a new Git repository

There are a couple of steps to follow to set up a new Git repo, especially if using a new server/computer/VM.

First, set the user's name and the user's e-mail. To do this system-wide (globally), use the following commands:

```
git config --global user.name $NAME
git config --global user.email $EMAIL_ADDRESS
```

To do the same for only the current repo, use the following commands:

```
git config --local user.name $NAME
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

## The correct way to do a rollback

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

## Patch adding and patch committing

Sometimes, I don't want to add or commit all the changes in a file. Rather, I only want to commit *some* of the changes made to a file.

There's a really great flag for this: "-p"

The "p" stands for "patch," and it's the solution to this problem.

An example use case is making some changes to a vimrc file. There have been times when I have some changes to that vimrc file that I don't want to commit. However, one day, I make a small change that I *do* want to commit. Instead of trying to fix the vimrc file so that the only changes are the ones I want to commit, I can use a patch commit instead:

```
git commit -pm "commit message"
```

NB: this should be obvious, but if the example vimrc file was staged using `git add`, then the "-p" won't work as intended. This is because all the changes to that file have already been staged. In order for the "-p" to work as intended, the file has to be a tracked file without being a staged file.

By running `git commit -p`, git will look at all staged files that have changes, and will go through chunks of the file to ask which chunks should be staged. This gives me some discretion about what changes should be included in the commit, and which should not.

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
