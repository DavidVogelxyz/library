# Creating random passwords on Linux

## tr

`tr` is a coreutil in Linux that stands for "translate", and it allows a user to take standard input and translate it to standard output.

`tr` can also be used to create a random password using the following options and specifications:

Example for a 16 character password:

```
tr -dc A-Za-z0-9 </dev/urandom | head -c 16; echo
```

- 'd' represents "delete", and deletes the characters from the first array (which is the input from /dev/urandom)
- 'c' represents "complement", and it's what allows `tr` to take in the random input and output a password using the characters provided
- "A-za-z0-9" is the list of characters that are allowed to be used by `tr` to make the random password
- "</dev/urandom" uses "/dev/urandom" as the source of randomness, and uses that as the input for `tr` to translate into a password

By piping that output into `head` and specifying a character length, a password can be obtained. Using `echo` at the end removes the unnecessary character produced by the shell.

## OpenSSL

`openssl` is a toolkit that enables the user to do a bunch of tasks related to secure communications, but can also be used to create random passwords.

Example for a 16 character password:

```
openssl rand -base64 16
```

This is generally an easier option to use, as opposed to `tr`, but it requires that `openssl` be installed on the Linux machine. Most Linux installations will have `openssl` by default, but it's even more likely that `tr` will already be installed, as `tr` is a GNU coreutil.
