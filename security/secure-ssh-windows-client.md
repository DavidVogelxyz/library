# Securing SSH connections when using Windows client machines

NB: This guide assumes that the server/VM being set up is a Debian-based machine (Ubuntu included). This is relevant when using the `systemd` command. Other init systems will have slightly different commands.

First, create a new public/private keypair specific to the server that's being working on.

In Windows, it *should* be possible to open a PowerShell terminal and use the following command to do that:

```
ssh-keygen -b 4096 -t rsa
```

For ease of access (etc), it is not *required* to add a password to the keyfile. That being said, anyone with this keyfile would be able to access the server, so use best judgment to decide how secure the servers need to be. In the future, the private key's password can always be changed with the following:

```
ssh-keygen -p -f /PATH/TO/$PRIVATE_KEYFILE
```

To get the *public* key ("pubkey") onto the server, a Windows user would use the following command:

```
type $env:USERPROFILE\PATH\TO\$PUBLIC_KEYFILE | ssh $USER@$IP_ADDRESS "cat >> .ssh/authorized_keys"
```

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

From here, a final step would be to configure the PowershellProfile.ps1 file to include an alias for that `ssh -i` command.
