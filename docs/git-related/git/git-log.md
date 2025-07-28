# git-log - Viewing a History of Changes

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [git-log-all](#git-log-all)
- [git-log-decorate](#git-log-decorate)
- [git-log-oneline](#git-log-oneline)
- [git-log-graph](#git-log-graph)
- [git-log-parents](#git-log-parents)
- [Showing only a certain number of commits back from HEAD](#showing-only-a-certain-number-of-commits-back-from-head)
- [Searching through "git-log"](#searching-through-git-log)
- [Reviewing changes with "git-log" and "git-show"](#reviewing-changes-with-git-log-and-git-show)
- [Useful aliases for "git-log"](#useful-aliases-for-git-log)
- [References](#references)

## Introduction

`git log` is the easiest and most approachable way for a user to check the commit history and view the changes made to a repository. This section of the guide will review some additional options and ways to utilize `git log` to get more information out of the commit history.

As with other Git commands, switches that are detailed in this guide can be used together. A well cited acronym for useful `git log` switches is "a dog" -- `--all`, `--decorate`, `--oneline`, and `--graph`.

## git-log-all

The switch `--all` does what it says -- it includes all the refs in the log output. This can be useful, for example, to include hunks that have been stashed into the history output.

## git-log-decorate

The switch `--decorate` add branch names and tags (such as HEAD) to the commit history. When using `git log` in the CLI, these indicators appear by default. However, if the output of `git log` is redirected to a file, these indicators do not show unless the `--decorate` switch is added.

To see the difference, change the working directory to a Git repo and run the following to output `git log` to a file:

```
git log > no-decorate.txt
```

Now, do the same with the `--decorate` switch:

```
git log --decorate > yes-decorate.txt
```

Notice that `--decorate` includes some useful information that normally appears when using `git log`, but will not appear when `git log` is output to a file.

## git-log-oneline

The `--oneline` switch tells Git to condense each ref into a single line. This can be incredibly useful in a history that includes many refs and commits.

`git log --oneline` will render commits and refs into a line that follows a simple format:

```
<UNIQUE_COMMIT_HASH> <TAGS> <TITLE_OF_COMMIT>
```

## git-log-graph

The switch `--graph` is very useful when a commit history includes merges. This switch tells Git to draw a graphical representation of the commit history's timeline, which can assist with identifying where history diverges, and where it converges again.

As is discussed more in the section on ["git-merge"](git-merge.md#how-to-perform-a-three-way-merge), running `git log --decorate --oneline --graph` would produce an output similar to the following:

```
*   <HASH_MERGE_COMMIT> (HEAD -> prod) Merge branch 'dev' into prod
|\
| * <HASH_C> (dev) C
| * <HASH_B> B
* | <HASH_E> (prod) E
* | <HASH_D> D
|/
*   <HASH_A> A
```

By viewing the above commit history with the `--graph` switch, it becomes clear how the tip of branch `prod` (`E`) and the tip of branch `dev` (`C`) were merged together to create a new commit on `prod`.

## git-log-parents

What if the user wanted a bit more information than what was provided with the `--graph` switch?

The switch `--parents` is also very useful in a commit history with merges. It will include the hash of the parent commit in the following format:

```
<UNIQUE_COMMIT_HASH> <PARENT_HASH> <TAGS> <TITLE_OF_COMMIT>
```

If a commit is a merge commit, and has multiple parents, the format would look something like:

```
<UNIQUE_COMMIT_HASH> <PARENT_1_HASH> <PARENT_2_HASH> <TAGS> <TITLE_OF_COMMIT>
```

When paired with `--graph`, a clear image of the commit history is rendered by `git log`. **To see most clearly the divering and convering paths, run the following command**:

```
git log --decorate --oneline --graph --parents
```

As is discussed more in the section on ["git-merge"](git-merge.md#how-to-perform-a-three-way-merge), running `git log --decorate --oneline --graph --parents` would produce an output similar to the following:

```
*   <HASH_MERGE_COMMIT> <HASH_E> <HASH_C> (HEAD -> prod) Merge branch 'dev' into prod
|\
| * <HASH_C> <HASH_B> (dev) C
| * <HASH_B> <HASH_A> B
* | <HASH_E> <HASH_D> (prod) E
* | <HASH_D> <HASH_A> D
|/
*   <HASH_A> A
```

Using this set of switches, `git log` produces an even more clear picture of how the commit history diverges and reconverges.

## Showing only a certain number of commits back from HEAD

To show `git log` only for a certain number of commits from the tip of the branch (ex. `1`), run the following command:

```
git log -1
```

As with other switches for `git log`, this switch can be combined with other switches, such as `git log --oneline -5`.

For more information on `HEAD`, refer to the [section on Git's internal file structure](git-internal-file-structure.md#what-is-head).

## Searching through "git-log"

There are a few ways to search the commit messages of `git log`.

One way, though less optimal than other, is to pipe the output of `git log` into `grep`, and search for a keyword.

```
git log | grep <KEYWORD>
```

However, `git log` comes with a built-in `grep` function. This can be utilized with the following command:

```
git log --grep "<KEYWORD>"
```

`git log --grep` can be great in situations where the `<KEYWORD>` can be found in the commit message. But, what if the user knows what to look for within the file itself? In that case, the user can run `git log -S "<KEYWORD>"` to search through the text *of the changes*. To do so, run the following command:

```
git log -S "<KEYWORD>"
```

`git log -S` can be extremely useful -- but, it's searching through the text of the changes, and the user can only view the commit message. To view the text of the changes at the same time, add the `-p` switch to the command. To learn more about `git log -p`, check out the section on [reviewing changes through "git-log"](#reviewing-changes-through-git-log).

## Reviewing changes with "git-log" and "git-show"

`git log` can also be used to view the changes of commits.

To do so, run the following command:

```
git log -p
```

To see the changes only from `HEAD` back a certain number of commits (ex. `1`), run the following command:

```
git log -p -1
```

The number passed will denote how many commits back from `HEAD` that should be shown. In this case, `1` is equivalent to `HEAD~1`, and represents "show the changes between `HEAD` and the previous commit".

However, to see the changes for only a single commit, run the following command:

```
git show <COMMIT_HASH>
```

It's also possible to search changes by file using `git log`. To do so, run the following command:

```
git log -p -- <PATH>
```

Note that `git log -p -- <PATH>` can be run on any path, whether it be to a file or a directory.

Just as `git show <COMMIT_HASH>` will show the changes for a specific commit, `git show <COMMIT_HASH> -- <PATH>` will show the changes for a specific path, within that single commit.

```
git show <COMMIT_HASH> -- <PATH>
```

`git log -p` can be combined with `--grep`, so that Git will show the changes of any commit with the keyword passed to `--grep`:

```
git log -p --grep "<KEYWORD>"
```

As mentioned in the section on [searching through "git-log"](#searching-through-git-log), `git log -p` can also be combined with the `-S` switch in order to search through the text of the commits. To do this, run the following command:

```
git log -pS "<KEYWORD>"
```

This command will allow a user to search through the text of the changes, as well as view the text containing the keyword.

## Useful aliases for "git-log"

An example of some useful aliases for `git log` are shown below:

```
[alias]
    lg = lg1
    lg1 = log --all --decorate --graph --abbrev-commit --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(white)- %an%C(reset)%C(auto)%d%C(reset)'
    lg2 = log --all --decorate --graph --abbrev-commit --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(auto)%d%C(reset)%n''          %C(white)%s%C(reset) %C(white)- %an%C(reset)'
```

See the [following reference](https://stackoverflow.com/questions/1838873/visualizing-branch-topology-in-git/34467298#34467298) for images of what these `git log` aliases look like.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Your First Git Repo](https://theprimeagen.github.io/fem-git/lessons/intro/your-first-git-repo)
    - Basic Git log use
- [thePrimeagen - Everything You'll Need to Know About Git - Bisect](https://theprimeagen.github.io/fem-git/lessons/git-gud/bisect)
    - Using `git log --grep`, `git log -S`, and `git log -p`
- [StackOverflow - Visualizing branch topology in Git](https://stackoverflow.com/questions/1838873/visualizing-branch-topology-in-git/34467298#34467298)
    - Reference for useful `git log` aliases
