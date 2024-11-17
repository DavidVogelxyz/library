# Restricting SSH Connections to Only Users with Registered Private Keys

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Securing the "sshd_config" file](#securing-the-sshd_config-file)
- [Testing the configs](#testing-the-configs)

## Introduction

Note: this section of the guide assumes that the "target computer" is a Debian-based machine (Ubuntu included). This is because of the assumption that the target machine is running `systemd` as the init system. Other init systems will have slightly different commands.

Once the pubkey has been added to a user's `.ssh/authorized_keys` file, it is good practice to confirm that the key is working as intended. Once that verification step has been performed, the user can lock down the system such that only users with a registered keypair can access the machine.

## Securing the "sshd_config" file

Now that it is confirmed that the keyfile works, the next step is to remove "access via SSH without providing a keyfile." Do the following:

1. SSH into the target computer, using the private key.
1. Run `sudo <EDITOR> /etc/ssh/sshd_config` to open the `sshd_config` file in an editor of the user's choosing.
1. There are three options that should be reviewed:
    - `PermitRootLogin`
    - `PasswordAuthentication`
    - `KdbInteractiveAuthentication`
1. `PermitRootLogin` is an option that controls whether a user can log directly into the root user, via SSH.
    - For more systems and situations, it is best to set `PermitRootLogin no`.
        - Setting this option to `no` does not preclude a user from using `sudo`, or from switching users to `root` once logged in.
    - On systems where logging directly into the root user is required, set `PermitRootLogin prohibit-password` to require a registered private key.
1. `PasswordAuthentication` is an option that controls whether users can log in with passwords.
    - Set `PasswordAuthentication no` to require users to provide a registered private key to log in.
1. `KdbInteractiveAuthentication` is an option that allows for other authentication methods, such as "RSA SecurID", "RADIUS", and "PAM".
    - `Kdb` represents "keyboard".
    - For more systems and situations, it is best to set `KdbInteractiveAuthentication no`.
1. Save the file.
1. Now, while still logged into the target computer, run the command `sudo systemctl restart sshd`.

## Testing the configs

At this point, the changes made to the `/etc/ssh/sshd_config` file should be active. A "best practice" would be to open a second terminal window and attempt to connect via SSH into both the root user and the "working user". If no private key was provided, these connections should fail.

Then, using the second terminal window, connect to the target machine using the private key by running the following command:

```
ssh -i <PATH_TO_PRIVATE_KEYFILE> <USER>@<DOMAIN>
```

If the private key was secured with a passphrase, the user will have to enter it. In either case, this command should allow the user to log in successfully!

The reason why it's best to use a second terminal window for this purpose is because, in the case of user error, there's still an open and active SSH session from which the user can make corrections. If the user used the same terminal window, they would have to log out to attempt to log back in. If a mistake was made, it's entirely possible that the user has just locked themselves out of the target machine.
