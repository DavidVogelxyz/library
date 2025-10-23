Configuring a server running Alpine Linux
=========================================

NB:

- On multiple occasions, this guide makes reference to the `service` command. Know that `service` functions the same way as `rc-service`.
- In addition, `rc-update add $SERVICE` is a shorter way to write `rc-update add $SERVICE default`.
    - Both commands will add the `$SERVICE` to the "default" run level.
- When using a tty on Alpine Linux, "Ctrl-Alt-Del" reboots the computer.

Introduction
------------

This guide will assist with perform basic server tasks on Alpine Linux, such as creating a new user, securing the SSH connection and adding some configuration files for quality-of-life adjustments.

Table of contents
-----------------

- [Introduction](#introduction)
- [Updating packages](#updating-packages)
- [Enabling SSH](#enabling-ssh)
    - [SSH on a Proxmox container](#ssh-on-a-proxmox-container)
- [Creating a new user](#creating-a-new-user)
- [Securing SSH](#securing-ssh)
- [Adding configuration files](#adding-configuration-files)
    - [Adding configuration files to new user](#adding-configuration-files-to-new-user)
    - [Adding configuration files to root user](#adding-configuration-files-to-root-user)
- [Checking user access logs](#checking-user-access-logs)

Updating packages
-----------------

To begin, log in to the server. Next, update the package repositories and install some immediate essentials:

```
apk update && apk add openssh-server shadow sudo vim
```

Enabling SSH
------------

At this point, the guide splits depending on whether Alpine Linux is being installed using a Proxmox Alpine container or a VM from a cloud service provider (CSP). This is because Proxmox Alpine containers do not have SSH enabled by default, while VMs from CSPs do. For a Proxmox container, start at [SSH on a Proxmox container](#ssh-on-a-proxmox-container); for a VM from a CSP, start at [creating a new user](#creating-a-new-user).

### SSH on a Proxmox container

Configure `ssh` by editing the "/etc/ssh/sshd_config" file. Most notably, "PermitRootLogin" needs to be set as "yes", as the Alpine Linux container doesn't have a user created by default.

```
vim /etc/ssh/sshd_config
```

At least on the Alpine Linux container for Proxmox, when `ssh` is installed, `sshd` is not automatically configured to run as a service. This can be a problem on a container restart, if the service is expected to be running on reboot.

To ensure that `ssh` lockouts do not occur, run the following command:

```
service sshd start && rc-update add sshd
```

In order to test `ssh` login, get the IP address of the VM container.

```
ip a
```

Now, log in to the Alpine Linux server as the root user, using SSH.

Creating a new user
-------------------

If the root user was used to log into the server, the next step is to create a new user to administer and manager the server. It is highly advisable that the root user is not used for this purpose.

A new user can be created with the `adduser` command. By adding `wheel` after the username, the new user will be added to the "wheel" group, allowing the user to execute commands as if they were the root user.

Create a new user account with the following command:

```
adduser $USERNAME wheel
```

Alternatively, if the `shadow` package has been installed, a new user can be created with the `useradd` command. Add the user to the "wheel" group with `-G wheel`, and use `-m` to give that user a home directory. The `-s /bin/ash` is also useful in order to guarantee that the user's default shell is `ash`.

The `useradd` can be utilized with the following command:

```
useradd -G wheel -s /bin/ash -m $USERNAME
```

Once the user has been created, set the user's password with the following command:

```
passwd $USERNAME
```

On Alpine, just like on Arch Linux, it is important to check the "/etc/sudoers" file and confirm that users in the "wheel" group are able to run commands with `sudo`. Use the following `sed` command to confirm that this is the case:

```
sed -i "s/^# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g" /etc/sudoers
```

Now, log out, and log back in using the new user account using SSH. Once logged in as the new user, it's time to secure the SSH connection.

Securing SSH
------------

A crucial step in the setup process is to properly configure SSH login. There are two main components to this:

- disabling "root login via SSH"
- disabling "password-based authentication"

By disabling both of those options, the server will be limited to only accepting SSH connections from non-root users, and those users will not be able to login using a password alone. Therefore, a public/private keypair will need to be configured to allow the user to login securely.

Create a key on the computer that will access the server by using the following command:

```
ssh-keygen -t rsa -b 4096
```

Note that the above command works both with Linux machines and Windows computers (through PowerShell). During keyfile generation, the command will prompt the user for a password. It is an acceptable practice to have the keyfile password be the same as the user's password on the server.

Next, the key needs to be copied over to the server and added to the list of keys authorized for SSH. On a Linux machine, this is most easily accomplished through the use of the following command. Note that `$USERNAME` should be the name of the user created in the previous step, and `$DOMAIN` can be either the domain of the Alpine Linux server, its hostname, or its IP address.

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

Note: if using the root user to login (not recommended), "PermitRootLogin" should be set to "prohibit-password"

Once these lines have been edited, the `sshd` service requires a restart for the changes to take effect. This can be accompished with the following command:

```
service sshd restart
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

Adding configuration files
--------------------------

First, update packages repositiories again, upgrade installed packages, and install some additional packages:

```
sudo apk update && sudo apk upgrade && sudo apk add curl git htop tmux tree ufw
```

### Adding configuration files to new user

Next, create some directories that will store some of the configuration files:

```
mkdir -pv ~/.config/shell ~/.local/src
```

Clone a few GitHub repositories with already-created configuration files.

```
git clone https://github.com/davidvogelxyz/dotfiles ~/.dotfiles && ln -s ../../.dotfiles ~/.local/src/dotfiles && git clone https://github.com/davidvogelxyz/vim ~/.local/src/vim
```

Symbolically link the `vim` configurations to the home directory of the active user.

```
ln -s .local/src/vim ~/.vim
```

Symbolically link the "aliasrc" file for Alpine Linux into the newly created "~/.config/shell" directory.

```
ln -s ../../.dotfiles/.config/shell/aliasrc-alpine ~/.config/shell/aliasrc && source ~/.config/shell/aliasrc
```

To the "~/.ashrc" file, add the line: `source ~/.config/shell/aliasrc`.

```
echo -e "\\nsource ~/.config/shell/aliasrc" >> ~/.ashrc
```

Another change that, while minimal, can be helpful, is to change the "/etc/profile" file so that the username shows along with the hostname. This can easily be accomplished with the following `sed` command, which also sources the "/etc/profile" file after to update the shell:

```
sudo sed -i "s/PS1='\\\h/PS1='\\\u@\\\h/g" /etc/profile && source /etc/profile
```

### Adding configuration files to root user

It can be helpful to apply the same configuration changes to the root user as well. Examples of this include using `sudo vim`. If the Vim configuration files don't exist in the same way as for the new user, then the root account's Vim will be the default, and won't have the "quality of life" improvements that the new user's Vim has.

To make this easy, the following commands can all be run as a single set of commands, as can be seen in the following code block:

```
mkdir -pv ~/.config/shell ~/.local/src
git clone https://github.com/davidvogelxyz/dotfiles ~/.dotfiles && ln -s ../../.dotfiles ~/.local/src/dotfiles && git clone https://github.com/davidvogelxyz/vim ~/.local/src/vim
ln -s .local/src/vim ~/.vim
ln -s ../../.dotfiles/.config/shell/aliasrc-alpine ~/.config/shell/aliasrc && source ~/.config/shell/aliasrc
echo -e "\\nsource ~/.config/shell/aliasrc" >> ~/.ashrc
```

Since the root user is the active user, this is also the best time to update the "/etc/profile" file with a new line. To the "/etc/profile" file, add the line: `source ~/.ashrc`.

```
echo -e "\nsource ~/.ashrc" >> /etc/profile && source /etc/profile
```

Since the `sed` command was run using `sudo`, and already modified the "/etc/profile" for all users, it is not necessary to run this command again. However, if using the above code block for the new user as well (or, if "new user creation" was skipped entirely), do remember to run the `sed` at this time (no `sudo` required).

Checking user access logs
-------------------------

While the configurations have all been pushed, and the server should be largely secured (sans `ufw`), it is also useful to know how to verify that no unintended logins have occurred while performing the initial configuration.

Checking the user logins on Alpine is a bit more obtuse than it is on a Debian or Arch Linux machine. It requires that the user reference the actual log files found in "/var/log" -- the files that contain the relevant information are the "messages" files. As an example, consider a machine that has two log files: an older "messages.0" file, and a more recent "messages" file. A simple way to manage the information found in those files is to run the following command:

```
grep "Accepted" /var/log/messages.0 >> logfile && grep "Accepted" /var/log/messages >> logfile
```

Now, there exists a "logfile" in the current working directory that will list out all of the connections accepted by the server.
