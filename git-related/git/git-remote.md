# Working with Remote Repositories in Git

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Remote doesn't have to be remote?](#remote-doesn't-have-to-be-remote)
    - [A note on the definition of "remote"](#a-note-on-the-definition-of-remote)
- [Using "git-clone" to clone the current state of a remote repo](#using-git-clone-to-clone-the-current-state-of-a-remote-repo)
- [Using "git-fetch" to grab the current state of a remote repo](#using-git-fetch-to-grab-the-current-state-of-a-remote-repo)
- [Using "git-merge" to update the current state of a local branch](#using-git-merge-to-update-the-current-state-of-a-local-branch)
- [Using "git-rebase" to update the current state of a local branch](#using-git-rebase-to-update-the-current-state-of-a-local-branch)
- [Using "git-pull" to update the current state of a local branch](#using-git-pull-to-update-the-current-state-of-a-local-branch)
    - [A note about using "git-pull" with rebase](#a-note-about-using-git-pull-with-rebase)
- [Using "git-push" to update a remote branch](#using-git-push-to-update-a-remote-branch)
    - [A note about errors when using "git-push"](#a-note-about-errors-when-using-git-push)
- [More information about tracking](#more-information-about-tracking)
- [Creating a new GitHub repo with an already existing local repo](#creating-a-new-github-repo-with-an-already-existing-local-repo)
- [Syncing a local copy of a fork with the upstream project](#syncing-a-local-copy-of-a-fork-with-the-upstream-project)
    - [A follow up to working with a fork](#a-follow-up-to-working-with-a-fork)
- [Pushing to a branch that is currently checked out on the remote](#pushing-to-a-branch-that-is-currently-checked-out-on-the-remote)
- [Deleting a remote branch](#deleting-a-remote-branch)
- [References](#references)

## Introduction

A "remote repository" is, very simply, a copy of the repo that is *somewhere else*.

The most common example of a "remote repo" is a user's Git repo on GitHub or GitLab, though it doesn't have to be. In fact, the remote repo doesn't have to be online, nor does it need to be on another computer. The only requirement is that the remote repo exists at a path that is different that the "local" version.

## Remote doesn't have to be remote?

Consider the following example: a user has a Git project in their home directory, located at the path `~/original-repo`. The user then creates an empty directory in their home directory, and calls it `~/new-repo`. This user then runs `cd new-repo` and `git init`, initializing the new directory as a Git repo. The user can now associate `original-repo` and `new-repo` together, and it would work the exact same way as if the user had used their GitHub account to create a copy of the repo online.

Of course, there's a bit more to it than that. How would Git know that `original-repo` and `new-repo` are associated?

The user would change directory into `~/new-repo`, and run the following command:

```
git remote add <REMOTE> <PATH>
```

In this case, the user would run `git remote add origin ~/original-repo`, or `git remote add origin ../original-repo`. A configuration would be added to the repo's local config, which is located in the `.git/config` file, telling Git that the `remote` repo named `origin` can be found at the path `~/original-repo`, or at the path `../original-repo`. Either path would get Git correctly from the current repo to `remote`. For more information on the `.git/config` file, check out the section on ["git-config"](git-config.md#example-local-config-file).

This is exactly the same command that is used when an already existing Git project is added to GitHub or GitLab. Those websites direct the user to run this command to add the same config to the project -- however, in the case of GitHub and GitLab, the path is a "URI/URL" pointing to the repo on their website. An example of this would be `git remote add origin https://github.com/DavidVogelxyz/library` -- this version of the command would set the `origin` remote to the path where this Git repo exists on GitHub's website.

To list out all the `remote` repos that are associated with the current project, run the following command:

```
git remote -v
```

### A note on the definition of "remote"

The word "remote" is sometimes used with different meanings.

In the case of a Git project that exists on GitHub, and there's a local copy on the user's machine, `remote` refers to the version that's on GitHub. This `remote` is often called `origin`, and it is usually considered the *source of truth* version of the repo.

But, what if a user forks a GitHub repo, and then clones the fork onto their local machine? Then, the user's fork on GitHub will be called `origin`, and the original project repo will be called `upstream`. In this case, the fork (`origin`) is often called `remote`.

Note that `origin` and `upstream` are just naming conventions -- realistically, a user could choose any `<REMOTE>` when creating these configs. However, `origin` and `upstream` are the standard naming conventions.

## Using "git-clone" to clone the current state of a remote repo

`git clone` is a very simple command that allows the user to take the current state of a "remote" repo and initialize it "locally". The syntax for `git clone` is as follows:

```
git clone <URI> <PATH_TO_LOCAL_REPO>
```

As is discussed in the section on ["git-config"](git-config.md#example-local-config-file), the `<URI>` can be any of the following:

- To clone a repo from:
    - GitHub/GitLab, using HTTPS: `https://github.com/<USER>/<NAME_OF_REPO>`
    - GitHub/GitLab, using `ssh`: `git@github.com:<USER>/<NAME_OF_REPO>`
    - A local directory: `../<NAME_OF_OTHER_REPO>`
    - A remote directory, in `ssh`/`scp` format: `<USER>@<DOMAIN>:/<PATH_TO_REPO>`

Often, when cloning from GitHub or GitLab, the `<NAME_OF_REPO>` will have `.git` appended to it. This is actually a reference to the fact that the repo is initialized on GitHub and GitLab as a bare repo. While the directory's name on the server *is* `<NAME_OF_REPO>.git`, it is not necessary to include the `.git` when specifying the `<URI>` for the repo. For more information on bare repos, refer to the section on ["git-init"](git-init.md#bare-repositories).

`<PATH_TO_LOCAL_REPO>` allows the user to path somewhere else, or give the repo a different name on the local machine. This is also not necessary. If the `git clone` command is run without `<PATH_TO_LOCAL_REPO>`, then `git clone` will clone the repo into the current working directory, with the same name as the `remote`. Passing `<PATH_TO_LOCAL_REPO>` is most often used when the local path is not going to be within the current working directory, or when the user wants to change the name of the directory being created locally.

It is also possible to run `git clone` with the `--bare` switch, thus making the clone a bare repo. For more information on bare repos, refer to the section on ["git-init"](git-init.md#bare-repositories).

## Using "git-fetch" to grab the current state of a remote repo

The command `git fetch` can be used to grab information about the state of Git from the remote repo. This is often used in the context of fetching commits from a GitHub or GitLab repo. But, as discussed in the example from earlier, a user can run `git fetch` from within `~/new-repo`, and it will fetch the state of `original-repo`. Note that `git fetch` does not update or change the state of `new-repo`, but solely grabs the information about the current state of `original-repo`.

If the user was to run `git log` at this point, the command would return with the error: `fatal: your current branch '<BRANCH>' does not have any commits yet`. This is a confirmation that `git fetch` did not change the state of `new-repo`, but only grabbed the information about `original-repo`. While the branch `origin/<BRANCH>` has been updated, branch `<BRANCH>` has not.

This is important to note: if a branch is from a remote, it will contain the name of the remote it came from.

Another important note: if no remote is specified, `git fetch` will use the `origin` remote by default, unless there is an `upstream` remote set **for the current branch**.

As is described in the section on ["git-branch"](git-branch.md#listing-branches), to view a list of all the branches for which `git fetch` has grabbed information, run the following command:

```
git branch -a
```

This command will return a list of all branches associated with the repo, including those that exist only remotely.

## Using "git-merge" to update the current state of a local branch

Now that `git fetch` has grabbed all the information about the state of the remote branch, how does a user update their local branch?

As was discussed in the section on ["git-merge"](git-merge.md#how-to-perform-a-fast-forward-merge), the command `git merge` can be used to update the point for the current branch to point to the same commit as the remote branch. Run the following command:

```
git merge origin/<BRANCH>
```

While it may appear that nothing happened, running `git log` will reveal that the tip of `<BRANCH>` now points to the same commit as `origin/<BRANCH>`.

## Using "git-rebase" to update the current state of a local branch

Imagine a slightly different example: a user has made some commits on their local branch, and now they want to use `git fetch` and `git merge` in order to update their current branch. This user is going to run into a problem: the new commits on `origin` don't have the user's new commits in their history.

When the history diverges, what can the user do? As was discussed in the section on ["git-rebase"](git-rebase.md#how-to-perform-a-rebase), they can run `git rebase`!

```
git rebase origin/<BRANCH>
```

By rebasing, Git checks out the latest commit on `origin/<BRANCH>`, and then plays all the new commits from `<BRANCH>` on top. As with `git merge`, it may not appear as if anything happened, but running `git log` will reveal that the base of `<BRANCH>` has been updated to the tip of `origin/<BRANCH>`.

## Using "git-pull" to update the current state of a local branch

The command `git pull` is essentially a combination of `git fetch` and `git merge origin/<BRANCH>` -- the command will grab all of the branch information from the `origin` remote, and then will *pull* the changes into the current branch.

Using the previous example, imagine that there is a commit in `original-repo` that the user wants to pull into `new-repo`. If the user runs `git pull`, as everything stands now, it will actually fail with the error: `There is no tracking information for the current branch`. This actually makes sense -- Git doesn't know which branch to merge into the current branch. It may seem dumb, but Git is smart enough to refuse to assume that two branches with the same name *are* the same. Git is also careful in this instance because the user may have set more than one remote. In that case, the user definitely wants to specify which remote to be pulling from!

There are two ways to navigate the situation.

One method is to specify which branch to pull from, as is shown below:

```
git pull <REMOTE> <BRANCH>
```

So, the user could run `git pull origin <BRANCH>`, and Git would then pull `origin/<BRANCH>` and merge it into the current branch. In this case, the current branch should be `<BRANCH>`. However, these specifications would have to be entered manually, every time a `git pull` or a `git push` is run.

Another method is to set the tracking information, such that Git can automatically and conveniently perform pulls and pushes based on those configs. To do this, the user would run the following:

```
git branch --set-upstream-to=origin/<BRANCH> <BRANCH>
```

The user can also run the `git pull` command with the `-u` switch (short for `--set-upstream`), and it will both perform the pull *and* set the upstream with the information provided, all in one command. The full syntax for that command follows:

```
git pull -u <REMOTE> <BRANCH>
```

As was the case when setting a path to a remote repo, this configuration will also be added to the local Git config, located at the path `.git/config`. Now, every time that the user wants to use `git pull` on branch `<BRANCH>`, Git will know to use `origin/<BRANCH>` as the remote. For more information on the `.git/config` file, check out the section on ["git-config"](git-config.md#example-local-config-file).

Additional notes:

- It is possible to run `git pull` on a branch with a different name than the one receiving the changes. This is achieved by using the syntax `git pull <REMOTE> <SOURCE_BRANCH>:<DESTINATION_BRANCH>`. In this case, the source branch might be `origin/prod` and the destination branch might be `dev`. So, the user could run `git pull origin prod:dev` in order to pull the changes on `origin/prod` into `dev`.
- As as offshoot of this, running the command `git pull <REMOTE> :<DESTINATION_BRANCH>` will delete the local branch `<DESTINATION_BRANCH>`. This is because the user is pulling "nothing" into the `<DESTINATION_BRANCH>`.

### A note about using "git-pull" with rebase

There are many reasons to use `git pull` in combination with rebasing:

1. The more merge commits exist in the history, the more difficult it is to perform `git revert`.
    - If the history of a repo is completely linear, the easier it is to go back through the history.
1. New commits can always be compared against the current state of the *source of truth* version of the repo.

There are two ways to achieve `git pull` with rebase.

One method is to run `git pull --rebase`.

The other method is to set "pulling with rebase" as a configuration. To do this, run the following command:

```
git config pull.rebase true
```

## Using "git-push" to update a remote branch

What is `git push`?

Simply put, `git push` is the opposite of `git pull`. Instead of bringing changes from a remote branch into a local copy, `git push` will take local changes and update a remote, such as `origin` (often, GitHub or GitLab).

Some switches that are often used with `git push` include `-f` and `-u`.

- `-f` is a switch that specifies that Git should "force push". This is useful in situations where a rebase may have caused the local history to differ from the remote history. Running `git push -f` allows the user to overwrite the state of the remote branch with the state of the local branch.
    - Note that this also works with `git pull` -- a `git pull -f` will overwrite any local changes, and bring the state of the local version in line with the remote. For many reasons, this is generally *not* something users want to do.
- `-u` is a switch that represents `--set-upstream`. As with `git pull`, a `git push -u <REMOTE> <BRANCH>` will perform a push *and* set the upstream with the information provided, all in one command.
    - Note that, if the `<REMOTE>` and `<BRANCH>` are not changing, then a `git push -u` only needs to be run once per association.

Additional notes:

- It is possible to run `git push` on a branch with a different name than the one receiving the changes. This is achieved by using the syntax `git push <REMOTE> <SOURCE_BRANCH>:<DESTINATION_BRANCH>`. In this case, the source branch might be `prod` and the destination branch might be `origin:dev`. So, the user could run `git push origin dev:prod` in order to push changes on `prod` into `origin:dev`.
- As as offshoot of this, running the command `git push <REMOTE> :<DESTINATION_BRANCH>` will delete the branch `<DESTINATION_BRANCH>` on `<REMOTE>`. This is because the user is pushing "nothing" to `<REMOTE>/<DESTINATION_BRANCH>`.

### A note about errors when using "git-push"

When pushing to a `remote` that isn't GitHub or GitLab, it's possible to encounter the error `refusing to update checked out branch`. Further down, the message will state `! [remote rejected] <BRANCH> -> <BRANCH> (branch is currently checked out)`. As the error implies, this is due to the fact that the branch that's being pushed to is the branch that's actively checked out on the `remote`.

This makes sense when thinking about the example of `original-repo` and `new-repo`. If the user was actively making changes on `original-repo/<BRANCH>`, and then decides to push changes in `new-repo/<BRANCH>` over to `original-repo/<BRANCH>`, the changes in `original-repo` would be overwritten. This can be dangerous and unintended, so Git, by default, prevents this type of behavior.

In order to resolve this error, `original-repo` would check out any branch that's not `<BRANCH>`. This would allow `new-repo` to push `<BRANCH>` to `original-repo/<BRANCH>`.

## More information about tracking

It can be useful to know that, when a local branch is tracking a single `remote`, and the user creates a new branch in the local repo that already exists on the `remote`, Git will automatically set up tracking for `<BRANCH>` to `<REMOTE>/<BRANCH>`. This can be demonstrated with the following example.

Imagine that the user from earlier in this section has branches `prod` and `dev` in `original-repo`, but only has branch `prod` in `new-repo`. The user doesn't have to create branch `dev` in `new-repo` -- if the user runs `git checkout dev`, they will switch to new branch `dev` with tracking already set to `origin/dev` (which is the `dev` from `original-repo`).

## Creating a new GitHub repo with an already existing local repo

Any user who has ever created a local repo (that eventually was pushed to GitHub/GitLab) has run these next few commands before. Hopefully, the commands now make sense on a more fundamental level.

Once it comes time for a local repo to make its way to GitHub/GitLab, the user will first create the repo on the website. Then, the user is directed to associate the local repo to the copy on the website.

First, the user is directed to add the `origin` remote to their repo with the following command:

```
git remote add origin <URI>
```

Next, GitHub/GitLab directs the user to rename the current branch to the same name as the branch that exists on the website by running the following:

```
git branch -m <BRANCH>
```

Last, GitHub/GitLab directs the user to push the state of the repo to the website with the following command:

```
git push -u origin <BRANCH>
```

Now, the local copy of the repo is associated with the GitHub/GitLab remote version. Anytime a user wants to push to GitHub/GitLab in the future, they need only run `git push`. In addition, `git pull` now has the correct tracking information to know how to pull changes from `origin/<BRANCH>` into the local branch `<BRANCH>`.

## Syncing a local copy of a fork with the upstream project

After having reviewed how to use `git merge` and `git fetch` to update a repo with remote information, the following should make sense.

On GitHub and GitLab, when a user forks a repo, there will be some UI features on the main page of the repo that allow the user to easily update the fork with the information from the upstream project. This is achieved by running Git operations, which means it's completely possible (and rather easy) to do the same in CLI using `git`.

For the sake of clarity, when a user does use the UI features on GitHub/GitLab, the project's `origin` (GitHub/GitLab) will be updated with information from the upstream project. So, the user would then go to their local project and perform one set of the following:

- Either, `git fetch`, and then either `git merge origin/<BRANCH>` or `git rebase origin/<BRANCH>`
- Or, either `git pull` (to merge) or `git pull --rebase` (to rebase)

These commands work because, when a user runs `git clone` on a GitHub/GitLab repo, the tracking information is already set in `.git/config` to point to `origin/<BRANCH>`. And, in this case, `origin/<BRANCH>` is the *fork's* version of the branches. However, if the user wants to grab the information from `upstream` themself, they have to specify from where to get this information.

Therefore, syncing a local copy of a fork with `upstream` is nearly identical. However, since the local copy of the fork is (likely) configured to have the `origin` remote as the upstream for the branches, the user must specify that they want to `git fetch` and `git rebase` using the `upstream` remote.

First, the user will checkout the branch they want to sync with the the upstream project:

```
git checkout <BRANCH>
```

Then, the user will run `git fetch <REMOTE>` to grab the information about the branches from `<REMOTE>`:

```
git fetch upstream
```

Next, the user will run either `git merge upstream/<BRANCH>` or `git rebase upstream/<BRANCH>` to bring those changes onto their local copy of `<BRANCH>`. Assuming that the user has some commits on their local branch, they would run the following:

```
git rebase upstream/<BRANCH>
```

At this point, the user's local copy of `<BRANCH>` point to the same commit as `upstream/<BRANCH>`. If the user needs to update a development branch with the new information, they can check out that branch and perform a rebase to update the base of that branch to the tip of `<BRANCH>`. At this point, this can be achieved by running `git checkout <DEV_BRANCH>` and then `git rebase <MASTER_BRANCH>`.

However, if the user checks their fork on GitHub/GitLab, nothing will have changed -- the `origin` never received these updates. So, the user would simply run `git push` to send those changes over to the `origin` remote.

### A follow up to working with a fork

Checking the `.git/config` of a local copy of a forked repo can be revealing.

A user who runs `git clone` on a GitHub fork is likely to see that both `origin` and `upstream` are defined within `.git/config`. `origin` will point to GitHub's version of the fork, and `upstream` will point to GitHub's version of the original project. However, all branches will be configured to point to `origin`, as opposed to `upstream`. This makes sense -- when a user runs `git push`, they almost certainly want to push to their fork, and not the `upstream`. Therefore, when it comes time to sync the fork to the `upstream`, then the `git fetch` command needs to specify to fetch the changes on the `upstream`.

## Pushing to a branch that is currently checked out on the remote

If a user ever attempts to push to a branch that is currently checked out on the `remote`, they will likely see a message very similar to the following:

```
remote: error: refusing to update checked out branch: refs/heads/<BRANCH>
remote: error: By default, updating the current branch in a non-bare repository
remote: is denied, because it will make the index and work tree inconsistent
remote: with what you pushed, and will require 'git reset --hard' to match
remote: the work tree to HEAD.
remote:
remote: You can set the 'receive.denyCurrentBranch' configuration variable
remote: to 'ignore' or 'warn' in the remote repository to allow pushing into
remote: its current branch; however, this is not recommended unless you
remote: arranged to update its work tree to match what you pushed in some
remote: other way.
remote:
remote: To squelch this message and still keep the default behaviour, set
remote: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
To <REPO>
 ! [remote rejected] <BRANCH> -> <BRANCH> (branch is currently checked out)
error: failed to push some refs to '<REPO>'
```

The key message is the second line from the bottom:

```
 ! [remote rejected] <BRANCH> -> <BRANCH> (branch is currently checked out)
```

By default, Git will not allow a user to push to a branch that is currently checked out. This is intentional, and makes sense -- if this was allowed, it would change the tip of the branch, and could cause changes that are being made to be lost.

There are two ways to solve this:

1. The user can checkout a different branch on the repo, which will allow the push to take place.
1. The user can set a configuration on the remote that allows for pushes to take place, even when the branch is currently checked out.

There are some cases where pushing to a branch that is currently checked out on the `remote` makes sense. Use the following shortlist to identify the right candidate for a `remote`:

When the `remote`:

- is a "non-bare" repo
- is not a `remote` where work is performed
- will often have the branch being pushed checked out as the current branch

When all of the above are true, it may make sense to use `git push` to push a branch that is currently checkout out on the `remote`.

As mentioned in the error message, the relevant config key is `receive.denyCurrentBranch`, and it should be **set on the remote** to `ignore` or `warn` in order to allow these pushes. As explained in the section on ["git-config"](git-config.md#other-configs), either set the value in the `.git/config` file (local) or the `~/.gitconfig` file (global). Alternatively, use `git config receive.denyCurrentBranch ignore` to achieve the same outcome. Note that, considering how this config works, it is more likely to be set at the local level than at the global level.

As mentioned, this config should only be used in certain circumstances. For a "Git server", or when using a `remote` as a backup, then the user would be advised to use "bare repos". For more information, refer to the section on [git-init](git-init.md#bare-repositories).

## Deleting a remote branch

It is possible to delete a remote branch directly from the command line -- this includes deleting a branch from GitHub.

A simple use case for this is the following: imagine a user who creates a fork on GitHub in order to submit a pull request. Once the pull request is approved, the user may no longer have a need for the development branch on their fork. This user may want to delete the branch from their fork, and can easily do this locally using `git branch -D <BRANCH>`. But, they can also delete the branch from GitHub, directly from the command line.

To do this, the user would run the following command:

```
git push <REMOTE> --delete <BRANCH>
```

As discussed in the section [using "git-push" to update a remote branch](#using-git-push-to-update-a-remote-branch), this command is equivalent to `git push <REMOTE> :<DESTINATION_BRANCH>`. However, `git push <REMOTE> --delete <BRANCH>` is much simpler to remember, and use correctly.

Note that `git push <REMOTE> --delete <BRANCH>` works with any version of Git from `v1.7.0` onward. For users running Git `v2.8.0` or newer, the following syntax will also work:

```
git push <REMOTE> -d <BRANCH>
```

To delete the local branch after deleting the remote branch, follow the steps in the section on [deleting branches](git-branch.md#deleting-branches).

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Remote Git](https://theprimeagen.github.io/fem-git/lessons/going-remote/remote-git)
    - Information on how remote repositories works
- [FreeCodeCamp - A Beginner's Guide to Git](https://www.freecodecamp.org/news/a-beginners-guide-to-git-how-to-create-your-first-github-project-c3ff53f56861/)
    - Reference for setting up a `git` repository with GitHub
    - Specifically, [this image](https://cdn-media-1.freecodecamp.org/images/cxRrZUe-tW2Wkn0WUg-MsN1m1WesvGPlJT7V) for the command: `git remote add origin <URI>`
- [Atlassian - Setting up a repository](https://www.atlassian.com/git/tutorials/setting-up-a-repository)
    - Another reference for setting up a `git` repository with GitHub
    - Again, for the command: `git remote add origin <URI>`
- [StackOverflow - Git - How do I update or sync a forked repository on GitHub?](https://stackoverflow.com/questions/7244321/how-do-i-update-or-sync-a-forked-repository-on-github)
    - An additional reference for syncing a forked repository
- [StackOverflow - How do I delete a Git branch locally and remotely?](https://stackoverflow.com/questions/2003505/how-do-i-delete-a-git-branch-locally-and-remotely/2003515#2003515)
    - A reference for deleting a Git branch from a `remote`
