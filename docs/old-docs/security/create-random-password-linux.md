Creating random passwords on Linux
==================================

Table of contents
-----------------

- [tr](#tr)
- [OpenSSL](#openssl)
- [GPG](#gpg)

tr
--

`tr` is a coreutil in Linux that stands for "translate", and it allows a user to take standard input and translate it to standard output.

`tr` can also be used to create a random password using the following options and specifications:

Example for a 16 character password:

```
tr -dc A-Za-z0-9 </dev/urandom | head -c 16; echo
```

The options for `tr` are as follows:

- `-d` represents "delete", and deletes the characters from the first array (which is the input from /dev/urandom)
- `-c` represents "complement", and it's what allows `tr` to take in the random input and output a password using the characters provided
- `A-za-z0-9` is the list of characters that are allowed to be used by `tr -c` to make the random password
- `</dev/urandom` tells `tr` to use "/dev/urandom" as the input source
    - that input is then used as the source of randomness for `tr` to translate into a password

The options for `head` are as follows:

- `-c` tells `head` to only show the amount of characters taken in as an argument (in the above case, 16 characters)

By piping that output into `head` and specifying a character length, a password can be obtained. Using `echo` at the end removes the unnecessary character produced by the shell.

OpenSSL
-------

`openssl` is a toolkit that enables the user to do a bunch of tasks related to secure communications, but can also be used to create random passwords.

Example for a 16 character password:

```
openssl rand -base64 16
```

This is generally an easier option to use, as opposed to `tr`, but it requires that `openssl` be installed on the Linux machine. Most Linux installations will have `openssl` by default, but it's even more likely that `tr` will already be installed, as `tr` is a GNU coreutil.

GPG
---

Another way to generate a random string for use as a password is by using the `gpg` package.

The command is as follows:

```
gpg --gen-random --armor 1 16
```

The options for `gpg --gen-random` are as follows:

- `--armor` tells `gpg` to output using ASCII characters, which makes the password legible.
- `1` select the level of quality used for generating randomness.
    - `0` and `1` will use `/dev/urandom`, while `2` uses `/dev/random` to generate entropy.
- `16` is the number of bytes used to generate the password.

While this is yet another option along with `tr` and `openssl rand`, it has its pros and cons. For most passwords, `tr` is a perfectly acceptable way to generate randomness and turn it into a password.

References
----------

- [StackExchange - How to generate a random string?](https://unix.stackexchange.com/questions/230673/how-to-generate-a-random-string)
- [Linux for Devices - How to Generate Random Passwords On Linux Shell](https://www.linuxfordevices.com/tutorials/linux/generate-random-passwords)
- [StackExchange - gpg --gen-random quality level: is higher "better"?](https://crypto.stackexchange.com/questions/30376/gpg-gen-random-quality-level-is-higher-better)
