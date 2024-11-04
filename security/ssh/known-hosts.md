# Working with the "known_hosts" file

[Back to the home page](README.md)

## Table of contents

- [Introduction](#Introduction)
- [Hashing the "known_hosts" file](#Hashing-the-known_hosts-file)
- [Searching a "known_hosts" file](#Searching-a-known_hosts-file)
- [Removing entries from a "known_hosts" file](#Removing-entries-from-a-known_hosts-file)
- [References](#References)

## Introduction

This section of the guide was written to specifically address the `.ssh/known_hosts` file and "hasing".

Some systems will hash the `.ssh/known_hosts` file by default, and others do not. Knowing how to work with a hashed `known_hosts` file is a useful skill.

## Hashing the "known_hosts" file

If a user has a `.ssh/known_hosts` file that isn't hashed, and they want the file hashes, they can direct `ssh-keygen` to do so with the following command:

```
ssh-keygen -H
```

If the `known_hosts` file is stored in a different location, the user can pass the `-f` switch to specify the path:

```
ssh-keygen -H -f <PATH>
```

Note that the hashing of the `known_hosts` file is a unidirectional process -- once hashed, the file cannot be "unhashed".

To direct SSH to either hash or "refrain from hashing" the `known_hosts` file, the following options can be provided to the `.ssh/config` file.

To direct SSH to hash the file:

```
Host *
    HashKnownHosts=yes
```

To direct SSH **not** to hash the file:

```
Host *
    HashKnownHosts=no
```

For more information, refer to the section on the ["ssh_config" file](ssh-config.md#General-configuration).

## Searching a "known_hosts" file

To search the `.ssh/known_hosts` file, run the following command:

```
ssh-keygen -F <HOSTNAME>
```

If the `known_hosts` file is hashed, then SSH will hash the `<HOSTNAME>` provided and search the file for the hash. While `ssh-keygen -F` can be used to search both hashed and "non-hashed" `known_hosts` files, it is absolutely necessary for a hashed file.

To search the file and return hosts in their hashed format, run the following command:

```
ssh-keygen -H -F <HOSTNAME>
```

## Removing entries from a "known_hosts" file

To remove entries from the `.ssh/known_hosts` file, run the following command:

```
ssh-keygen -R <HOSTNAME>
```

As with the `ssh-keygen -F` command, `ssh-keygen -R` works for both hashed and "non-hashed" `known_hosts` file. But, as is also true for `ssh-keygen -F`, `ssh-keygen -R` is absolutely necessary for a hashed `known_hosts` file.

## References

- [StackExchange - SSH: benefits of using hashed known hosts](https://security.stackexchange.com/questions/56268/ssh-benefits-of-using-hashed-known-hosts)
    - Reference on the benefits of hashing the `known_hosts` file
- [StackExchange - Why does my SSH Known Hosts have hashes instead of hostnames or IPs?](https://unix.stackexchange.com/questions/719513/why-does-my-ssh-known-hosts-have-hashes-instead-of-hostnames-or-ips)
    - Reference for "hashed" `known_hosts` files
- [Superuser - Identify the entries within .ssh/known_hosts?](https://superuser.com/questions/1341788/identify-the-entries-within-ssh-known-hosts)
    - Reference for searching a `known_hosts` file
- [StackExchange - How to decrypt hostnames of a crypted .ssh/known_hosts with a list of the hostnames?](https://unix.stackexchange.com/questions/175071/how-to-decrypt-hostnames-of-a-crypted-ssh-known-hosts-with-a-list-of-the-hostna/175072#175072)
    - Another reference for searching a `known_hosts` file, and returning entries in both hashed and unhashed formats
