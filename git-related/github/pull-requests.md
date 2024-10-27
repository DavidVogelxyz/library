# Pull requests (PRs)

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#Introduction)
- [What's in a PR?](#What's-in-a-PR)
- [Creating a PR](#Creating-a-PR)
    - [Creating a fork](#Creating-a-fork)
    - [Preparing commits to be submitted as a PR](#Preparing-commits-to-be-submitted-as-a-PR)
    - [Submitting a PR for review](#Submitting-a-PR-for-review)
- [Finalizing a PR](#Finalizing-a-PR)
    - [Limiting merge commits](#Limiting-merge-commits)
    - [Preserving GPG signatures](#Preserving-GPG-signatures)
    - [How to both limit merge commits and preserve GPG signatures](#How-to-both-limit-merge-commits-and-preserve-GPG-signatures)
- [Making changes during the PR review process](#Making-changes-during-the-PR-review-process)
- [References](#References)

## Introduction

Pull requests (PRs) are an important utility for a GitHub user, and they are one of the few features that don't exist in Git. PRs allow a user to submit changes to a repo they don't own or control.

The process for creating a PR is relatively simple. First, the user forks the GitHub repo to which they want to submit changes. Then, the user clones their fork and, optionally, creates a different branch in order to develop the PR. Once the commits have been made (and hopefully, squashed), the user pushes the changes to their GitHub fork. From there, the user can sign into GitHub and submit a PR.

The PR will then go through a review process. The process is different for every repo, and depends on what the repo's owners have set as the standard. The PR's reviewers can accept the changes, deny them, or request additional changes before approving. However, once a sufficient number of reviewers have approved the changes, the PR can be merged into the branch for which it was submitted.

This section of the guide will highlight some of these steps, in order to provide clarity and additional details.

## What's in a PR?

Most people believe that a PR is a commit being submitted to another branch with suggested changes. While this is partially true, it doesn't capture the complete picture.

This is most easily demonstrated with a PR that includes multiple commits. Most people would claim that the PR is the collection of those commits. Again, this is partially true, but not completely.

A PR is actually the entire branch that's being submitted. This is an important distinction, because it will explain a few peculiarities about how GitHub handles certain manipulations of a PR during the review process.

## Creating a PR

The first step to creating and submitting a PR is to create a fork of the repo and clone it.

### Creating a fork

To create a fork, log into GitHub, navigate to the repo, and select the "Fork" option. This creates a repo in the user's list of repos that is a copy of the repo at the time of forking. From here, the user can clone their fork onto their local machine and beging working.

Under the hood, a fork is simply a repo that is configured with multiple `remotes`. Using GitHub's vocabulary, the fork has both an `origin` and an `upstream`. This diverges from a user's standard repo, which only has an `origin` configured as a `remote`. For a fork, the `origin` is still the version of the repo found under the user's account. However, it also has an `upstream`, which is the original repo from which the fork came. For more information, refer to the section on [working with remote repositories in Git](../git/git-remote.md).

In theory, it *is* possible to clone the original repo, change the `origin` of that repo to be named `upstream`, and then configure a new `remote` named `origin` that points to the user's version. The user would still need to create a new, empty repo on their account in order to push to GitHub.

However, the reason this doesn't work is because GitHub needs to know that the repo is a fork of another. By creating a fork in this way, GitHub doesn't know the association between the fork and the original repo. Therefore, the best way to create a fork is through GitHub.

### Preparing commits to be submitted as a PR

Once the fork has been created and cloned, the process is relatively straightforward.

The user can make changes directly to the branch to which they want to submit the PR. However, in many cases, the user will opt to create a development branch to create the commits to be submitted.

One tactic that's worth mentioning is the idea of "committing small changes often". This allows the user to save their work and prevent loss. However, when it comes time to push the branch back to GitHub and submit the PR, it is advisable to squash those commits into a single commit. This makes the reviewer's lives easier, as they won't have to sift through a bunch of smaller commits, and can more easily see what's changed between their repo and the PR. For more information, refer to the section on [squashing multiple commits](../git/interactive-rebase.md#Squashing-multiple-commits-into-one-commit).

Whether or not the commits are squashed, they will then be pushed to GitHub. At this point, the changes are ready to be submitted as a PR.

### Submitting a PR for review

Once the changes have been committed and pushed to GitHub, the user will navigate to their fork. A button will appear at the top of the page that directs the user to submit their change via a PR. Selecting this button will take the user to a page where they can customize a title and message for the PR. By default, the title of the PR will be the commit's title, and the message will be the contents of the commit message.

When ready, the user will select "Submit", and the PR will be availble for review!

## Finalizing a PR

Once the PR has been reviewed and accepted, the user has the ability to merge in their PR. There are a few different ways the user can accomplish this:

- The PR can be merged into the branch.
    - If the user selects the "merge" option, then GitHub will attempt to "fast-forward merge" the commit. If successful, the commit will be added to the tip of the branch, without a merge commit.
    - However, if the "fast-forward merge" is unsuccessful, then GitHub will create a merge commit.
    - For more information on merging, refer to the section on [git-merge](../git/git-merge.md).
- The PR can be rebased into the branch.
    - If the user selects the "rebase" option, then GitHub will rebase the branch underneath the commit, checking out the branch and then playing back the commit on top of it.
    - For more information on rebasing, refer to the section on [git-rebase](../git/git-rebase.md).

Especially for users who want to limit the number of merge commits made, and care to preserve their GPG signatures, it is important to know which option to use at which time.

### Limiting merge commits

Simply put, merge commits can only occur when the following states are all true:

1. The "merge" option is chosen.
2. It isn't possible for GitHub to perform a "fast forward merge".

When all of the above states are true, GitHub will be forced to merge through the use of a merge commit.

If a user wants to limit merge commits, they can choose to do either of the following:

1. Confirm that the branch being merged in is current with the branch being merged into.
2. Choose the "rebase" option.

However, these options have particularities that should be considered, especially when the user wants to preserve their GPG signature.

### Preserving GPG signatures

When choosing the "merge" option, GPG signatures on the commit being merged in are **always** preserved.

If the commit is merged in with a "fast-forward merge", then the commit is appended to the tip of the branch, and will be merged into the target branch with the same hash (and signature) as on the branch being merged in.

If the commit is merged in using a merge commit, then the commit will also retain its hash (and signature), and the merge commit will be signed by GitHub.

When choosing the "rebase" option, the GPG signature is **almost always** removed. This is because GitHub performs the rebase, which changes the commit's hash and requires a new GPG signature. However, GitHub doesn't have access to the GPG private key that was used to sign the commit, so it's not able to reapply the signature. Thus, the net effect is that the signature is removed.

### How to both limit merge commits and preserve GPG signatures

The key to limiting merge commits ***and*** preserving GPG signatures is to perform `git rebase <TARGET_BRANCH>` with the submitted branch checked out, *before* performing a merge. Then, the user can use a "fast forward merge" to merge their PR without a merge commit, while retaining their GPG signature.

When a PR is under review, and the submitted branch falls behind the target branch, a user may consider running `git rebase <TARGET_BRANCH>` to check out the target branch, and reapply their submitted commits on top of the new tip of the branch. However, the user may stop themself, especially if the PR has already received some approvals. It's important to know that, unless the contents of the commits have changed, rebasing to the tip of the target branch ***will not*** affect approvals.

Therefore, a user can wait until after their submitted branch has received a sufficient number of approvals to rebase. Then, *before* merging the PR into the target branch, the user can run `git rebase <TARGET_BRANCH>`.

At this point, if the user runs `git rebase <TARGET_BRANCH>` while their submitted branch is checked out, the current tip of the target branch will be checked out and the PR's commits will be played back on top. Assuming that there are no merge conflicts, the submitted branch will be up-to-date with the target branch, and the PR will not lose its approvals. If there are merge conflicts, then the user must accept the changes from the target branch ("ours"); otherwise, divergence will occur, and the submitted branch *will* lose its approvals. For more information, refer to the section on ["ours" and "theirs"](../git/resolving-merge-conflicts.md#The-concept-of-ours-and-theirs).

Obviously, if the submitted branch is changed locally, it must be force pushed onto the `remote` (GitHub) for the changes to show up. However, as already stated, the force push will **not** affect approvals, so long as the content of the PR's commits haven't changed.

## Making changes during the PR review process

During the PR review process, a user may be requested to make supplemental changes to their PR before it can be approved. Some "best practices" are detailed below.

As already mentioned in the section on [limiting merge commits and preserving GPG signatures](#How-to-both-limit-merge-commits-and-preserve-GPG-signatures), a user can make changes to their submitted branch and then force push those changes. So, the user can easily go back to the local version of their submitted branch, create a new commit to address the requested changes, and then push them to GitHub.

It can also be a good idea to create the new commit, and then squash it into the originally submitted commit. Since both are going to change the contents of the PR, both will dismiss (reset) the current approvals. However, some teams may want the additional changes in a separate commit, in order to more easily identify what changes were made against the original submission. However, every new commit will be added to the repo's commit history upon merging, so use discretion when deciding whether or not to squash the new commit. For more information, refer to the section on [squashing multiple commits](../git/interactive-rebase.md#Squashing-multiple-commits-into-one-commit).

## References

- [Medium - GitHub -> Forks and Pull Requests](https://medium.com/swlh/forks-and-pull-requests-how-to-contribute-to-github-repos-8843fac34ce8)
    - Reference for creating forks and submitting pull requests
- [StackOverflow - How to do a GitHub pull request](https://stackoverflow.com/questions/14680711/how-to-do-a-github-pull-request)
    - Reference for submitting pull requests
- [GitHub - Reviewing proposed changes in a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/reviewing-proposed-changes-in-a-pull-request)
    - Reference for reviewing changes
