Renaming Files in Bulk
======================

[Back to the home page](README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Bulk renaming files](#bulk-renaming-files)
- [References](#references)

Introduction
------------

Vim makes it easy to rename multiple files in the same directory.

While Vim doesn't have a mechanism for doing this within the editor, Vim *does* make it easier to create a list of `mv` commands, to be executed by the `bash` shell.

This guide explains how to bulk rename files, with assistance from Vim.

Bulk renaming files
-------------------

First, pipe the output of `ls` into Vim.

```
\ls | vim -
```

To explain in more detail:

- `\` directs `ls` to only output file names.
- To output the names of all files, including dotfiles (hidden files), run `\ls -a`.
    - `\ls` will not output the name of dotfiles, just as `ls` does not output dotfiles.
- `vim -` directs Vim to receive the the pipe's `STDOUT` as input.
    - NOTE: Neovim doesn't appear to require the `-`, but Vim does. Neovim works with both `\ls | nvim` and `\ls | nvim -`.

Once Vim is open and contains the contents of the directory, run a "find-and-replace" command on the whole file (`:%s`):

```
:%s/<INPUT_PATTERN>/<OUTPUT_PATTERN>/g
```

- For the `<INPUT_PATTERN>`, substitute `.*`.
    - The `.*` will select the entirety of the line's content.
- For the `<OUTPUT_PATTERN>`, substitute `mv -i & &`.
    - The `&` character will substitute the `<INPUT_PATTERN>` back in.

Once this command executes, the user will have a series of lines that follow the format of `mv -i <FILENAME_INPUT> <FILENAME_OUTPUT>`.

The user can remove any lines for files that they don't want to rename, and edit the `<FILENAME_OUTPUT>` for the files that should be renamed. Very quickly, the user will create a series of `mv` commands to be executed.

When the list of commands is complete, the user can run the following Vim command:

```
:w !bash
```

The `:w` command will send the buffer as input to `bash`, running each command.

References
----------

- [Vim - Bulk rename files with Vim](https://vim.fandom.com/wiki/Bulk_rename_files_with_Vim)
    - Reference for using Vim to bulk rename files
- [Superuser - How to run Unix commands from within Vim?](https://superuser.com/questions/285500/how-to-run-unix-commands-from-within-vim)
    - Reference for running Unix (Linux) commands within Vim
