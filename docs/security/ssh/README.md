SSH
===

[Back to "Docs and guides"](../../README.md)

Introduction
------------

SSH (Secure Shell) is an essential tool for any user connecting to other machines.

This guide will go through some of the basics for working with SSH. In addition, it will also review some more "advanced concepts", such as working with a `.ssh/config` file and using an `ssh-agent` to store unlocked private keys.

Table of contents
-----------------

- [Introduction](#introduction)
- [Terms and Definitions](#terms-and-definitions)
- [Creating SSH Keys](create-ssh-keys.md)
- [Uploading the Public Key to Another Machine](upload-public-key-to-another-machine.md)
- [Restricting SSH Connections to Only Users with Registered Private Keys](restrict-connections-to-only-users-with-private-keys.md)
- [Utilizing the SSH Config File to Simplify the Login Process](ssh-config.md)
- [Utilizing the "SSH-Agent" to Store Unlocked Private Keys](ssh-agent.md)
- [Working with the "known_hosts" file](known-hosts.md)
- [Working with the "authorized_keys" file](authorized-keys.md)

Terms and Definitions
---------------------

- **Keypair**: A term to refer to a public key and its corresponding private key.

- **Pubkey**: A shorthand for "public key". This is the key that will be shared with other machines, in order to authenticate the user in the future.

- **Private key**: Also known as a "secret key". This is the key that must be kept safe, as a private key (without a passphrase) can be used to log into any machine for which the pubkey has already been shared.

- **Client machine**: This is a term to reference "the machine from which the user is connecting, via SSH".

- **Target machine**: This is a term to reference "the machine that the user is logging into, via SSH". While the target machine is often a server, it isn't always.
