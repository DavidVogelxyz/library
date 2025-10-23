Utilizing a SSH Config File to Simplify the Login Process
=========================================================

[Back to the home page](README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [General configuration](#general-configuration)
- [Specific entries](#specific-entries)
- [Adding multiple entries for the same host](#adding-multiple-entries-for-the-same-host)
- [References](#references)

Introduction
------------

Once a user starts using key-based authentication, they may build a collection of various keypairs to different target machines. At some point, it may become cumbersome to type `ssh -i <PATH_TO_PRIVATE_KEYFILE> <USER>@<DOMAIN>` each time the user wants to login. One creative solution is to create aliases for each connection -- but, in its own way, this can become annoying to maintain.

Fortunately, SSH has a convenient solution for managing identities and connection profiles -- the `.ssh/config` file!

Using a `.ssh/config` file, a user can create entries and direct SSH on how to handle different connections. While the `.ssh/config` doesn't exist in the `.ssh` directory by default, it's trivially simple to create the file and begin adding entries.

General configuration
---------------------

It is best practice to add the following three lines to the beginning of the `.ssh/config` file:

```
Host *
    IdentitiesOnly=yes
    AddKeysToAgent=yes
    HashKnownHosts=yes
```

To explain in more detail:

- `Host *` directs SSH to use these settings for all hosts enumerated in the file.
- `IdentitiesOnly=yes` directs SSH to only use identities for the entry in which they're found.
    - In other words, without this setting, SSH will try *all* identities for any target machine that *doesn't* have an entry in the `.ssh/config` file.
- `AddKeysToAgent=yes` directs SSH to add private keys to the `ssh-agent`, if it's running.
    - For more information, refer to the section on ["ssh-agent"](ssh-agent.md#adding-identities).
- `HashKnownHosts=yes` directs SSH to hash the `.ssh/known_hosts` file.
    - For more information, refer to the section on the ["known_hosts" file](known-hosts.md#hashing-the-known_hosts-file).

Specific entries
----------------

When a user wants to add a new entry to the `.ssh/config` file, they can use the following format to create a standard entry:

```
Host <HOST1> <HOST2>
    HostName <DOMAIN>
    User <USER>
    IdentityFile <PATH_TO_PRIVATE_KEYFILE>
```

To explain in more detail:

- `Host <HOST1> <HOST2>` directs SSH to look for connections using any `<HOST>` specified.
    - Imagine the following entry: `Host github.com github`
        - A user can run `ssh github.com`, and SSH will find this entry and use its listed configs.
        - A user can also run `ssh github`, and SSH will find this entry and use its listed configs.
    - A user can specify as many `<HOST>` values as they want, so long as they don't conflict with other entries.
        - `Host` is the **only** option that can accept multiple values.
    - Essentially, the `<HOST>` value acts as an alias for the `HostName <DOMAIN>`.
- `HostName <DOMAIN>` directs SSH to connect using the `<DOMAIN>` specified.
    - Using the previous example, if the user is connecting to `github.com`, then the `<HOST>` values could be `website` and `example`.
        - So long as the `<DOMAIN>` is `github.com`, then `ssh website` and `ssh example` will direct SSH to connect to `github.com`.
- `User <USER>` directs SSH to connect to the `<DOMAIN>` with the specified username.
    - In the case of the previous example, a connection to `github.com` is usually achieved by connecting with the `git` user.
- `IdentityFile <PATH_TO_PRIVATE_KEYFILE>` directs SSH to use the private key found at `<PATH_TO_PRIVATE_KEYFILE>`.

Therefore, an entry for connecting to `github.com` may look similar to the following:

```
Host github.com github
    HostName github.com
    User git
    IdentityFile <PATH_TO_PRIVATE_KEYFILE>
```

With this entry, the user could run either `ssh github` or `ssh github.com` to open a connection. Both would work, because both `<HOST>` values (`github` and `github.com`) direct SSH to use that entry.

In addition, it is no longer necessary to specify the `<USER>` during the connection, as the entry contains a `<USER>` value (`git`). Therefore, SSH is directed to make the connection to `git@github.com`.

Adding multiple entries for the same host
-----------------------------------------

What happens when a user has two user accounts on the same target machine?

While it isn't possible to use the same `<HOST>` value for two entries, it *is* possible to have two entries that point to the same `<DOMAIN>`.

Consider the following example:

```
Host user_server
    HostName 192.168.1.10
    User user
    IdentityFile <PATH_TO_PRIVATE_KEYFILE>

Host otheruser_server
    HostName 192.168.1.10
    User other
    IdentityFile <PATH_TO_PRIVATE_KEYFILE>
```

In this case, the value for both `HostName` lines is `192.168.1.10`. However, one entry is configured to log into to the `user` account, and the other is configured to log in to the `other` account.

Therefore, the user now has two entries for two user accounts on the same server.

- Running `ssh user_server` logs the user into `user@192.168.1.10`.
- Running `ssh other_server` logs the user into `other@192.168.1.10`.

References
----------

- [SuperUser - How do I configure SSH so it doesn't try all the identity files automatically?](https://superuser.com/questions/268776/how-do-i-configure-ssh-so-it-doesnt-try-all-the-identity-files-automatically/268777#268777)
    - Reference for the `IdentitiesOnly=yes` option
- [StackOverflow - Specify an SSH key for git push for a given domain](https://stackoverflow.com/questions/7927750/specify-an-ssh-key-for-git-push-for-a-given-domain)
    - Reference for adding multiple entries for the same host
