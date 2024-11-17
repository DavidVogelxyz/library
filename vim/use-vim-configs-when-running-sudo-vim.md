# Using Vim Configs When Running "sudo vim"

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Configure Vim to use the user's configs](#configure-vim-to-use-the-users-configs)
- [References](#references)

## Introduction

Sometimes, a user will set up Vim configs for their local user, but the configs will not be set up for the root user.

Perhaps the user is using a shared computer or server, and the root user's Vim configs can't be changed. Or, perhaps the user only set up the Vim configs for their local user, but the root user's Vim configs weren't changed. Maybe, the user doesn't want to install two separate versions of the plugins since `sudo vim` is seldom ran.

In any of these cases, the user can direct the `sudo vim` account to use their own Vim configs.

## Configure Vim to use the user's configs

First, the user should have the environment variable `$EDITOR` set to the editor they want to use when using `sudo $EDITOR`. This can be achieved with the following command:

```
export EDITOR="vim"
```

Obviously, if the user would prefer to use `nvim`, or a different editor, substitute the preferred editor in place of `vim`.

Then, the user will run the following command:

```
sudo -e <PATH>
```

The `-e` switch directs `sudo` to open the path, designated by `<PATH>`, in whatever editor is set by `$EDITOR`. However, `sudo` will use the user's configs, but elevate that instance to a root-owned process.

## References

- [StackOverflow - Vim - vimrc settings for user dont work for root](https://stackoverflow.com/questions/28235592/vimrc-settings-for-user-dont-work-for-root)
    - Reference for using Vim configs when running `sudo vim`
