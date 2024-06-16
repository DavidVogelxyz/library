# SSH

NB:

- This guide assumes that the "target computer" being set up is a Debian-based machine (Ubuntu included).
    - This is relevant when using the `systemctl` command.
    - Other init systems will have slightly different commands.
- The term "target computer" refers to the machine that `ssh` is connecting to. It can be a server, VM, or any other machine.

## Table of contents

- [Creating SSH keys](#creating-ssh-keys)
- [Changing the passphrase to an SSH key](#changing-the-passphrase-to-an-ssh-key)
- [Uploading the public key to another machine](#uploading-the-public-key-to-another-machine)
    - [Linux client machines](#linux-client-machines)
    - [Windows client machines](#windows-client-machines)
- [Securing SSH connections](#securing-ssh-connections)

## Creating SSH keys

First, create a new public/private keypair specific to the target computer that's being working on. On a Linux client machine, creating a new SSH key is as simple as using the following command. In Windows, it *should* be possible to open a PowerShell terminal and use the same command to create a new SSH key.

```
ssh-keygen -t rsa -b 4096
```

The two options in this command set the encryption bits (`-b`) and the encryption type (`-t`).

For ease of access (etc), it is not *required* to add a passphrase to the keyfile. That being said, anyone with this keyfile would be able to access the target computer, so use best judgment to decide how secure the target computer needs to be.

## Changing the passphrase to an SSH key

The private key's passphrase can always be changed with the following:

```
ssh-keygen -p -f /PATH/TO/$PRIVATE_KEYFILE
```

## Uploading the public key to another machine

Once a public/private keypair has been created, the next step is to get the *public* key ("pubkey") registered onto the computer to be logged into.

The commands differ depending on whether the client machine (the computer the connection is made *from*) is a Linux or a Windows machine. The following instructions explain how to upload the public key on each type of client machine.

### Linux client machines

To get the pubkey onto the target computer, a Linux user would use the following command:

```
ssh-copy-id -i $PATH_TO_FILE $USER@$DOMAIN
```

Even though the command seems to prefer the private key, the public key is what will be copied over to the target computer. This assumes the public key is found in the same directory with the same name, which should be the case if the keys were previously generated with `ssh-keygen`.

Note: the variable `$DOMAIN` can either be the hostname, the domain, or the IP address of the machine.

### Windows client machines

To get the pubkey onto the target computer, a Windows user would use the following command:

```
type $env:USERPROFILE\PATH\TO\$PUBLIC_KEYFILE | ssh $USER@$DOMAIN "cat >> .ssh/authorized_keys"
```

Note: as with a Linux client, the variable `$DOMAIN` can either be the hostname, the domain, or the IP address of the machine.

## Securing SSH connections

To test that the pubkey copied correctly, attempt to log in using the keyfile:

```
ssh -i \PATH\TO\$PUBLIC_KEYFILE $USER@$DOMAIN
```

This should allow access to the target computer via SSH. If a passphrase was added to the keyfile, it is *that* passphrase that will be required during the sign-in attempt. If there was no passphrase set, access should be given without a prompt for a passphrase.

Now that it is confirmed that the keyfile works, the next step is to remove "access via SSH without providing a keyfile." Do the following:

- SSH into the target computer.
- Open up the '/etc/ssh/sshd_config' file using `sudo` and a text editor of choice (nano/vim/whatever else).
- From here, search the file for a line that says "PasswordAuthentication." Likely, this is around line 58. Uncomment that line and change the option to "no".
- It is also recommended to confirm that the "KdbInteractiveAuthentication" option is also set to "no". This line is usually found just below the "PasswordAuthentication" option.
- It is also highly advisable to remove the option for users to log into the root account using SSH. This option is often around line 33 -- "PermitRootLogin." Again, change this option to "no".
- Save the file.
- Now, within the target computer, run the command `sudo systemctl restart sshd`.
- Test the functionality by logging out of the target computer, and attempting to SSH into both the root user and the working user (both without a keyfile). These should fail, on the basis of not providing a public key.
- To SSH in using the public key, using the following command: `ssh -i \PATH\TO\KEYFILE $USER@$DOMAIN`. This should allow access to the target computer.

From here, a final step would be to configure the client to have an alias for that `ssh -i` command. For Linux users, this would be an adjustment to the "aliasrc" file; for Windows users, this would mean editing the "PowershellProfile.ps1" file.
