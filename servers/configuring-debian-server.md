# Configuring a server running Debian

## Introduction

This guide will assist with perform basic server tasks on Debian, such as creating a new user, securing the SSH connection and adding some configuration files for quality-of-life adjustments.

## Table of contents

- [Introduction](#introduction)
- [Updating packages](#updating-packages)
- [Creating a new user](#creating-a-new-user)
- [Securing SSH](#securing-ssh)
- [Adding configuration files](#adding-configuration-files)
    - [Adding configuration files to new user](#adding-configuration-files-to-new-user)
    - [Adding configuration files to root user](#adding-configuration-files-to-root-user)
- [Checking user access logs](#checking-user-access-logs)

## Updating packages

To begin, log in to the server. Next, update the package repositories and install some immediate essentials:

```
apt update && apt install -y nala vim
```

## Creating a new user

If the root user was used to log into the server, the next step is to create a new user to administer and manager the server. It is highly advisable that the root user is not used for this purpose.

A new user can be created with the `useradd` command. Add the user to the "sudo" group with `-G sudo`, and use `-m` to give that user a home directory. The `-s /bin/bash` is also useful, because Debian tends to default the user to the `sh` shell, which has far less features than the standard `bash` shell most users are used to.

Create a new user account with the following command:

```
useradd -G sudo -s /bin/bash -m $USERNAME
```

Once the user has been created, set the user's password with the following command:

```
passwd $USERNAME
```

Now, log out, and log back in using the new user account using SSH. Once logged in as the new user, it's time to secure the SSH connection.

## Securing SSH

A crucial step in the setup process is to properly configure SSH login. There are two main components to this:

- disabling "root login via SSH"
- disabling "password-based authentication"

By disabling both of those options, the server will be limited to only accepting SSH connections from non-root users, and those users will not be able to login using a password alone. Therefore, a public/private keypair will need to be configured to allow the user to login securely.

Create a key on the computer that will access the server by using the following command:

```
ssh-keygen -t rsa -b 4096
```

Note that the above command works both with Linux machines and Windows computers (through PowerShell). During keyfile generation, the command will prompt the user for a password. It is an acceptable practice to have the keyfile password be the same as the user's password on the server.

Next, the key needs to be copied over to the server and added to the list of keys authorized for SSH. On a Linux machine, this is most easily accomplished through the use of the following command. Note that `$USERNAME` should be the name of the user created in the previous step, and `$DOMAIN` can be either the domain of the Debian server, its hostname, or its IP address.

```
ssh-copy-id -i /PATH/TO/SSH/KEY $USERNAME@$DOMAIN
```

The benefit of `ssh-copy-id` is that the key will automatically be entered into the '~/.ssh/authorized_keys' file for that particular user.

If `ssh-copy-id` is unavailable, the next best method to add the key to the 'authorized_keys' file is to use `scp` to securely copy the 'KEY.pub' file over to the VPN server. Then, the contents of that 'KEY.pub' file can be appended to the '~/.ssh/authorized_keys' file for that user.

From a Windows machine, neither `ssh-copy-id` not `scp` are readily available in PowerShell. The simplest method for a Windows admin is to follow [this guide](/security/ssh.md) on setting up SSH keys; specifically, reference the section on adding SSH public keys from a Windows client machine.

Once the public key is confirmed as existing within the '~/.ssh/authorized_keys' file, it is time to update the '/etc/ssh/sshd_config' file by disabling root access and password-based authentication. Since this is a file located in the "/etc" directory, open up the file using elevated privileges (eg. `sudo vim /etc/ssh/sshd_config`).

After opening the file, there are three lines of interest:

- PermitRootLogin
- PasswordAuthentication
- KbdInteractiveAuthentication

Whether by commenting out the already-existing lines and adding additional lines, or by replacing what's already there, those three lines should read:

```
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Once these lines have been edited, the `sshd` service requires a restart for the changes to take effect. This can be accompished with the following command:

```
sudo systemctl restart sshd
```

Confirm the changes by logging out and trying to log in again using the original `ssh $USER@$HOST` command. This should be refused with an error message of "Permission denied (publickey)." To connect using the keyfile that was added to the server, modify the command to read:

```
ssh -i /PATH/TO/THE/KEY.pub $USER@$HOST
```

This command should work, and will prompt the user for the password added during the key's creation.

Now, change the user's password, if necessary -- on a cloud server, it's often a good idea to change passwords after securing the SSH connection.

```
passwd
```

Also, if using a VM provided by a CSP, change the root password as well:

```
sudo passwd root
```

It is also worth checking the "/etc/hostname" and "/etc/hosts" files at this point. Confirm that the hostname is correct for the server, and confirm that it is found in "/etc/hosts" under a loopback address. If any adjustments were made to the "/etc/hostname" file, it is advisable to restart the server after and then to log back in.

## Adding configuration files

First, update packages repositiories again, upgrade installed packages, and install some additional packages:

```
sudo nala update && sudo nala upgrade -y && sudo nala install -y curl git htop rsync tmux tree ufw
```

### Adding configuration files to new user

Next, create some directories that will store some of the configuration files:

```
mkdir -pv ~/.cache/bash ~/.cache/zsh ~/.config/shell ~/.local/bin ~/.local/src
```

Clone a few GitHub repositories with already-created configuration files.

```
git clone https://github.com/davidvogelxyz/dotfiles ~/.dotfiles && ln -s ../../.dotfiles ~/.local/src/dotfiles && git clone https://github.com/davidvogelxyz/vim ~/.local/src/vim
```

Symbolically link the `vim` configurations to the home directory of the active user.

```
ln -s .local/src/vim ~/.vim
```

Symbolically link the "aliasrc" file for Debian into the newly created "~/.config/shell" directory.

```
ln -s ../../.dotfiles/.config/shell/aliasrc-debian ~/.config/shell/aliasrc && source ~/.config/shell/aliasrc
```

To the "~/.bashrc" file, add the line: `source ~/.config/shell/aliasrc`.

```
echo -e "\\nsource ~/.config/shell/aliasrc" >> ~/.bashrc
```

### Adding configuration files to root user

It can be helpful to apply the same configuration changes to the root user as well. Examples of this include using `sudo vim`. If the Vim configuration files don't exist in the same way as for the new user, then the root account's Vim will be the default, and won't have the "quality of life" improvements that the new user's Vim has.

To make this easy, the following commands can all be run as a single set of commands, as can be seen in the following code block:

```
mkdir -pv ~/.cache/bash ~/.cache/zsh ~/.config/shell ~/.local/bin ~/.local/src
git clone https://github.com/davidvogelxyz/dotfiles ~/.dotfiles && ln -s ../../.dotfiles ~/.local/src/dotfiles && git clone https://github.com/davidvogelxyz/vim ~/.local/src/vim
ln -s .local/src/vim ~/.vim
ln -s ../../.dotfiles/.config/shell/aliasrc-debian ~/.config/shell/aliasrc && source ~/.config/shell/aliasrc
echo -e "\\nsource ~/.config/shell/aliasrc" >> ~/.bashrc
```

## Checking user access logs

While the configurations have all been pushed, and the server should be largely secured (sans `ufw`), it is also useful to know how to verify that no unintended logins have occurred while performing the initial configuration.

On Debian, there are two commands that can be used to easily query the logs for attempted logins.

The first command is `last`, which can be run as any user, without requiring `sudo`. `last` will list out all the successful user logins on the machine, and will print the user that logged in, as well as the IP address of the machine that successfully logged in. As stated, `last` can be used by simply running the following:

```
last
```

The second command is `lastb`, which can only be run by a user with superuser privileges. Therefore, it must be run with `sudo` as a prefix. `lastb` will list out all of the *failed* login attempts to the machine, and will print the username used, as well as the IP address of the machine that attempted the login. As stated, `lastb` can be used by running the following command:

```
sudo lastb
```
