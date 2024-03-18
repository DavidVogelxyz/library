# SSH

NB: This guide assumes that the server/VM being set up is a Debian-based machine (Ubuntu included). This is relevant when using the `systemd` command. Other init systems will have slightly different commands.

## Table of contents

- [Creating SSH keys](#Creating-SSH-keys)
    - [Linux client machines](#Linux-client-machines)
    - [Windows client machines](#Windows-client-machines)
- [Securing SSH connections](#Securing-SSH-connections)

## Creating SSH keys

First, create a new public/private keypair specific to the server that's being working on. On a Linux client machine, creating a new SSH key is as simple as using the following command. In Windows, it *should* be possible to open a PowerShell terminal and use the same command to create a new SSH key.

```
ssh-keygen -b 4096 -t rsa
```

The two options in this command set the encryption bits (`-b`) and the encryption type (`-t`).

For ease of access (etc), it is not *required* to add a password to the keyfile. That being said, anyone with this keyfile would be able to access the server, so use best judgment to decide how secure the servers need to be. In the future, the private key's password can always be changed with the following:

```
ssh-keygen -p -f /PATH/TO/$PRIVATE_KEYFILE
```

### Linux client machines

To get the *public* key ("pubkey") onto the server, a Linux user would use the following command:

```
ssh-copy-id -i $PATH_TO_FILE $USER@$DOMAIN
```

Even though the command seems to prefer the private key, the public key is what will be copied over to the server. This assumes the public key is found in the same directory with the same name, which should be the case if the keys were previously generated with `ssh-keygen`. For "$DOMAIN", this can either be the hostname of the machine, or the IP address of the machine.

### Windows client machines

To get the *public* key ("pubkey") onto the server, a Windows user would use the following command:

```
type $env:USERPROFILE\PATH\TO\$PUBLIC_KEYFILE | ssh $USER@$IP_ADDRESS "cat >> .ssh/authorized_keys"
```

## Securing SSH connections

To test that the pubkey copied correctly, attempt to log in using the keyfile:

```
ssh -i \PATH\TO\$PUBLIC_KEYFILE $USER@$IP_ADDRESS
```

This should allow access to the server via SSH. If a password was added to the keyfile, it is *that* password that will be required during the sign-in attempt. If there was no password set, access should be given without a password prompt.

Now that it is confirmed that the keyfile works, the next step is to remove "access via SSH without providing a keyfile." Do the following:

- SSH into the server.
- Open up the '/etc/ssh/sshd_config' file using `sudo` and a text editor of choice (nano/vim/whatever else).
- From here, search the file for a line that says "PasswordAuthentication." Likely, this is around line 58. Uncomment that line and change the option to "no".
- It is also recommended to confirm that the "KdbInteractiveAuthentication" option is also set to "no". This line is usually found just below the "PasswordAuthentication" option.
- It is also highly advisable to remove the option for users to log into the root account using SSH. This option is often around line 33 -- "PermitRootLogin." Again, change this option to "no".
- Save the file.
- Now, within the server, run the command `sudo systemctl restart sshd`.
- Test the functionality by logging out of the server, and attempting to SSH into both the root user and the working user (both without a keyfile). These should fail, on the basis of not providing a public key.
- To SSH in using the public key, using the following command: `ssh -i \PATH\TO\KEYFILE $USER@$IP_ADDRESS`. This should allow access to the server.

From here, a final step would be to configure the client to have an alias for that `ssh -i` command. For Linux users, this would be an adjustment to the "aliasrc" file; for Windows users, this would mean editing the "PowershellProfile.ps1" file.
