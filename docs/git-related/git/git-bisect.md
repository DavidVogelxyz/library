git-bisect - Testing to Find a "Breaking Change" in a Branch
============================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Understanding binary search](#understanding-binary-search)
- [Using "git-bisect" to find "breaking changes"](#using-git-bisect-to-find-breaking-changes)
- [Using "git-bisect" to automate searching for "breaking changes"](#using-git-bisect-to-automate-searching-for-breaking-changes)
- [References](#references)

Introduction
------------

Sometimes, a user will discover that some code they're working on is breaking. There are times when the change was introduced in a recent commit, so it's easy to address. In other cases, it may be possible using `git log` and its switches to find the commit where the "breaking change" occurred. But, what if the snippet of code was committed at an unknown time, in an unknown commit, and the history is hundreds of commits long?

This situation can happen, and it can be an absolute nightmare to resolve, if the user is unaware that there are tools at their disposal that can assist them.

Enter `git bisect`.

`git bisect` is a very simple tool. Provided a test to run, it can perform a "binary search" on a range of commits, and will return to the user a commit hash where the test first fails.

As mentioned, `git bisect` is not always the best tool to use:

- In the case of a repo with a small commit history, with infrequent changes, it may be easier to run `git log -p`.
- If a certain file is known to contain the "breaking change", then the "break" may be identifiable with `git log -p -- <FILE>`.
- If the failing snippet of code has a unique keyword, then `git log -S <KEYWORD>` may be the best tool.
    - An example of this would be a specific function that is known to be failing.
- If a repo has well-written commit messages with useful keywords, `git log --grep "<KEYWORD>"` may return the answer faster than `git bisect`.

But, if any (or all) of the following are true:

- the history has hundreds of commits
- the file where the "break" occurs changes often
- the commit messages are unhelpful
- too many commits return when using `git log -S <KEYWORD>` or `git log --grep "<KEYWORD>"`

Then, it may be advisable to run `git bisect` to find the "breaking change".

Understanding binary search
---------------------------

`git bisect` operates based on two assumptions:

1. Commits are chronologically ordered.
    - This is the most important assumption. It is the basis for the understanding that, if a history is 100 commits long, and `git bisect` checks commit `50` and can verify the problem exists, then there's no need to check commits `51` through `100`. In contrast, if the issue is **not** present, then the change *must* have occurred between commits `1` and `49`.
1. The user can specify a range in which the change occurred.

Assumption #1 is the basis for the "binary search" algorithm. To understand it visually, refer to the following diagram:

```
 ----------------------------------
 |                                |
 1 ---------- unknown ---------- 100
```

If it's known that commit `1` works, and commit `100` fails, then the "breaking change" lies somewhere in the middle. If the middle commit is chosen:

```
 -----------------------------------
 |                                 |
 1 -- unknown -- 50 -- unknown -- 100
```

If `50` contains the "breaking change", then only `1` through `50` need to be searched. If `50` does not contain the "breaking change", then only `51` through `100` need to be searched. The "binary search" algorithm effectively reduced the "search space" in half, by testing against only one commit.

This process can be iterated upon, until the "breaking commit" is discovered.

So long as the user has a method by which the change can be identified with a "pass/fail" test, then utilizing "binary search" using `git bisect` is the quickest way to search a sorted range in order to find where the change occurs.

`git bisect` can be used to find more than just bugs. If the user has the right test, it can search for nearly any type of change.

Since `git bisect` is just a "binary search" under the hood, it follows the same "time complexity" -- `O(log n)`. What this means is the following:

```
COMMITS     =   SEARCHES
1           =   1
2           =   1
4           =   2
8           =   3
16          =   4
32          =   5
64          =   6
128         =   7
256         =   8
512         =   9
```

Using "git-bisect" to find "breaking changes"
---------------------------------------------

To start `git bisect`, run the following command:

```bash
git bisect start
```

Set the current commit as a "bad" commit with the following command:

```bash
git bisect bad
```

Set the "last known good commit" with the following command:

```bash
git bisect good <COMMIT_HASH>
```

Now, the user is in `git bisect` mode. At this point, the user would run the test and determine if the commit is "good" or "bad".

If the test passes, tell Git with the following command:

```bash
git bisect good
```

Or, if the test fails:

```bash
git bisect bad
```

This process continues until `git bisect` arrives at the commit where the "breaking change" occurs, returning the following message:

```
<COMMIT_HASH> is the first bad commit
```

Once `git bisect` completes, the user will run the following command to exit `git bisect` mode:

```bash
git bisect reset
```

Using "git-bisect" to automate searching for "breaking changes"
---------------------------------------------------------------

While `git bisect` is a great tool, manually running the test for each search is a bit laborious. Fortunately, `git bisect` can be automated!

The user will begin the same way:

```bash
git bisect start
git bisect bad
git bisect good <COMMIT_HASH>
```

At this point, the user can run the following command:

```bash
git bisect run <TEST_COMMAND>
```

For a Bash script, this command may appear as `git bisect run ./test.sh`. For JavaScript code, the `<TEST_COMMAND>` may be a node module, such as `git bisect run ./node_modules/.bin/test --run`.

Regardless of the `<TEST_COMMAND>`, `git bisect` will run the test and arrive at the "first bad commit" as it did before!

References
----------

- [thePrimeagen - Everything You'll Need to Know About Git - git bisect](https://theprimeagen.github.io/fem-git/lessons/git-gud/bisect)
    - A comprehensive overview of how to use `git bisect`
