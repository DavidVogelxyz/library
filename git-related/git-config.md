# git-config - Setting Up a New Git Environment

[Back to the home page](README.md)

## Table of contents

- [Introduction](#Introduction)
- [Setting up a new Git environment](#Setting-up-a-new-Git-environment)
- [Viewing Git configurations](#Viewing-Git-configurations)
    - [Using "git-config-get" to get the value of a configuration](#Using-git-config-get-to-get-the-value-of-a-configuration)
    - [Viewing the config files](#Viewing-the-config-files)
    - [Using "git-config-list" to list all the configuration values](#Using-git-config-list-to-list-all-the-configuration-values)
- [Editing Git configurations](#Editing-Git-configurations)
    - [Using "git-config-edit" to edit Git configurations](#Using-git-config-edit-to-edit-Git-configurations)
    - [Editing the config file](#Editing-the-config-file)
- [A key can have multiple values](#A-key-can-have-multiple-values)
- [Signing commits with GPG keys](#Signing-commits-with-GPG-keys)
- [Example configurations in the Git config files](#Example-configurations-in-the-Git-config-files)
    - [Example global config file](#Example-global-config-file)
        - [Global configs for "git-difftool"](#Global-configs-for-git-difftool)
        - [Global configs for Git aliases](#Global-configs-for-Git-aliases)
    - [Example local config file](#Example-local-config-file)
    - [Other configs](#Other-configs)
- [References](#References)

## Introduction

Git configurations can be set for different locations, such as global and local. Global configs affect all repositories on a computer, and local configs affect only the current repo. If a configuration is set at both the global and the local level, the local level will override the global level -- this allows for project specific configurations to be set. There are also other levels, such as "system" and "worktree". However, this guide will focus on "global" and "local".

All Git configurations share the same scheme: `<SECTION>.<KEY>`. To add a configuration value, the command structure will always be `git config --global <KEY> "<VALUE>"`.

NB: For any `git config` command, omitting the location (`--global` or `--local`) will set the config locally. In other words, `git config` and `git config --local` are the same; a system-wide configuration *must* be specified with `git config --global`.

NB: For any `git config` command, switches can be passed in any order. As an example, in the command `git config --get --global`, `--get --global` works equally as well as `--global --get`.

## Setting up a new Git environment

When setting up a new Git environment, it is absolutely necessary to set the name and e-mail of the Git user -- Git will not allow a commit to be made until these two configurations are set.

To set the user's name system-wide, run the following command:

```
git config --global user.name <NAME>
```

To set the user's e-mail system-wide, run the following command:

```
git config --global user.email <E-MAIL_ADDRESS>
```

With these two configurations set, a user can now make commits to a repo, so long as the key is set within that repo.

## Viewing Git configurations

There are a few different ways to check the values of keys.

### Using "git-config-get" to get the value of a configuration

One way to confirm that the config has been set is to run `git config --get`, like in the following example:

```
git config --get user.name
```

This command will return only one value -- the highest level value of the key `user.name`. Therefore, if a key has been set at both the global and local levels, `git config --get` will return the local value. To use `git config --get` on a specific level (global/local), add the correct location switch to the command.

As an example, if both global and local values are set for the key `user.name`, return the global value with:

```
git config --get --global user.name
```

### Viewing the config files

All of Git's configurations are saved into files. For any repo, those files will be found in the `.git` directory. This includes the file that saves all of the local configs, located conveniently in the `.git/config` file.

By opening up this file in a text editor, it's possible to view all of the keys and values set at the local level. This will include origin and upstream URIs, which connect Git repos to other Git repos (often, repos found on GitHub or GitLab).

As an example, assuming the user's current working directory is the root of the project, the following command will output the contents of the `.git/config` file to the terminal's `STDOUT`:

```
cat .git/config
```

The global configs are also stored in a file, but that file (`.gitconfig`) is found in the user's home directory. Similarly, this file can also be displayed in the terminal with the following command:

```
cat ~/.gitconfig
```

### Using "git-config-list" to list all the configuration values

Another way to view Git configurations is to run the following command:

```
git config --list
```

`git config --list` will display all the keys and values set, both globally and locally. As before, to show only the global or local subset, pass the `--global` or `--local` switch (respectively).

In addition, as is true for other Git commands, `git config --list` can be piped into `cat`, in order to send the output to `STDOUT`. This can be achieved with the following command:

```
git config --list | cat
```

An additional switch worth noting is `--show-origin`, which allows the user to see in which file a configuration is set. To see a view of where each config value is set, run the following command:

```
git config --list --show-origin
```

Note that the `git config --list --show-origin` command does not require a `--global` or `--local` switch, as that would be redundant. The command works by showing all active configs, and where each is set. So, all the global configs will appear, unless there is a local override.

## Editing Git configurations

Viewing the Git configs is one thing, but what are some ways to edit them besides directly setting the values with `git config`?

### Using "git-config-edit" to edit Git configurations

To quickly edit Git's config files, run `git config --edit`, like in the following example:

```
git config --edit
```

This command will open up a concatenated file in the system's default editor, showing both the global and local configs. To see only one, pass the `--global` or `--local` switch. Passing one of the switches will directly open up the corresponding file in the system's text editor.

NB: Use the following command to see what editor is set by the system:

```
echo $EDITOR
```

### Editing the config file

However, the files can also be opened manually using a text editor and edited in the same way.

To open the config file local to a repo (`.git/config`), run the following command in the root directory of the project:

```
vim .git/config
```

The same works for the global file (`~/.gitconfig`) -- run the following command to open the file containing the global configs:

```
vim ~/.gitconfig
```

Saving changes in these files will update the keys and values that are shown using the `git config --list` command.

## A key can have multiple values

In the same way that a key can have a global value and a local value, a key can have multiple values within a location. A value can be added to the same key without overwriting the first value by running `git config --add <SECTION>.<KEYNAME> <VALUE>`. See the following example:

```
git config --add user.email user1@example.com
git config --add user.email user2@example.com
```

To see all the values for `user.email`, run the following command:

```
git config --get-regexp email
```

Another way to get the same values, though less optimal, is to use:

```
git config --list | grep email
```

If a user adds a new value with `git config --add user.email user3@example.com`, and then runs `git config --get user.email`, only `user3@example.com` would return. But, checking the `.git/config` file with `git config --list` shows that all 3 user e-mails are set as values for the same key. Another way to show all the keys for a value is to run the following command:

```
git config --get-all user.email
```

Configuration keys are not unique, and can be set with multiple values. This makes sense in the context of "global vs local".

To unset a value, run the command: `git config --unset <SECTION>.<KEY>`. This only works when a single value is set to a key -- using `--unset` when there are multiple values will return an error: `warning: <SECTION>.<KEY> has multiple values`. This prevents the accidental unset of a key with multiple values, as Git would be unsure of which value to unset.

To unset all values for a key, run the command: `git config --unset-all <SECTION>.<KEY>`.

To unset all values for a section, run the command: `git config --remove-section <SECTION>`

Key values are grabbed in a specific order:

1. Values are grabbed from the most specific location possible.
2. Newer values are preferred over older values.

For this reason, it may be advisable to work on configs by directly editing the files, using either `git config --edit` or `vim .git/config`. Editing configs this way allows a more comprehensive view of the states of all sections and keys, and can make it easier to navigate and modify configs.

## Signing commits with GPG keys

As seen above, it's very easy to set the user's name and e-mail to any value. What stops a user from impersonating another and sharing commits, as if they were someone else?

Many users like signing their commits with a GPG key, in order to provide a level of "verification" that the commit came from the expected user. The public key of the GPG keypair can be upload to a site such as GitHub or GitLab. If the user making the commit has the public key that corresponds to the private key that signed the commit, a "verified" badge will appear next to the commit in the commit history, allowing others to have some degree of confidence that the commit is legitimate.

In order to set up GPG signing for Git commits, some steps must be taken. First, Git needs to be aware that the user wants to sign commits with a GPG key. This can be achieved with the following command:

```
git config --global commit.gpgsign true
```

Once `commit.gpgsign` is enabled, Git needs to know which key to use to sign the commits. As with the `commit.gpgsign` configuration, this can be done either globally or locally.

There are a few ways to direct Git to the right key. One way is to tell Git to use the value set by `user.email` to check the GPG keyring for a corresponding key -- this assumes a 1 to 1 ratio of keys to that e-mail address. Another way is to point Git to the specific key to be used.

To direct Git to check the keyring based on the value of `user.email`, run the following command:

```
git config --global user.signing key
```

To specify a unique key to Git, run the following command:

```
git config --global user.signingkey <KEY_ID>
```

When specifying a key ID, both short and long key IDs will work, so long as it's unique (as compared to other keys in the keyring).

## Example configurations in the Git config files

The following are some examples of useful configurations for both the `~/.gitconfig` (global) and `.git/config` (local) files.

### Example global config file

In a global `~/.gitconfig` file, a user may want the following configs set:

```
[user]
    name = <NAME>
    email = <E-MAIL>
    signing = key
[commit]
    gpgsign = true
```

As explained in previous sections, it is often a good idea to have a global `name` and `email` set. If a specific repo requires a different `name` and `email`, that can be set in the repo's local config file (`.git/config`). In addition, users who use GPG keys to sign commits may want to specify that in the global config file. If a specific repo doesn't require commits to be signed, then `commit.gpgsign` can be set to `false` at the local level.

#### Global configs for "git-difftool"

As described in the [section on "git difftool"](git-difftool.md#Configuring-the-default-tool-for-git-difftool), the user may also want to specify global defaults for `git difftool`:

```
[diff]
    tool = nvimdiff
[merge]
    tool = nvimdiff
[difftool]
    prompt = false
```

For more information on the function of these configs, refer to the section on ["git-difftool"](git-difftool.md#Configuring-the-default-tool-for-git-difftool).

#### Global configs for Git aliases

In addition, the global `~/.gitconfig` file is the best place to specify any Git aliases the user may want access to across all repos:

```
[alias]
    lg = lg1
    lg1 = log --all --decorate --graph --abbrev-commit --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(white)- %an%C(reset)%C(auto)%d%C(reset)'
    lg2 = log --all --decorate --graph --abbrev-commit --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(auto)%d%C(reset)%n''          %C(white)%s%C(reset) %C(white)- %an%C(reset)'
    rebase-int = rebase --committer-date-is-author-date -i
    status-remote = rev-list --left-right --count
    s = status
```

This is an example of a few Git aliases, which are explained in more detail in the proper sections:

- ["git-log"](git-log.md#Useful-aliases-for-git-log)
- ["Interactive rebase"](interactive-rebase.md#An-early-note-on-dates-when-rebasing-interactively)
- ["git-status" and "git-rev-list"](git-status.md#Using-an-alias-with-git-status)

### Example local config file

In a local `.git/config` file, a user have some of the following configs set:

```
[user]
    name = <NAME>
    email = <E-MAIL>
[commit]
    gpgsign = false
[remote "origin"]
    url = <URL>
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
```

To explain the above:

- A user may want a specific and different `name` and `email` for a local repo, as compared to the global setting.
- A user may have a local repo that does not require signed commits; so, `commit.gpgsign` is set to `false`.
- The `remotes` for a repo will be set in the `.git/config` file for that repo. They follow a specific format:
    - The `remote` is given a name.
    - The `remote` has a value for `url`, which can take the form of the following:
        - A GitHub/GitLab URI: `https://github.com/<USER>/<NAME_OF_REPO>`
        - A GitHub/GitLab URI, in `ssh`/`scp` format: `git@github.com:<USER>/<NAME_OF_REPO>`
        - A path to a local directory: `../<NAME_OF_OTHER_REPO>`
        - A path to a remote directory, in `ssh`/`scp` format: `<USER>@<DOMAIN>:/<PATH_TO_REPO>`
    - The `remote` also has a value for `fetch`, which points to the "path to the refs" for that remote (ex. `refs/remotes/origin`).
    - For more information, refer to the section on [working with remote repositories in Git](git-remote.md#Remote-doesn't-have-to-be-remote).
- When working with `remotes`, the different branches will be configured in this file. They also follow a specific format:
    - The branch name is specified.
    - The branch has a value for `remote`, which tells Git which `remote` to use when running `git push` and `git pull`.
    - The branch also has a value for `merge`, which tells Git which ref to use.
    - For more information, refer to the section on [working with remote repositories in Git](git-remote.md#Using-git-pull-to-update-the-current-state-of-a-local-branch).

### Other configs

Another config worth noting is the following:

```
[receive]
    denyCurrentBranch = ignore
```

A user cannot `git push` a branch to a remote that has the same branch currently checked out. This option ignores that default, and will allow users to `git push` a branch, even if it is the current branch on the `remote`. For more information, refer to the section on [working with remote repositories in Git](git-remote.md#Pushing-to-a-branch-that-is-currently-checked-out-on-the-remote).

## References

- [thePrimeagen - Everything You'll Need to Know About Git - Your First Git Repo](https://theprimeagen.github.io/fem-git/lessons/intro/your-first-git-repo)
    - Basic Git config review, and `user.name` and `user.email`
- [thePrimeagen - Everything You'll Need to Know About Git - Config](https://theprimeagen.github.io/fem-git/lessons/intro/config)
    - In depth Git config review, including "keys having multiple values"
- [Git SCM documentation - Getting Started - First-Time Git Setup](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)
    - Reference for `user.name` and `user.email` commands
- [StackOverflow - Where is the global git config data stored?](https://stackoverflow.com/questions/2114111/where-is-the-global-git-config-data-stored)
    - Reference for the path to the `~/.gitconfig` global config file
- [GitHub Documentation - Signing Commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
    - Reference for `commit.gpgsign` command
- [GitHub Documentation - Telling Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
    - Reference for `user.signingkey` command
- [StackOverflow - Configuring diff tool with .gitconfig](https://stackoverflow.com/questions/6412516/configuring-diff-tool-with-gitconfig)
    - Reference on how to configure a default `difftool`
- [StackOverflow - Visualizing branch topology in Git](https://stackoverflow.com/questions/1838873/visualizing-branch-topology-in-git/34467298#34467298)
    - Reference for useful `git log` aliases
