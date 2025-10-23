Operating on Lines Based on a Pattern
=====================================

[Back to the home page](README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [References](#references)

Introduction
------------

To operate on a line, based on a pattern, run the `:g` command.

To delete any line that contains a pattern, run the following command:

```
:g/PATTERN/d
```

To delete any line that **does not** contain a pattern, run the following command:

```
:g!/PATTERN/d
```

To explain in more detail:

- `:g!` is analogous to `grep -v`
    - `grep -v` in**V**erts a match

References
----------

- [SuperUser - Vim - How to delete all lines that do not contain a certain word?](https://superuser.com/questions/265085/how-to-delete-all-lines-that-do-not-contain-a-certain-word-in-vim)
    - Reference for deleting lines that do not match a pattern
- [StackExchange - Sed - How to remove all lines that do not match](https://unix.stackexchange.com/questions/223897/sed-how-to-remove-all-lines-that-do-not-match/223899)
    - Alternate reference for deleting lines that do not match a pattern
