# Working with the "authorized_keys" file

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Best practices](#best-practices)

## Introduction

Especially on systems with many keys added to the `.ssh/authorized_keys` file, having good "hygiene" is important and useful.

This section will highlight some tips for managing the `.ssh/authorized_keys` file.

## Best practices

The main point to address is that the `.ssh/authorized_keys` file, like any other Bash file, can have lines commented out with `#`.

Therefore, it's a good practice to maintain heavily used `authorized_keys` files by adding comments above pubkey entries. For example, a helpful comment could include the date when the key was added, who added the key, and for what purpose.

This can allow a user to return in the future, quickly skim the file, and understand which keys are what. Especially if a key needs to be revoked in the future, a cursory glance of useful comments will help the user to identify which key should be revoked. Extending the example, a key doesn't have to be deleted to be revoked -- it can simply be commented out. Then, a comment can be added to reference when the key was revoked, and by whom, increasing the utility of the `authorized_keys` file as a record of which keys have been allowed to access the system.
