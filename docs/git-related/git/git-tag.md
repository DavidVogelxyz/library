git-tag - Creating Tags
=======================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Adding a tag to a commit](#adding-a-tag-to-a-commit)
- [Listing tags](#listing-tags)
- [Deleting a tag](#deleting-a-tag)
- [Syncing tags with a remote](#syncing-tags-with-a-remote)
- [References](#references)

Introduction
------------

Tags are a useful way to mark a commit with an identifier.

A tag can serve many purposes. Maybe a user wants to mark a commit as the most recent `stable` version of the code; or, a team may want to specify a commit as a new version of a software. In another case, consider a user who has a `base` commit, from which many different branches will diverge.

In these situations, a user can use `git tag` to add a name to a commit, which can then be used in the same way as a hash. A tag can be referenced with commands such as `git checkout <TAG>` in order to quickly navigate the commit history.

Note that a tag essentially acts as a branch that cannot be changed. Therefore, to remove a tag, it must be deleted.

Adding a tag to a commit
------------------------

To add a tag to a commit, run the following command:

```bash
git tag <TAG>
```

This will apply `<TAG>` to whichever commit is currently `HEAD`. To apply a tag to a commit that occurs at some point in history, that commit must first be checked out.

Listing tags
------------

To view all tags, run the following command:

```bash
git tag
```

Deleting a tag
--------------

To delete a tag, run the following command:

```bash
git tag -d <TAG>
```

Syncing tags with a remote
--------------------------

Tags, by default, do not sync with a `remote` when running `git push`, and don't sync with the local version when running `git pull`.

This is similar to a branch, in that only the active branch is "pushed" or "pulled". Because a tag doesn't belong to any branch, and can be considered its own immutable branch, running `git push` or `git pull` will not bring along tag information, even if the tagged commit is part of the branch being "pushed" or "pulled".

In order to push tags to a remote, run the following command:

```bash
git push --tags
```

To pull tags from a remote, run the following command:

```bash
git pull --tags
```

Note that the `--tags` switch directs `git push` or `git pull` to grab tags *in addition to* branches.

References
----------

- [thePrimeagen - Everything You'll Need to Know About Git - Tags](https://theprimeagen.github.io/fem-git/lessons/git-gud/tags)
    - A quick summary of how to use `git tag`
