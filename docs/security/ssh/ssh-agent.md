# Utilizing the "SSH-Agent" to Store Unlocked Private Keys

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Starting the "ssh-agent"](#starting-the-ssh-agent)
- [Adding identities](#adding-identities)
- [Listing stored identities](#listing-stored-identities)
- [Removing identities](#removing-identities)
- [Starting the "ssh-agent" on a non-DWM system](#starting-the-ssh-agent-on-a-non-dwm-system)
- [References](#references)

## Introduction

`ssh-agent` is an extremely powerful utility, especially in combination with a configured `.ssh/config` file. `ssh-agent` enables the user to enter the passphrase to a private key only once per session -- once they key has been unlocked, it is stored in the `ssh-agent` for the remainder of the session, allowing the user to reconnect without having to enter the passphrase again.

## Starting the "ssh-agent"

Depending on the system, there are a few different ways to start the `ssh-agent` process.

For example, imagine a user who runs `dwm`, the dynamic window manager. This user could add the following command to their `.xinitrc` file, so it runs once during startup:

```
ssh-agent dwm
```

In contrast, for a user who runs the `zsh` shell and wants the `ssh-agent` to run on a server, they would run the following command:

```
exec ssh-agent zsh
```

However, this user would be advised not to put this command in their `.zshrc` file, because the `.zshrc` file is sourced every time a new `zsh` shell is opened. This could very easily create a situation where there are multiple instances of `ssh-agent` running at the same time.

## Adding identities

Once the `ssh-agent` is running, adding keys is fairly simple.

To add an identity, the user would run the following command:

```
ssh-add <PATH_TO_PRIVATE_KEYFILE>
```

At this point, the user will be prompted to enter the passphrase for the key. However, once the key is added, anytime the key is used for a connection, the `ssh-agent` will use the key stored in the agent and will not require the passphrase to be entered again.

When the `.ssh/config` file is configured with `AddKeysToAgent=yes`, any connection made that utilizes a private key will automically add the key to the `ssh-agent`.

## Listing stored identities

To list out the identities stored in the `ssh-agent` in a "shorthand" format, run the following command:

```
ssh-add -l
```

To list out the identities stored in the `ssh-agent` in a "longhand" format, run the following command:

```
ssh-add -L
```

It's worth noting that, in both the case of `-l` and `-L`, the key's comment shows up next to the pubkey. When the user has multiple keys for multiple target machines, having keys with same `<USER>@<HOSTNAME>` comment doesn't help to discern which identities are loaded into the `ssh-agent`. Consider changing the key's comment with `ssh-keygen -c -f <PATH_TO_PRIVATE_KEYFILE>` in order to better identify keys. For more information on setting and changing comments, refer to the section on [creating SSH keys](create-ssh-keys.md#creating-a-ssh-key).

## Removing identities

To remove an identity from the `ssh-agent`, run the following command:

```
ssh-add -d <PATH_TO_PRIVATE_KEYFILE>
```

To remove all identities from the `ssh-agent`, run the following command:

```
ssh-add -D
```

## Starting the "ssh-agent" on a non-DWM system

Add the following to either the `.bashrc` or `.zshrc` file:

```
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env
```

## References

- [Arch Wiki - SSH keys - SSH agents](https://wiki.archlinux.org/title/SSH_keys#SSH_agents)
    - The Arch Wiki page on `ssh-agent`
- [GitHub - LukeSmithxyz - voidrice - Issue #417](https://github.com/LukeSmithxyz/voidrice/issues/417)
    - GitHub issue referencing the difference between `ssh-agent dwm` and `ssh-agent`
- [GitHub - Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
    - Reference for starting the `ssh-agent` on a machine that isn't running `dwm`
- [Superuser - How to avoid being asked to enter passphrase for key when I'm doing SSH](https://superuser.com/questions/988185/how-to-avoid-being-asked-enter-passphrase-for-key-when-im-doing-ssh-operation)
    - Reference for running the `ssh-agent`
- [StackExchange - How can I run ssh-add automatically, without a password prompt?](https://unix.stackexchange.com/questions/90853/how-can-i-run-ssh-add-automatically-without-a-password-prompt)
    - Another reference for running the `ssh-agent` automatically
- [StackOverflow - Start ssh-agent on login](https://stackoverflow.com/questions/18880024/start-ssh-agent-on-login)
    - Another reference for running the `ssh-agent` automatically
- [GitHub docs - Auto-launching ssh-agent on Git for Windows](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows)
    - Reference for the code snippet for the `~/.ssh/agent.env` file
