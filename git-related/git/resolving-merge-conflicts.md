# Resolving Merge Conflicts

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Conflicts during a merge](#conflicts-during-a-merge)
    - [Resolving a conflict when merging](#resolving-a-conflict-when-merging)
        - [Rejecting the "incoming change" when merging](#rejecting-the-incoming-change-when-merging)
        - [Accepting the "incoming change" when merging](#accepting-the-incoming-change-when-merging)
- [Conflicts during a rebase](#conflicts-during-a-rebase)
    - [Resolving a conflict when rebasing](#resolving-a-conflict-when-rebasing)
        - [Rejecting the "incoming change" when rebasing](#rejecting-the-incoming-change-when-rebasing)
        - [Accepting the "incoming change" when rebasing](#accepting-the-incoming-change-when-rebasing)
- [Using "git-rerere" to reuse recorded resolutions](#using-git-rerere-to-reuse-recorded-resolutions)
- [The concept of "ours and theirs"](#the-concept-of-ours-and-theirs)
- [References](#references)

## Introduction

Whether it be through merging two branches together, rebasing a branch, or the use of interactive rebases, every user will eventually run into merge conflicts. Understanding what they mean, and how to resolve them, is a crucial skill to anyone working with Git.

Merge conflicts occur when there are changes in both the "local" branch and the branch being brought in that cannot be resolved. Put another way, when merging or rebasing, if both versions have edits on the same line, then Git isn't going to automatically know which state to pick. Thus, there is a conflict.

Note that conflicts can happen when merging or rebasing branches, as well as when using `git pull` to merge or rebase with a `remote`.

## Conflicts during a merge

When a conflict occurs during a merge, the user will be greeted with a message, such as the following:

```
Auto-merging <FILE>
CONFLICT (content): Merge conflict in <FILE>
Automatic merge failed; fix conflicts and then commit the result.
```

By running `git status`, the user can more easily see what files are in conflict:

```
On branch <CURRENT_BRANCH>
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   <FILE>

no changes added to commit (use "git add" and/or "git commit -a")
```

Note that it's possible to abort the merge and go back to the previous state by running the following command:

```
git merge --abort
```

### Resolving a conflict when merging

Imagine a situation where a file `newfile` on branch `prod` has the following contents:

```
a
```

A user creates branch `dev`, and commits the following changes:

```
2
```

However, the branch `prod` updates `newfile` in a subsequent commit to contain:

```
1
```

The user attempts to merge `dev` into `prod` by switching branches to `prod` and running `git merge dev`. However, they discover that there is a conflict in `newfile`. The user runs `cat newfile` to see the following:

```
<<<<<<< HEAD
1
=======
2
>>>>>>> dev
```

First, the user will notice that the sections marked `<<<<<<<, =======, >>>>>>>` show the location of the conflict. The content between `<<<<<<< HEAD` and `=======` show the changes present on the `HEAD` of the current branch (`prod`), and everything between `=======` and `>>>>>>> dev` show the changes present in branch `dev` (or whatever is the "incoming change").

Note the following:

- When using `git merge`, the tag for the incoming change is `<BRANCH>`.
- When using `git pull --ff`, the tag will be the commit hash.
- However, in both cases, `HEAD` will always be the current branch (in this case, `prod`).

In either case, the user will choose which changes to keep. Then, the user will add the file to the index with `git add <FILE>`, and commit the changes with `git commit`.

#### Rejecting the "incoming change" when merging

To choose the change made in branch `prod` (`HEAD`), remove the following lines:

```
<<<<<<< HEAD
=======
2
>>>>>>> dev
```

This would leave the file with the following contents:

```
1
```

The user may want to run `git status` between `git add` and `git commit`, to verify the state of the branch before committing. If the user chose to keep the changes from `prod` (`HEAD`), and rejected the "incoming changes" from `dev`, doing so would display the following message:

```
On branch <CURRENT_BRANCH>
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)
```

This is an indication that the "incoming changes" were rejected, and the "change" is essentially empty. Because of the way that `git merge` works, the history of the change will not be lost, but the change will not be accepted. Therefore, a merge commit is still needed, and a `git log` after a successful merge commit will show this.

It is important to note that, if a merge conflict is resolved without accepting the changes in `upstream`, then merge commits will continue to be needed until the local changes are pushed to `upstream`.

#### Accepting the "incoming change" when merging

To choose the change made in branch `dev`, remove the following lines:

```
<<<<<<< HEAD
1
=======
>>>>>>> dev
```

This would leave the file with the following contents:

```
2
```

The user may want to run `git status` between `git add` and `git commit`, to verify the state of the branch before committing. If the user chose to accept the "incoming changes" from `dev`, doing so would display the following message:

```
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   newfile
```

This is an indication that the "incoming changes" were accepted. Since there is a change, as compared to the working tree of `prod`, then those changed need to be committed.

## Conflicts during a rebase

In contrast to a merge conflict, a conflict that occurs when rebasing can present with more challenges.

This is because of the way that `git rebase` works. Rebasing takes the tip of the branch passed as an argument, and then replays all the commits that diverged onto that branch. If a conflict happens in the past, then the conflict may need to be resolved multiple times.

Imagine the following setup:

```
  C --- D   dev
 /
A --- B     prod
```

The user wants to rebase `dev` with `prod`. In doing so, the user discovers a conflict between `B` and `D`. The user resolves the conflict, and successfully rebases `dev`. The setup now looks like the following:

```
        C --- D     dev
       /
A --- B             prod
```

However, a new commit (`X`) is pushed to `prod`. The state of the branches is now:

```
        C --- D     dev
       /
A --- B --- X       prod
```

The user again wants to rebase `dev` with `prod`, to get to the following state:

```
              C --- D   dev
             /
A --- B --- X           prod
```

When replaying `C` and `D` on top of `X`, the user will have to resolve the same conflict that was resolved during the first rebase!

When a conflict occurs during a rebase, the user will be greeted with a message, such as the following:

```
CONFLICT (content): Merge conflict in <FILE_1>
Auto-merging <FILE_2>
error: could not apply <COMMIT_HASH_OF_INCOMING_CHANGE>... <COMMIT_TITLE_OF_INCOMING_CHANGE>
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply <COMMIT_HASH_OF_INCOMING_CHANGE>... <COMMIT_TITLE_OF_INCOMING_CHANGE>
```

In this case, the error message explains that there were two files with changes: `<FILE_1>` and `<FILE_2>`. The changes from `<FILE_2>` were not in conflict, so they were automatically merged. However, the changes from `<FILE_1>` did experience a conflict.

By running `git status`, the user can more easily see what files are in conflict:

```
interactive rebase in progress; onto <COMMIT_HASH_OF_CURRENT_BRANCH>
Last command done (1 command done):
   pick <COMMIT_HASH_OF_INCOMING_CHANGE> <COMMIT_TITLE_OF_INCOMING_CHANGE>
No commands remaining.
You are currently rebasing branch '<BRANCH>' on '<COMMIT_HASH_OF_CURRENT_BRANCH>'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   <FILE_2>

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
        both modified:   <FILE_1>
```

As shown by `git status`, `<FILE_2>` is ready to be committed, because it contained no conflicts. However, `<FILE_1>` is "unmerged" because both commits have modified that file.

Note that it's possible to abort the rebase and go back to the previous state by running the following command:

```
git rebase --abort
```

### Resolving a conflict when rebasing

Imagine a situation where a file `newfile` on branch `prod` has the following contents:

```
a
```

A user creates branch `dev`, and commits the following change to the file:

```
2
```

However, the branch `prod` commits `newfile` a different change to that file:

```
1
```

The user attempts to rebase `dev` with `prod` by switching branches to `dev` and running `git rebase prod`. However, they discover that there is a conflict in `newfile`. The user runs `cat newfile` to see the following:

```
<<<<<<< HEAD
1
=======
2
>>>>>>> <COMMIT_HASH_OF_INCOMING_CHANGE> (<COMMIT_TITLE_OF_INCOMING_CHANGE>)
```

At this point, it's worth noting that the headers for `HEAD` and the "incoming changes" are in the same place, with `HEAD` on top and the "incoming changes" on bottom.

While the contents of `HEAD` (`1`) and the "incoming change" (`2`) are also in the same place as during the merge example, an astute observer will note that, in this example, `HEAD` is not the current branch:

- Even though `dev` was the current branch, it is seen as the "incoming change".
- `prod` is considered `HEAD`, even though it was not the current branch.

This should make sense with an understanding of how `git merge` and `git rebase` work:

- In the merge example of `git merge dev`, `prod` was the current branch, and the changes from `dev` were being merged into `prod`.
    - Therefore, `dev` would be the "incoming changes" to `prod`.
- In the rebase example of `git rebase prod`, `dev` was the current branch, and the changes from `prod` were being rebased underneath `dev`.
    - However, `git rebase` works by checking out the tip of `prod`, and then playing back the commits from `dev` on top of `prod`.
    - Therefore, `dev` would be again be the "incoming changes" to `prod`.

Put another way:

- When using `git merge <BRANCH>`, the current branch is `HEAD`, and `<BRANCH>` is the "incoming change".
- When using `git rebase <BRANCH>`, `<BRANCH>` is `HEAD`, and the current branch is the "incoming change".

This is the concept of "ours and theirs", and is explained more in the section on [the concept of "ours and theirs"](#the-concept-of-ours-and-theirs).

As with a merge conflict, the user will choose which changes to keep and which to discard. Once the changes have been chosen, add the file with `git add <FILE>` and then continue the rebase with `git rebase --continue`.

If there are no other conflicts, the rebase will complete successfully. If the user runs `git log` once the rebase completes, they will see no merge commits. As discussed in the section on ["git-rebase"](git-rebase.md#introduction), this is to be expected, as `git rebase` does not create merge commits.

#### Rejecting the "incoming change" when rebasing

To accept the change made in branch `prod` (`HEAD`), remove the following lines:

```
<<<<<<< HEAD
=======
2
>>>>>>> <COMMIT_HASH_OF_INCOMING_CHANGE> (<COMMIT_TITLE_OF_INCOMING_CHANGE>)
```

This would leave the file with the following contents:

```
1
```

The user may want to run `git status` between `git add` and `git commit`, to verify the state of the branch before committing. If the user accepts from `prod` (`HEAD`), and rejects the "incoming changes" from `dev`, doing so would display the following message:

```
interactive rebase in progress; onto <COMMIT_HASH_OF_CURRENT_BRANCH>
Last command done (1 command done):
   pick <COMMIT_HASH_OF_INCOMING_CHANGE> <COMMIT_TITLE_OF_INCOMING_CHANGE>
No commands remaining.
You are currently rebasing branch 'dev' on '<COMMIT_HASH_OF_CURRENT_BRANCH>'.
  (all conflicts fixed: run "git rebase --continue")
```

This is an indication that the "incoming changes" were rejected, and the "change" is essentially empty. The branch `prod` is checked out, so rejecting the changes from `dev` would result in no changes to `newfile`.

#### Accepting the "incoming change" when rebasing

To accept the change made in branch `dev` (`<COMMIT_HASH_OF_INCOMING_CHANGE>`), remove the following lines:

```
<<<<<<< HEAD
1
=======
>>>>>>> <COMMIT_HASH_OF_INCOMING_CHANGE> (<COMMIT_TITLE_OF_INCOMING_CHANGE>)
```

This would leave the file with the following contents:

```
2
```

The user may want to run `git status` between `git add` and `git commit`, to verify the state of the branch before committing. If the user chose to accept the "incoming changes" from `dev`, doing so would display the following message:

```
interactive rebase in progress; onto <COMMIT_HASH_OF_CURRENT_BRANCH>
Last command done (1 command done):
   pick <COMMIT_HASH_OF_INCOMING_CHANGE> <COMMIT_TITLE_OF_INCOMING_CHANGE>
No commands remaining.
You are currently rebasing branch 'dev' on '<COMMIT_HASH_OF_CURRENT_BRANCH>'.
  (all conflicts fixed: run "git rebase --continue")

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   newfile
```

This is an indication that the "incoming changes" were accepted. Since there is a change, as compared to the working tree of `prod`, then those changed need to be committed.

As mentioned in the section on [conflicts during a rebase](#conflicts-during-a-rebase), if the "incoming change" is accepted during a rebase, then it's almost a guarantee that any other commits that are played back during the rease process will also encounter the same conflict. This is due to the nature of how `git rebase` works -- by accepting the "incoming change", then other commits played back during the rebase will continue to reconflict, since the conflict exists in those other commits.

## Using "git-rerere" to reuse recorded resolutions

[The Git documentation](https://git-scm.com/book/en/v2/Git-Tools-Rerere) explains the following about `git rerere`:

"The `git rerere` functionality is a bit of a hidden feature. The name stands for “reuse recorded resolution” and, as the name implies, it allows you to ask Git to remember how you’ve resolved a hunk conflict so that the next time it sees the same conflict, Git can resolve it for you automatically."

This configuration can be set with the following command:

```
git config rerere.enabled true
```

`git rerere` can be a double-edged sword. In the case of a `git rebase` where a bunch of commits will continue to conflict because an "incoming change" was accepted, it can be a great benefit. But, consider a situation where a bad resolution is recorded. In this case, it may be unfortunate that Git remembers the resolution. Of course, `git rerere` can be disabled, and there are also commands to delete a "recorded resolution".

## The concept of "ours and theirs"

Sometimes, when resolving conflicts, the user may want to accept all the changes from one side. In these situations, an understanding of the concept of "ours and theirs" can save the user a lot of headache.

Whether merging or rebasing, "ours" is always the change from the current branch (`HEAD`), and "theirs" is always the incoming change. As was seen in the previous examples, "ours" are always the changes on top, and "theirs" are always the changes on the bottom. When "ours" (`HEAD`) is the accepted change, staging the file with `git add` will result in the change disappearing from `git status`. When "theirs" is the accepted change, staging the file will show a change when running `git status`.

However, as was also seen in the previous examples, what is "ours" and what is "theirs" changes when using `git merge` and `git rebase`.

Consider the difference between the examples:

- When merging `dev` into `prod`, `git merge dev` was run on the current branch `prod`.
    - Therefore, `prod` was "ours", and `dev` was "theirs".
- When rebasing `prod` onto `dev`, `git rebase prod` was run on the current branch `dev`.
    - However, `git rebase` works by checking out the tip of `prod`, and then playing back the commits from `dev` on top of `prod`.
    - Therefore, `prod` was again "ours", and `dev` was "theirs".

Put another way:

- When using `git merge <BRANCH>`, the current branch is "ours", and `<BRANCH>` is "theirs".
- When using `git rebase <BRANCH>`, `<BRANCH>` is "ours", and the current branch is "theirs".

When resolving a conflict, it is possible to choose the changes from "ours" by running the following command:

```
git checkout --ours <PATH>
```

Or, to accept the changes from "theirs":

```
git checkout --theirs <PATH>
```

Note that `<PATH>` can be anything from a "file in conflict" to `.` (the entire current working directory).

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Rebasing is Complicated](https://theprimeagen.github.io/fem-git/lessons/going-remote/more-rebasing-and-merging)
    - A deep dive into resolving merge conflicts, both when merging and rebasing
