# Resolving Merge Conflicts

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Conflicts during a merge](#Conflicts-during-a-merge)
    - [Resolving a conflict when merging](#Resolving-a-conflict-when-merging)
        - [Rejecting the "incoming change" when merging](#Rejecting-the-incoming-change-when-merging)
        - [Accepting the "incoming change" when merging](#Accepting-the-incoming-change-when-merging)
- [Conflicts during a rebase](#Conflicts-during-a-rebase)
    - [Resolving a conflict when rebasing](#Resolving-a-conflict-when-rebasing)
        - [Rejecting the "incoming change" when rebasing](#Rejecting-the-incoming-change-when-rebasing)
        - [Accepting the "incoming change" when rebasing](#Accepting-the-incoming-change-when-rebasing)
- [Using "git-rerere" to reuse recorded resolutions](#Using-git-rerere-to-reuse-recorded-resolutions)
- [The concept of "ours and theirs"](#The-concept-of-ours-and-theirs)
- [References](#References)

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
On branch <BRANCH>
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

In either case, once the changes have been chosen, add the file with `git add <FILE>` and then commit the changes of the merge with `git commit`.

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
On branch prod
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
error: could not apply <COMMIT_HASH>... <COMMIT_TITLE>
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply <COMMIT_HASH_INCOMING>... <COMMIT_TITLE>
```

In this case, the error message explains that there were two files with changes: `<FILE_1>` and `<FILE_2>`. The changes from `<FILE_2>` were not in conflict, so they were automatically merged. However, the changes from `<FILE_1>` did experience a conflict.

By running `git status`, the user can more easily see what files are in conflict:

```
interactive rebase in progress; onto <COMMIT_HASH_INCOMING>
Last command done (1 command done):
   pick <COMMIT_HASH_TIP> <COMMIT_TITLE>
No commands remaining.
You are currently rebasing branch '<BRANCH>' on '<COMMIT_HASH_INCOMING>'.
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
2
=======
1
>>>>>>> <COMMIT_HASH_OF_CONFLICT_ON_PROD> (<COMMIT_TITLE_OF_CONFLICT_ON_PROD>)
```

At this point, it's worth noting that `HEAD` and the "incoming changes" are in the same place, with `HEAD` on top and the "incoming changes" on bottom. However, what `HEAD` and the "incoming changes" reference have changed -- `HEAD` now refers to `dev`, and the "incoming changes" are the changes from `prod`. This is the opposite of a merge conflict, where `prod` was `HEAD` and `dev` was the "incoming change".

This should make sense with an understanding of how `git merge` and `git rebase` work. When merging, branch `prod` is checked out, and changes from `dev` are being merged onto `prod`, so `dev` would be the "incoming changes" to `prod`. In contrast, with a rebase, branch `dev` is checked out, so `prod` would be the "incoming changes" to `dev`. This is the concept of "ours and theirs", and will be explained more in a following section.

As with a merge conflict, the user will choose which changes to keep and which to discard. Once the changes have been chosen, add the file with `git add <FILE>` and then continue the rebase with `git rebase --continue`.

If there are no other conflicts, the rebase will complete successfully. If the user runs `git log` once the rebase completes, they will see no merge commits. As discussed in the section on ["git-rebase"](git-rebase.md#Introduction), this is to be expected, as `git rebase` does not create merge commits.

#### Rejecting the "incoming change" when rebasing

To choose the change made in branch `dev` (`HEAD`), remove the following lines:

```
<<<<<<< HEAD
=======
1
>>>>>>> <COMMIT_HASH_OF_CONFLICT_ON_PROD> (<COMMIT_TITLE_OF_CONFLICT_ON_PROD>)
```

This would leave the file with the following contents:

```
2
```

The user may want to run `git status` between `git add` and `git commit`, to verify the state of the branch before committing. If the user chose to keep the changes from `dev` (`HEAD`), and rejected the "incoming changes" from `prod`, doing so would display the following message:

```
interactive rebase in progress; onto <COMMIT_HASH_INCOMING>
Last command done (1 command done):
   pick <COMMIT_HASH_TIP> <COMMIT_TITLE>
No commands remaining.
You are currently rebasing branch 'dev' on '<COMMIT_HASH_INCOMING>'.
  (all conflicts fixed: run "git rebase --continue")
```

This is an indication that the "incoming changes" were rejected, and the "change" is essentially empty. The branch `dev` is checked out, so rejecting the changes from `prod` would result in no changes to `newfile`.

#### Accepting the "incoming change" when rebasing

To choose the change made in branch `prod` (`<COMMIT_HASH_OF_CONFLICT_ON_PROD>`), remove the following lines:

```
<<<<<<< HEAD
2
=======
>>>>>>> <COMMIT_HASH_OF_CONFLICT_ON_PROD> (<COMMIT_TITLE_OF_CONFLICT_ON_PROD>)
```

This would leave the file with the following contents:

```
1
```

The user may want to run `git status` between `git add` and `git commit`, to verify the state of the branch before committing. If the user chose to accept the "incoming changes" from `prod`, doing so would display the following message:

```
interactive rebase in progress; onto <COMMIT_HASH_INCOMING>
Last command done (1 command done):
   pick <COMMIT_HASH_TIP> <COMMIT_TITLE>
No commands remaining.
You are currently rebasing branch 'dev' on '<COMMIT_HASH_INCOMING>'.
  (all conflicts fixed: run "git rebase --continue")

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   newfile
```

This is an indication that the "incoming changes" were accepted. Since there is a change, as compared to the working tree of `dev`, then those changed need to be committed.

As mentioned in the section on [conflicts during a rebase](#Conflicts-during-a-rebase), if the "incoming change" is accepted during a rebase, then it's almost a guarantee that any other commits that are played back during the rease process will also encounter the same conflict. This is due to the nature of how `git rebase` works -- by accepting the "incoming change", then other commits played back during the rebase will continue to reconflict, since the conflict exists in those other commits.

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

- When merging `dev` onto `prod`, `prod` was the current branch, and `git merge dev` was the command that was run.
    - Therefore, `prod` was "ours", and `dev` was "theirs".
- When rebasing, `dev` was the current branch, and `git rebase prod` was the command that was run.
    - Therefore, `dev` was "ours", and `prod` was "theirs".

When resolving a conflict, it is possible to choose the changes from "ours" by running the following command:

```
git checkout --ours <PATH>
```

Or, to accept the changes from "theirs":

```
git checkout --theirs <PATH>
```

Note that `<PATH>` can be anything from a "file in conflict" to `.` (the entire current working directory).

Also, in the same way as `git add` and `git restore`, `git checkout` can also be used with the `-p` switch, allowing the user to perform a "patch checkout". This is an extremely useful tool when a majority of changes from "ours" or "theirs" are to be accepted. As an example, if most changes from "ours" are going to be accepted across all "files in conflict", the user could run `git checkout -p --ours .`, and then choose `y` for most of those changes. Then, the user could run `git checkout -p --theirs .` and select `a` to have all the remaining conflicts accept the incoming changes.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Rebasing is Complicated](https://theprimeagen.github.io/fem-git/lessons/going-remote/more-rebasing-and-merging)
    - A deep dive into resolving merge conflicts, both when merging and rebasing
