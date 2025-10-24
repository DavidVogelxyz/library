Getting started with SSH certificate authorities
================================================

[Back to the home page](README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Thought experiment](#thought-experiment)

Introduction
------------

A great place to start with understanding the relationship between SSH and the CA is to view the `~/.ssh/authorized_keys` file of a host that is configured to accept SSH keys to authenticate logins. The `authorized_keys` file is a list of pubkeys that are permitted to log into the host.

Normally, the `authorized_keys` file will look something like the following:

```
ssh-ed25519 <PUBKEY> <COMMENT>
ssh-ed25519 <PUBKEY> user1@host1
ssh-ed25519 <PUBKEY> user1@host2
ssh-ed25519 <PUBKEY> user2@host2
```

Thought experiment
------------------

Let's imagine that the host is called `mediaserver`, and the user account on the host is named `admin`. When a user runs `ssh admin@mediaserver`, the authentication request will be sent to `mediaserver` and checked against `admin`'s `authorized_keys` file. If the request contains a pubkey that can be found within the `authorized_keys` file, then the user submitting the request will be asked to sign off using the corresponding private key.

If the user passes that test, then they can be admitted to the host. If the user doesn't have the private key, then they will be rejected from login.

SSH certificate authorities operate similarly, but solve many problems of traditional SSH pubkey management.

One key issue that they resolve is the headache involved with managing the `authorized_keys` file. Every user that wants to sign in to the host must have a pubkey entry in the `authorized_keys` file. This means that an admin has to create that entry on any and all hosts for which a pubkey should have access.

Another problem that SSH certificate authorities resolve is the revocation of keys. In order to revoke access for a user, their pubkey must be removed (or commented out) from the `authorized_keys` file. When an administrator is managing only one host, or one login account, this may not appear to be a problem. But, when an administrator has to manage a site full of various hosts and user accounts, this can be extremely tedious. This is also an effort that can be prone to mistakes.

Now, let's look at the `authorized_keys` file for `admin@mediaserver`, when using a CA and configured to have many users logging into the host:

```
ssh-ed25519 <PUBKEY> master_site_key
```

Now, you may be saying to yourself, there's no way that the `authorized_keys` file has only one entry. You may ask yourself "how can that possibly manage authorization for multiple users?"

The best part? That pubkey isn't actually required! The `authorized_keys` file can be empty, and users can still authenticate to the host.

*That* is the magic of a SSH certificate authority.

[Next page](ssh-ca_user-point-of-view.md)
