# The "user commit counter"

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#Introduction)
- [Removing the "additional commits"](#Removing-the-additional-commits)
    - [How to remove the "additional commits"](#How-to-remove-the-additional-commits)

## Introduction

When a user interactively rebases commits on a public repo and then performs a force push, GitHub will register every new commit hash as a "new commit".

As an example, imagine a user creates 6 new commits, and then performs an interactive rebase to edit the commit message for the "first new commit" [the commit that is 6 behind `HEAD` (`HEAD~6`)]. When the user performs a force push, the user's profile will show that the user created another 6 commits, for a total of 12 commits.

When using interactive rebases frequently, or making changes to commits that are far back in the history, this can create a "user commit counter" that is far higher than the real number of commits made.

The user may not want this, as they may want the "user commit counter" to reflect the true number of commits created, instead of an inflated value.

## Removing the "additional commits"

One way to clean up the "user commit counter" is to delete the repo and replace it. Depending on how many other active users are making use of the repo, this can have bad consequences. If the user that owns the repo is careful, these consequences can be mitigated relatively easily.

The "delete and replace" method works because GitHub stores, for every repo, a reflog of the changes to `HEAD`. As the interactive rebases and force pushes occur, the reflog gets filled with changes to `HEAD`. When GitHub references the repo's reflog for a count of how many commits a user made, the system is simply counting all of the different commit hashes associated with the user. By deleting the repo, creating a new repo, and pushing the repo and its branches to the new instance of the repo, the commit history is retained while the reflog is not.

### How to remove the "additional commits"

To remove these "additional commits", first make sure that both the local and `remote` repos are up-to-date.

Then, rename the GitHub repo to something similar, such as `REPOSITORY2`. Then, create a new, empty repo on GitHub with the original name (`REPOSITORY`).

Now, perform a `git push` on the repo. This will push the current branch and its commit history to the new GitHub repo. It isn't necessary to change any of the `remote` information, because the URI for the new, empty repo is the same as the original version. All commits from the past will line up with their original author dates, and will not appear as a bunch of commits from the same day.

Next, copy over from the original repo any "repo descriptions" (etc) that should be kept. Finally, delete the repo named `REPOSITORY2`.

This process will essentially keep the entire repo intact, without affecting the commit history. Only the reflog will be wiped out, removing all of the "additional commits".

For any user that have a local version of the commit history from before the "clean up", a `git fetch` will show that the local branch is both "ahead of" and "behind" the `remote`. In order to resolve this, the user can run `git rebase <REMOTE>/<BRANCH>` to rebase the new version of `<REMOTE>/<BRANCH>` into their current branch. Depending on the changes made during the interactive rebase, some merge conflicts may occur.

If the user experiences merge conflicts, and is looking for their branch to be "in-line" with what exists on GitHub, then the user can easily resolve this with the command `git checkout --ours .`. This is because `<REMOTE>/<BRANCH>` is checked out during the rebase, and the commits from `<BRANCH>` are being played back on top of the tip of `<REMOTE>/<BRANCH>`. Therefore, `<REMOTE>/<BRANCH>` contains the "ours" changes, and `<BRANCH>` contains the "theirs" changes. For more information, refer to the section on [resolving merge conflicts](../git/resolving-merge-conflicts.md), specifically the section on ["ours" and "theirs"](../git/resolving-merge-conflicts.md#The-concept-of-ours-and-theirs).
