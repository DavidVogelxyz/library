# Uploading the Public Key to Another Machine

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Linux client machines](#linux-client-machines)
- [Windows client machines](#windows-client-machines)
- [Logging in with a private key](#logging-in-with-a-private-key)

## Introduction

Once a public/private keypair has been created, the next step is to get the pubkey onto the target machine.

A simple way to do this that works with all types of client and target machines is to copy the pubkey and paste it directly into the user's `.ssh/authorized_keys` file. However, there are more secure ways to append a pubkey that take advantage of the SSH protocol. At least on Linux, a built-in command exists for appending pubkeys to the target machine.

The commands differ depending on whether the client machine is a Linux or a Windows machine. The following instructions explain how to upload the pubkey on each type of client machine.

## Linux client machines

To get a pubkey onto a target machine, a Linux user would run the following command:

```
ssh-copy-id -i <PATH_TO_PUBLIC_KEYFILE> <USER>@<DOMAIN>
```

Once the connection is established, the user will need to enter the password for `<USER>` in order to append the public key to the `.ssh/authorized_keys` file. Therefore, if the target machine is already secured such that only users with a registered keypair can authenticate, and the user doesn't already have a public key registered, then this option will not be available to the user.

Even though the command will accept the private key's path, the corresponding pubkey is what will be copied over to the target machine. This assumes the pubkey is found in the same directory with the same name, which should be the case if the keys were previously generated with `ssh-keygen`.

Note that the variable `<DOMAIN>` can either be the hostname, the domain, or the IP address of the machine.

## Windows client machines

To get a pubkey onto a target machine, a Windows user would run the following command:

```
type $env:<USERPROFILE>\<PATH_TO_PUBLIC_KEYFILE> | ssh <USER>@<DOMAIN> "cat >> .ssh/authorized_keys"
```

Note that, as with a Linux client, the variable `<DOMAIN>` can either be the hostname, the domain, or the IP address of the machine.

## Logging in with a private key

To test that the pubkey copied correctly, attempt to log in using the private key:

```
ssh -i <PATH_TO_PRIVATE_KEYFILE> <USER>@<DOMAIN>
```

This should allow access to the target machine via SSH. If a passphrase was added to the keyfile, it is *that* passphrase that will be required during the sign-in attempt. If there was no passphrase set, access should be given without a prompt for a passphrase.
