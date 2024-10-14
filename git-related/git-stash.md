# git-stash - Saving Changes Without Commiting

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Pushing changes onto the stack](#Pushing-changes-onto-the-stack)
    - [Patch adding to the stack](#Patch-adding-to-the-stack)
- [Listing changes on the stack](#Listing-changes-on-the-stack)
- [Popping changes off the stack](#Popping-changes-off-the-stack)
- [Dropping changes from the stack](#Dropping-changes-from-the-stack)
- [Using "git-stash" with interactive rebases](#Using-git-stash-with-interactive-rebases)
- [References](#References)

## Introduction

`git-stash` is a useful command to understand, as it allows a user to save changes without commiting them to the repo. Without `git-stash`, the options are limited and underwhelming.

- A user could commit partial changes, merge in the `upstream` changes with a merge commit, and then commit again in the future to complete, or fix, the partial commit. This can get messy.
- Alternatively, the user could use `git rebase` to update the base of their branch to the tip of the `upstream`. This is more advisable than merging.

However, the user may to store their changes for later. This is where `git stash` shines.

What `git stash` does is take every change that's *tracked by Git* (changes to the repo's index) and hides them away in a stack called the "stash". A stack is a simple data structure that works under the principle of "last in,f irst out" (LIFO). There are two main operations with a stack: pushing and popping. Pushing adds an element to the stack; and popping removes the most recently added element. It's far easier to work with `git stash` when it's understood that the "stash" is simply an implementation of a stack. As a byproduct of this fact, this section will continuously refer to the "stash" as the "stack".

## Pushing changes onto the stack

Pushing changes onto the stack is easy -- simply run the following command:

```
git stash
```

Note that `git stash` is a shorthand for `git stash push`: both commands will push the current changes into the stack.

Just like with commits, stashed objects can have messages. Unlike commits, these message are best contained to a single line -- essentially, the message will act as a "title" to the stashed object. To add a message when pushing to the stack, run the following command:

```
git stash -m <COMMIT_MESSAGE>
```

As with `git stash` and `git stash push`, the command `git stash -m <COMMIT_MESSAGE>` also works interchangeably with `git stash push -m <COMMIT_MESSAGE>`.

A helpful suggestion: if a stash object is going to be saved for any length of time, it can be a good idea to add a date to the message. That way, it's easy to know in the future when each element was added to the stack.

### Patch adding to the stack

Just like with `git add`, `git stash` can be used with the `-p` switch, allowing the user to choose what hunks get added to the stash object. However, in order to use the `-p` switch *and* add a message, the user *must* run `git stash push -pm` -- `git stash -pm` does not work.

Note that patch adding to the stack only works on files that already exist in the working tree -- new files cannot be added to the stack with `-p`.

## Listing changes on the stack

To view the changes that have been added to the stack, run the following command:

```
git stash list
```

To output this list directly into the terminal's `STDOUT`, pipe `git stash list` into `cat`, as seen below:

```
git stash list | cat
```

It is also possible to view the dates when objects were added to the stack with the following command:

```
git stash list --date="local"
```

As a contrast to `git stash list`, `git stash show` will show the top element of the stack in "diff" format. To see the actual changes made in that stashed object, run the following command:

```
git stash show -p
```

To show the changes of a stashed object that isn't at the top of the stack, run the following command:

```
git stash show stash@\{<INDEX>}
```

Imagine that the user has two objects in their stash. `git stash show` works by showing the most recently added element, which would be the object at position `0`. But, what if the user wants to show their first stash, the other stashed object, which is now at position `1`? The user could run `git stash show stash@\{1\}` to show that stashed entry. Note that a user can also pass an index to `git stash show -p`, as well any any other `git stash` command that operates by default on the element at the top of the stack.

## Popping changes off the stack

To pop the top element off the stack, run the following command:

```
git stash pop
```

While `git stash pop` works by popping the most recent addition off the stack and back into the working tree, what happens if a user wants to pop a different stashed object?

To pop any specific element from the stack, run the following command:

```
git stash pop stash@\{<INDEX>}
```

Imagine that the user has two objects in their stash. `git stash pop` works by popping the most recently added element, which would be the object at position `0`. But, what if the user wants to pop their first stash, the other stashed object, which is now at position `1`? The user could run `git stash pop stash@\{1\}` to pop that stashed entry.

Note that, unlike `git stash show`, `git stash pop` can be run with the `--index` switch. Instead of popping element `stash@\{1\}`, the user can run `git stash pop --index 1`.

## Dropping changes from the stack

To drop the element at the top of the stack, run the following command:

```
git stash drop
```

Just like with `git stash pop`, the user is able to specify which element of the stash to drop with the following command syntax:

```
git stash drop stash@\{<INDEX>}
```

However, just like with `git stash show` (and unlike `git stash pop`), the `--index` switch does not work with `git stash drop`.

## Using "git-stash" with interactive rebases

`git stash` is a great tool to use in conjunction with `git rebase -i`. For more information, refer to the section on [interactive rebases](interactive-rebase.md#Adding-a-new-commit-to-the-middle-of-the-history).

As an example, imagine a situation where a user begins to make changes to their working tree, but only after decides to interactively rebase the changes into an older commit. The user can push the changes to the stack, run `git rebase -i`, and then pop the changes off the stack once working in the interactive rebase environment.

In fact, if a user attempts to run `git rebase -i` while changes to the index are present, Git will refuse to begin the interactive rebase until the changes are reverted, committed, or stashed.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Stash](https://theprimeagen.github.io/fem-git/lessons/going-remote/stash)
    - A deep dive into using `git stash`
- [Git SCM documentation - git stash](https://git-scm.com/docs/git-stash)
    - Reference for `git stash`
- [StackOverflow - How do I name and retrieve a Git stash by name?](https://stackoverflow.com/questions/11269256/how-do-i-name-and-retrieve-a-git-stash-by-name)
    - Reference for `git stash push -m`
- [StackOverflow - Get the creation date of a stash](https://stackoverflow.com/questions/15551618/get-the-creation-date-of-a-stash/15551690#15551690)
    - Reference for `git stash list --date="local"`
