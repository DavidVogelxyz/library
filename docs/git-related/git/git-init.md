# git-init - Initializing a New Git Repository

[Back to the home page](../README.md)

## Table of contents

- [Introduction](#introduction)
- [Using "git-init" to initialize a new Git repository](#using-git-init-to-initialize-a-new-git-repository)
- [Bare repositories](#bare-repositories)

## Introduction

`git init` is a command that allows the user to make any directory into a Git repo. Under the hood, `git init` works by creating a `.git` subdirectory, converting into a Git repo the directory on which `git init` operates.

As was discussed in [git-config](git-config.md#viewing-the-config-files), and will be discussed further in [Git's internal file structure](git-internal-file-structure.md#introduction), the `.git` directory is where all of the Git files for a given repo are stored. Any directory can become a Git repo by creating a `.git` directory with `git init`.

`git init` is a relatively simple command, but there are some key facts to note.

## Using "git-init" to initialize a new Git repository

There are two primary ways to use `git init`:

1. The user can run `git init` within a directory -- the command will create a `.git` subdirectory in the current working directory, turning that directory into the root of the Git repo.
1. The user can run `git init <PATH>`, and can specify the `<PATH>` to the directory that should be converted into a Git repo.

Most users will change directory (`cd`) into the directory, and then run `git init`. However, `git init` can also have a `<PATH>` passed as an argument.

Note that, in either case, the directory has to exist *prior* to running the `git init` command.

## Bare repositories

Bare repositories are Git repos that do not store any of the "actual files"; rather, bare repos only store the Git files. Put another way, a bare repository is a repo where there is no `.git` directory -- instead, the contents of the `.git` directory are found in the repo's root directory.

Bare repositories are very useful, in certain situations. Obviously, if the files that are being tracked by Git don't exist as "regular files", a bare repository is not useful for a user who wants access to the files. However, a bare repository is incredibly useful for a Git server, where there is no need to store the "actual files". Using a bare repository in this context can cut down the storage space requirement of a repo by about 50%.

To initialize a directory as a bare repository, the user will use the `--bare` switch with the command, as in the following example:

```
git init --bare
```

To dive a bit deeper, the `config` file for a bare repo will differ slightly from a "non-bare" repo.

The following is a `config` file from a bare repo:

```
[core]
        repositoryformatversion = 0
        filemode = true
        bare = true
```

The following is a `.git/config` from a "non-bare" repo:

```
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
```

Note that there are only two keys that differ. The obvious key, `bare`, has the value `true` in a bare repo, and the value `false` in a "non-bare" repo. The other key, `logallrefupdates`, is a concatentation of "log all ref updates", and instruct the Git repo on whether it should keep a `reflog` or not. By default, bare repos **do not** store a `reflog`.
