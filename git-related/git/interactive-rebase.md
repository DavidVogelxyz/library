# Interactive rebases - Editing Commit History

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [The basics of an interactive rebase](#The-basics-of-an-interactive-rebase)
    - [An early note on dates when rebasing interactively](#An-early-note-on-dates-when-rebasing-interactively)
- [Squashing multiple commits into one commit](#Squashing-multiple-commits-into-one-commit)
- [Dropping a commit from the history](#Dropping-a-commit-from-the-history)
- [Rewording a commit message](#Rewording-a-commit-message)
- [Editing the contents of a commit](#Editing-the-contents-of-a-commit)
- [Reordering commits](#Reordering-commits)
- [Adding a new commit to the middle of the history](#Adding-a-new-commit-to-the-middle-of-the-history)
- [Changing the date of a previous commit](#Changing-the-date-of-a-previous-commit)
    - [Changing the "committer date" of a previous commit](#Changing-the-committer-date-of-a-previous-commit)
    - [Changing the "author date" of a previous commit](#Changing-the-author-date-of-a-previous-commit)
- [Interactively rebasing the first commit of the history](#Interactively-rebasing-the-first-commit-of-the-history)
- [References](#References)

## Introduction

The [previous section](resolving-merge-conflicts.md) described how to resolve merge conflicts. With that explained, it is time to explain what an "interactive rebase" is, and what the use cases are.

An interactive rebase can be called with the command `git rebase -i`, and it allows a user to use rebase, interactively, to edit past history. This can be extremely useful in a variety of situations. A few examples include:

- Squashing a series of commits into one larger commit
- Dropping a commit from the history
- Rewording the message of a previous commit that cannot be amended with `git commit --amend`
- Editing the contents of a commit (for example, in the case where a commit included sensitive information)
- Reordering commits
- Adding a new commit into the middle of the history
- Changing the date of a previous commit

While some of the above examples are completely benign, some could be used to "fake" history. It is up to the user's discretion to know when it's proper to use these tools.

## The basics of an interactive rebase

Anyone who has resolved a conflict when using `git rebase` has already experienced an interactive rebase. Therefore, the basics will be nearly identical.

To initiate an interactive rebase, run the following command:

```
git rebase -i <COMMIT_HASH>
```

Note that `<COMMIT_HASH>` can also be a reference to `HEAD` -- if the user wants to rebase the previous two commits, then the user could run `git rebase -i HEAD~2`, which refers to "two commits back from `HEAD`". Regardless of whether the argument is a commit hash or a "committish" (ex. `HEAD~2`, or `HEAD^^`), the user will want to give `git rebase -i` the "last good commit". To understand this better, imagine the following example:

```
A --- B --- C --- D
```

If the tip of the branch is commit `D`, and the user wants to edit the contents of commit `C` and reword the message of commit `D`, the user would run `git rebase -i` and pass the commit hash for commit `B`. This is because `B` is the "last good commit". For more information on how to interactively rebase `A`, refer to the section on [interactively rebasing the first commit of the history](#Interactively-rebasing-the-first-commit-of-the-history).

Once `git rebase -i <COMMIT_HASH>` is run, an interactive window will open in the system's text editor, where the user will see a list of all commits that come after the "last good commit". Put another way, this list of commits is the list of commits that will be replayed during the interactive rebase. From here, the user can pick a variety of options for manipulating those commits. This guide will detail those options and how they work. For now, all the user needs to know is that `pick` means "take the commit as it is".

When using options such as `edit`, the user will enter an interactive rebase, similar to resolving a conflict. As changes are made to the commit, they must be added with `git add` to be incorporated into the commit. At any time, the user can abort the interactive rebase with `git rebase --abort`. And, when the user is done editing and is ready to proceed, they will run `git rebase --continue`.

When using an interactive rebase, the hashes of the rebased commits will change. This should make sense -- some part of the history will change, and those changes will propagate throughout the remainder of the replayed commits. Therefore, any commits that have been pushed to a `remote` that have new commit hashes will need to be overwritten with a `git push -f` ("force push"). When interactively rebasing a repo with multiple collaborators, care must be taken when force pushing, so as not to overwrite the changes of the other collaborators.

If an error ever occurs during an interactive rebase, and the user is unable to use `git rebase --abort` since the rebase completed, it is possible to use `git reflog` to find the hash of the previous `HEAD`. Then, the user can checkout that commit and overwrite its `HEAD` with `git checkout -B <BRANCH>` in order to return to the previous state. For more information, refer to the section on ["git-reflog"](git-reflog.md).

### An early note on dates when rebasing interactively

As discussed in the [section on "git commit"](git-commit.md#A-note-on-setting-the-date), Git keeps track of two separate date values: "committer date" and "author date".

Imagine a repo that has a `remote` on GitHub. A user interactively rebases some commits, and checks their work with `git log`. The user would see that the dates of the commits haven't changed from the original "committer dates". However, when the user pushes the changes to GitHub, the dates that GitHub shows are not the dates that `git log` shows. This is because `git log` shows the "committer date", while GitHub shows the "author date".

To address this, and preserve the committer's timestamps, use the following command in place of `git rebase -i`:

```
git rebase --committer-date-is-author-date -i
```

Note that this is a rare instance where the order of the switches matters: `--committer-date-is-author-date` should come *before* `-i`. As an example of why, a shell like `zsh` will autocomplete both switches if `--committer-date-is-author-date` comes before `-i`; otherwise, `-i` will autocomplete but `--committer-date-is-author-date` will not. This may be an indication that `-i` must terminate the command, otherwise subsequent switches will not be passed to the `git rebase` command.

Because the situation with "committer dates" and "author dates" occurs every time `git rebase -i` is run, it is advisable to use the `--committer-date-is-author-date` switch every time `git rebase -i` is run. To simplify this process, a configuration can be added to the user's `~/.gitconfig` (global) file to create an alias for this command:

```
[alias]
    rebase-int = rebase --committer-date-is-author-date -i
```

With this configuration added, the user can run `git rebase-int <COMMIT_HASH>`, and Git will instead run `git rebase --committer-date-is-author-date -i <COMMIT_HASH>`.

## Squashing multiple commits into one commit

One of the options available to a user during an interactive rebase is `squash`, which allows a user to take a commit marked `squash` and meld it into the previous commit. Essentially, for any `n` of commits that are sequentially marked `squash`, the user will end up with `n - 1` commits.

To expand on this, consider a list of commits such as the following:

```
pick commit #1
squash commit #2
squash commit #3
squash commit #4
pick commit #5
squash commit #6
squash commit #7
squash commit #8
```

In this case, commits `2`, `3`, and `4` will be squashed into commit `1`, and commits `6`, `7`, and `8` will be squashed into commit `5`. Therefore, what was originally a series of 8 commits, will become a series of two commits.

Note that `squash` can be abbeviated to `s`, as show below:

```
pick commit #1
s commit #2
s commit #3
s commit #4
pick commit #5
s commit #6
s commit #7
s commit #8
```

Using the above example, once the replay list is set, `git rebase` will begin to rebase these commits. First, an editor window will open, in which the user can edit the commit message for the squashed commit for `1`, `2`, `3`, and `4`. Once that buffer is saved and closed, a new editor window will open, in which the user can edit the commit message for the squashed commit for `5`, `6`, `7`, and `8`. After saving and closing that second buffer, the interactive rebase will complete, and the user can check the result with `git log --oneline`.

Note that squashing commits should **never** result in a conflict, as `git rebase` is going to replay commits one at a time, condensing along the way. Even if two commits change the same line, the replay happens sequentially, so there should never be a conflict.

Squashing is an effective tool that allows a user to make many small commits while working, saving changes and preventing lost work. Then, when the user is ready, they can squash all those small commits into one large commit, which keeps the history clean and makes it easy for another person to read through the history. When squashing, a good habit is to title the commits with `SQUASHME: <MESSAGE>`. This way, it's easy during an interactive rebase to know which commits should be squashed.

## Dropping a commit from the history

Another useful option when rebasing interactively is `drop`. `drop` allows the user to completly remove a commit from the history.

Note that `drop` can be abbreviated to `d`. However, `drop` can also be achieved by removing the commit from the list.

Consider the following example:

```
pick commit #1
pick commit #2
pick commit #3
```

If the user wants to `drop` commit `2`, there are multiple ways to accomplish this.

Using `drop`:

```
pick commit #1
drop commit #2
pick commit #3
```

Using `d`:

```
pick commit #1
d commit #2
pick commit #3
```

Removing the commit:

```
pick commit #1
pick commit #3
```

All three of these methods will remove the commit from the replay list, resulting in an altered history where that commit doesn't exist. There are limited use cases for a `drop`; but, a user should know the option exists.

## Rewording a commit message

Interactive rebases also allow a user to `reword` the commit message of any prior commit, no matter how far back in the history. This is in stark contrast to `git commit --amend`, which only works when the commit to be reworded is the previous commit.

Note that `r` is an abbreviation for `reword`, and functions in the same way.

Consider the following example:

```
pick commit #1
pick commit #2
pick commit #3
```

To reword the commit message of `2`, the user could specify either `reword` or `r`.

Using `reword`:

```
pick commit #1
reword commit #2
pick commit #3
```

Using `r`:

```
pick commit #1
r commit #2
pick commit #3
```

When the replay gets to commit `2`, the system's text editor will open a session, allowing the user to modify the commit message of that commit. Then, after the user saves and closes the buffer, the replay will continue. The changes to a commit message can be confirmed with either `git log` or `git show <COMMIT_HASH>`.

Rewording a commit message is useful in a few use cases. One example is a user that notices a typo in a commit message. If the user cannot abide this error, they can use `reword` to change the commit message. However, as noted earlier, doing so will change the commit hash of the reworded commit, and the hashes of all future commits. For a repo that exists on a `remote` such as GitHub, a `git push -f` ("force push") will be required to sync the changes to the `remote`.

## Editing the contents of a commit

Another feature of interactive rebasing is the ability to `edit` the changes present in a commit.

An obvious (and legitimate) use case is the following: consider a user who is doing some development work locally. The use commits `1`, `2`, and `3`. Before pushing to GitHub, the user notices that they made a mistake. When creating commit `2`, they used `git add .` and then `git commit`, without running `git status` in between. Because of this mishap, the user accidentally committed `secretfile` to commit `2`, which they **do not** want to be visible publicly.

This user could run `git rebase -i` and choose to `edit` commit `2`. Note that, as with other options, `edit` can be abbreviated with `e`. Consider the following example replay list:

```
pick commit #1
pick commit #2
pick commit #3
```

The user could choose either `edit` or `e` to edit commit `2`:

```
pick commit #1
e commit #2
pick commit #3
```

The user will be placed back into the command line in "interactive rebase" mode. Now, the user could run `git rm --cached secretfile` to remove `secretfile` from the index, while keeping the file in the directory. Then, the user would run `git add secretfile`, and confirmed its removal from the index with `git status`. Then, once the user has confirmed that the file will be removed from the index, they can run `git rebase --continue`. Because edits were made to the commit, `edit` will now allow the user to `reword` that commit's message. In the case some part of the edit was addressed in the commit message, Git intelligently allows the user to revise the commit message.

In a situation such as this, the user would likely benefit from adding `secretfile` to their `.gitignore` file for that repo. This can be accomplished with a future commit, but it can also be made part of the `edit` for commit `2`.

Just like with a `reword`, `edit` will change the commit hash of the edited commit, as well the hashes of all future commits. Therefore, a `git push -f` ("force push") would be necessary to sync the changes to any `remote` that has the unaltered commits in their history.

## Reordering commits

Interactive rebases also allow a use to reorder commits. Reordering commits is not often used, but it's good to know that it's possible.

An example of a legitimate use case for reordering is the following: imagine the user from [squashing multiple commits into one commit](#Squashing-multiple-commits-into-one-commit) has a bunch of commits they want to squash. However, the user would prefer to squash `6`, `7`, and `8` into `1` (along with `2`, `3`, and `4`). This may be because `6`, `7`, and `8` are relevant to `1`, and commit `5` is irrelevant to the other `7` commits. The user could reorder these commits for replay, such that `2`, `3`, `4`, `6`, `7`, and `8` can all be squashed into `1`.

So, while choosing which commits to squash, the user would reorder the replay list, as shown below:

```
pick commit #1
squash commit #2
squash commit #3
squash commit #4
squash commit #6
squash commit #7
squash commit #8
pick commit #5
```

During the rebase, Git will replay the commits in that order, allowing the user to squash all the commits into a single commit (`1`), and then replaying commit `5` after.

Note that, in this case, squash *can* result in conflict, if commit `5` makes a change to a file that is also changed in commits `6`, `7`, and `8`.

To extend this idea, consider a replay list such as the following:

```
pick commit #1
pick commit #2
pick commit #3
```

If all 3 commits change different files, there's nothing stopping a user from reordering them, as shown below:

```
pick commit #3
pick commit #2
pick commit #1
```

The only thing to note is that these commits will retain their "committer dates", while having different "author dates". Therefore, the commit dates shown in `git log` will now be out of order. If this is necessary, then the user may want to consider revising the "committer dates", as explained in [changing the date of a previous commit](#Changing-the-date-of-a-previous-commit).

## Adding a new commit to the middle of the history

It is also possible with interactive rebasing to add a new commit to the middle of a history.

Imagine a user wants to add a new file in a distinct commit, and the commit should have taken place in the past. There are a few ways to go about this.

The easiest method would be to create the commit at the tip of the branch, and then reorder that commit so it appears in an earlier part of the history.

Another way is to `edit` the commit before the spot where the new commit should be inserted. This is a great use case for `git stash` -- create the file and add it to the stack, before beginning the interactive rebase. Once the user is returned to the command line, they can add the new file to the index (or pop it off the stack) and run `git commit`. This will create a new commit that comes after the commit marked `edit`.

In both cases, it is likely that the commit will need to have its date changed, so that the history occurs in chronological order.

## Changing the date of a previous commit

A user can also use interactive rebasing to change the date of a commit. Which date the user wants to change will dictate the process.

As with `reword` and `edit`, changing the date of a commit (both for "committer date" and "author date") will affect the hash of that commit, and of all future commits. Therefore, as with those options, a `git push -f` ("force push") may be necessary to sync with a `remote` that contains the unaltered commits in its history.

### Changing the "committer date" of a previous commit

To change the "committer date" of a commit, the user will want to `edit` the commit for which the date will be changed.

Once back on the command line, the user will run `git commit --amend --date="MMM D HH:MM:SS YYYY"`. As described in the section on ["git-commit"](git-commit.md#Setting-the-date-with-git-commit-date), the date specified with this command will become the new "committer date". While the `--no-edit` switch can be passed with this command, running the command without it will allow the user to verify during the commit message rewording state that the date is what the user set. Once verified, the user can exit the editor without making changes to the commit message, and can verify the change again with `git log`.

However, for the purposes of pushing to GitHub (etc.), the user will also want to make sure that the commit's "author date" is also updated to reflect the committer date.

### Changing the "author date" of a previous commit

To change the "author date" of a commit, the user will want to run `git rebase --committer-date-is-author-date -i <COMMIT_HASH>`, where the `<COMMIT_HASH>` is a commit that appears prior to the "commit in question".

When selecting the options for the replay list, the user can leave all commits as `pick`. So long as `git rebase` replays the "commit in question", it will adjust the commit so that the "committer date" is the same as the "author date".

As mentioned before in [an early note on dates when rebasing interactively](#An-early-note-on-dates-when-rebasing-interactively), the effects of changing the "author date" are most apparent on a site like GitHub, where the date displayed alongside a commit is the "author date".

## Interactively rebasing the first commit of the history

As mentioned repeatedly in previous sections, to interactively rebase any commit, `git rebase -i` should be the "last good commit" before the commit to be changed. But, what if the user wants to `reword` or `edit` the first commit of a branch?

To do this, the `--root` switch will need to be passed to the `git rebase -i` command, such as in the following example:

```
git rebase --committer-date-is-author-date --root -i
```

In this specific case, no `<COMMIT_HASH>` is required, as `--root` tells `git rebase -i` to operate starting from before the first commit. Once the user is viewing the replay list, they can select whatever option to interactively rebase the first commit.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Rebasing is Complicated](https://theprimeagen.github.io/fem-git/lessons/going-remote/more-rebasing-and-merging)
    - Short section at the end of this reference that discusses "interactive rebases" and "squashing"
- [YouTube - The Modern Coder - Learn how to rewrite Git history - Amend, Reword, Delete, Reorder, Squash and Split](https://www.youtube.com/watch?v=ElRzTuYln0M)
    - Reference for using `git rebase -i`
- [StackOverflow - git rebase without changing commit timestamps](https://stackoverflow.com/questions/2973996/git-rebase-without-changing-commit-timestamps)
    - Reference for using `git rebase --committer-date-is-author-date -i`
- [StackOverflow - Reordering of commits](https://stackoverflow.com/questions/2740537/reordering-of-commits/58087338#58087338)
    - Reference for "reordering commits"
- [StackOverflow - How can one change the timestamp of an old commit in Git?](https://stackoverflow.com/questions/454734/how-can-one-change-the-timestamp-of-an-old-commit-in-git/5017265#5017265)
    - Reference for "changing the committer date and author date"
