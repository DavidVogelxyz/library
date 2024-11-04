# Creating SSH Keys

[Back to the home page](README.md)

## Table of contents

- [Introduction](#Introduction)
- [Creating a SSH key](#Creating-a-SSH-key)
    - [Ed25519](#Ed25519)
    - [RSA](#RSA)
- [References](#References)

## Introduction

While not mandatory, securing SSH connections with SSH keypairs is heavily suggested.

SSH keypairs make systems much more resistant to brute-force password attacks. This is because a correctly configured system using keypairs for authentication will not allow a user to connect without providing a private key that corresponds to a pubkey listed in the user's `.ssh/authorized_keys` file. If an attacker wanted to compromise a system using SSH keypairs, then they would need to brute-force private keys, which is far more difficult than brute-forcing a password.

One suggestion that could be considered "best practice" is to create a new keypair for every client/server connection. While this creates more overhead for the user when it comes to managing keys, it also make systems more secure -- if a key was ever compromised, it's far easier to change out one keypair for one system, than one keypair that has access to many systems.

Another best practice is to secure private keys with passphrases:

- Passphrases can be changed at any time with the `ssh-keygen -p -f <PATH_TO_PRIVATE_KEY>` command.
- While it may feel more convenient to use keys without passphrases, as it allows the user to sign in without giving any input to the system, it is not advisable.
    - Anyone with that insecure private key can access any system that has the corresponding pubkey in the `authorized_keys` file.
    - In addition, there are ways to achieve both the security of a passphrase, with the convenience of a key that doesn't require a passphrase for every sign in.

## Creating a SSH key

The `ssh-keygen` command allows a user to create a new public/private keypair.

The most basic key creation command will take the form of:

```
ssh-keygen -t <ALGORITHM>
```

The `-t` switch dictates which type of encryption algorithm to use when creating the keypair.

In addition, the key's comment can also be specified during key creation. To do this, run the following command:

```
ssh-keygen -t <ALGORITHM> -C "<COMMENT>"
```

By default, the comment takes the form of `<USER>@<HOSTNAME>`, and is especially relevant for the server receiving the pubkey for authentication. When multiple pubkeys have been appended to the `authorized_keys` file, the comment allows the server an easy way to identify which key came from which user. However, as explained in the section on [listing identities stored in the "ssh-agent"](ssh-agent.md#Listing-stored-identities), a user may want to change the comment for easier identification of their own keys. Just like with passphrases, comments can be changed at any time with the `ssh-keygen -c -f <PATH_TO_PRIVATE_KEY>` command.

### Ed25519

Ed25519 is a relatively recent encryption algorithm that uses a combination of SHA-512 and elliptic curves. While Ed25519 can create smaller keyfiles that are more secure than RSA-2048, not all systems accept Ed25519 keys. As of 2024, it is generally considered "best practice" to create keypairs with the Ed25519 algorithm.

To create a keypair using the Ed25519 algorithm, run the following command:

```
ssh-keygen -t ed25519
```

### RSA

RSA is a much older encryption algorithm, created in 1977.

RSA is still considered very secure when using an appropriate number of bits. The minimum recommended size is 2048 bits, but it's trivial to create keys using 4096 bits, which provides much greater security.

To create a keypair using the RSA algorithm, run the following command:

```
ssh-keygen -t rsa -b 4096
```

The two switches in this command set the encryption algorithm (`-t`) and the bits of encryption (`-b`).

## References

- [Arch Wiki - SSH keys - comments](https://wiki.archlinux.org/title/SSH_keys#Generating_an_SSH_key_pair)
    - The Arch Wiki page on comments for SSH keys
- [Superuser - How can I change the comment field of an RSA key (SSH)?](https://superuser.com/questions/361764/how-can-i-change-the-comment-field-of-an-rsa-key-ssh)
    - Reference for changing the comment field for a SSH key
