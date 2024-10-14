# Git's Internal File Structure

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [The files are in the computer](#The-files-are-in-the-computer)
- [What is HEAD?](#What-is-HEAD)
- [Excluding files from "git-add" with the ".gitignore" file](#Excluding-files-from-git-add-with-the-gitignore-file)
- [Excluding files from "git-add" with the ".git/info/exclude" file](#Excluding-files-from-git-add-with-the-gitinfoexclude-file)
- [References](#References)

## Introduction

Almost everything that Git works with is assigned a 40 character SHA (hash) value. Most people are familiar with the SHA values that denote their commits, but this is true as well for trees (directories), blobs (files), and refs (references). While the SHAs are 40 characters each, the first 7 characters are almost always unique. A SHA is written in hex format (16 options per character: "a-f" and "0-9"), so a 7 character hash value has 268,435,456 different values it can be (16^7).

SHA values are unique because they take in many outputs, including date and time, author name and e-mail, contents of files, and many other factors, when creating their outputs. Even if two users create the same repos, with the same names and file contents, and commit the same changes at the same time, they will receive two very different SHA values. In addition, if a minor change is made to a file, the hash of the file will be wildly different than the hash from before the change. For these reasons, SHAs are a great way to differentiate between files, commits, and refs.

Also, as stated in the section on ["git-config"](git-config.md#Viewing-the-config-files), everything that Git works with has a file associated with it, and those files can be found in the `.git` directory.

With these facts in mind, it's possible to work within Git to manipulate those files in extraordinary ways.

## The files are in the computer

Let's say a user creates their first commit, and the hash is `aabbccd`. If that user was to look into their `.git` directory, they would find a directory in the `.git/objects` directory with the name `aa`. Looking inside that directory would reveal a file: `bbccd`. This is a demonstration that a representation of a commit exists in the `.git/objects` directory.

What can be done with this information?

Using `cat` on this file will produce a random output that's completely unreadable. However, the command `git cat-file -p` *can* work with this file. An example of this command can be found below:

```
git cat-file -p aabbccd
```

This produces an output that looks almost exactly like the output for a commit entry from the `git log` command, but it has some additional information. There's a reference to a tree, and following it is another hash. What happens when the user runs `git cat-file -p` on that hash?

```
git cat-file -p <SHA>
```

Now, an output shows that heavily resembles the output from the `ls` command. It includes the file/directory permissions in octal format, the denotation "blob" or "tree", another SHA, and then the name of the file/directory.

This is an insight to how Git's internal file system works. Everything is denoted with a hash. These hashes refer to files that can be found in the `.git` directory. If a user uses `git cat-file -p` to walk the tree of hashes, they eventually will arrive at the SHA values for specific versions of files. If `git cat-file -p` is run on a SHA value for a blob, then the output is the *entirety* of that file. Git actually stores entire files in the `.git` directory -- it does ***not*** store diffs!

This is how Git is able to rebuild states of projects in their entirety, and do so very quickly. If a blob (file) or tree (directory) doesn't change between two commits, then the new commit will just store pointers to the last hash where the file changed. As an example, when a user uses `git checkout` to load a previous commit, Git takes the hash of that commit and walks the tree, eventually grabbing the directories and files that were present at the time of that commit, in the state that they were in at that time.

This wasn't said earlier, but running `git cat-file -p` on a commit that isn't the first commit of a repo will also include information on the "parents" of that commit. The second commit to the example repo will contain a line that includes `parent aabbccd`. This is how Git stores the history information for a Git repo -- the next commit in the chain includes a pointer to the previous commit. This is the basic principle behind `git log` -- Git starts at the current commit, gets the hash of the previous commit, and walks the tree back to the first commit. Since the first commit is the only commit that doesn't have a parent, Git knows that it's at the beginning of the commit history.

## What is HEAD?

`HEAD` is simply a pointer to the tip of the current branch. To learn more about branches, refer to [this section](git-branch.md) about `git-branch`.

Consider an example repo with two branches, `prod` and `dev`. If `prod` is checked out, then `HEAD` will point to the tip of `prod`. If `dev` is checked out, then `HEAD` points to the tip of `dev`.

This can be seen in the files within the `.git` directory. Assuming `prod` is checked out, running `cat` on the file `.git/HEAD` will return the following output:

```
ref: refs/heads/prod
```

Running `cat` on the file referenced by `.git/HEAD` (`.git/refs/heads/prod`) returns the commit SHA of the tip of `prod`.

Therefore, as commits are added to `prod`, `HEAD` will update accordingly, but without having to change this pointer!

When a different branch, such as `dev`, is checked out, then `.git/HEAD` will simply point to `.git/refs/heads/dev`. And, `.git/refs/heads/dev` will point to the commit SHA of the tip of `dev`.

## Excluding files from "git-add" with the ".gitignore" file

One of the most important Git files is the `.gitignore` file, and it's one of the few that isn't found in the `.git` directory. Instead, it is found in the root of the repo.

`.gitignore` is a very simple file -- it's a list of paths that `git add` will ignore when looking for candidates to be added to the index.

Consider the following example: a user has two directories with secrets in their repo (`secdir1` and `secdir2`). Within `secdir1`, there is the file `secfile`. Within `secdir2`, there are two files: `secfile1` and `secfile2`.

One way to ignore these files is to specify the following in a `.gitignore` file:

```
secdir1/secfile
secdir2/secfile1
secdir2/secfile2
```

By specifying the paths of these files, Git will know to ignore these files. Even with a `git add .`, these files will never be able to be staged, unless these entries are removed from the `.gitignore` file. However, if the user then created the file `secdir2/secfile3`, this file *could* be accidentally added to the index, because it is not specified in the `.gitignore` file.

As mentioned previously, the `.gitignore` file is a list of paths. Therefore, a clever user might instead have a `.gitignore` with the following paths:

```
secdir1/
secdir2/
```

By specifying the paths of the directories, Git will never allow any file contained in `secdir1` or `secdir2` to be staged. However, if the user created a new directory `secdir3`, that directory's files *could* be accidentally added to the index. As before, this is because `secdir3` is not specified in the `.gitignore` file.

But, the `.gitignore` file can also work with patterns. An even more clever user might put the following into their `.gitignore` file:

```
sec*
```

Now, any file or directory that starts with `sec` will not be able to be added to the index.

Obviously, the more granular the path, the more likely the user will have to add another path in the future, if a new sensitive file is added. The less specific the path, the more likely that a file that isn't sensitive will be ignored by the `.gitignore` file. A capable user will pick the right combination of file paths, directory paths, and patterns to suit their needs. Since the `.gitignore` file is local to the repo, it can be customized to the needs of each individual repo.

## Excluding files from "git-add" with the ".git/info/exclude" file

The one catch to the `.gitignore` file is that it is generally added to the repo's working tree. While it is possible to ignore the `.gitignore` file (by specifying its path in the `.gitignore` file), it is customary to include the file in the working tree. For example, a Python repo may have the `.env` directory ignored, because it is user-generated and (usually) a large directory, but nothing sensitive is contained within.

However, there are situations where a `.gitignore` is being used to ignore certain files that are not sensitive, but the user doesn't want to advertise to anyone reviewing the repo about the paths to actual sensitive files. In this case, there is a secondary file that exists in the `.git` directory at the path `.git/info/exclude`. This file works the same way as a `.gitignore` file -- but, it allows a user to list paths here without including them in the `.gitignore` file. Therefore, the path `PATH/TO/SECRET` can be safely ignored by `git add` without advertising its path to the public.

## References

- [thePrimeagen - Everything You'll Need to Know About Git - A Brief View into Git](https://theprimeagen.github.io/fem-git/lessons/intro/a-brief-view-into-git)
    - "Going past the porcelain and checking out the plumbing" of Git
- [thePrimeagen - Everything You'll Need to Know About Git - HEAD](https://theprimeagen.github.io/fem-git/lessons/branches-merges-and-more/head)
    - Information on how `HEAD` works
- [StackOverflow - Git seems to store the whole file instead of diff, how to avoid that?](https://stackoverflow.com/questions/41482898/git-seems-to-store-the-whole-file-instead-of-diff-how-to-avoid-that/41484463#41484463)
    - Reference regarding how Git stores "whole files and not diffs"
- [StackOverflow - Git internals: how does Git store small differences between revisions?](https://stackoverflow.com/questions/43359590/git-internals-how-does-git-store-small-differences-between-revisions/43364484#43364484)
    - Reference regarding "Git objects" being complete copies of files, and not diffs
