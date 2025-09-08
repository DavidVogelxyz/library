git-difftool - Viewing Differences using a Text Editor
======================================================

[Back to the home page](../README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Using "git-difftool" to view diffs in a text editor](#using-git-difftool-to-view-diffs-in-a-text-editor)
- [Configuring the default tool for "git-difftool"](#configuring-the-default-tool-for-git-difftool)
- [Getting text to wrap when using "vimdiff" or "nvimdiff"](#getting-text-to-wrap-when-using-vimdiff-or-nvimdiff)
- [References](#references)

Introduction
------------

As described in the [section on "git diff"](git-diff.md#introduction), `git difftool` is often a more effective way of working with diffs.

Beyond specifying the tool that `git difftool` uses, and configuring a default tool, the switches for `git difftool` are essentially the same as `git diff`. Therefore, refer to the [section on "git diff"](git-diff.md#using-git-diff-to-view-changes) for information on how to use `git difftool`.

This section will describe how to use and configure Vim `difftools` (`vimdiff` and `nvimdiff`) with `git difftool`. However, the user can use the same commands to select whichever tool they prefer.

Using "git-difftool" to view diffs in a text editor
---------------------------------------------------

When using `git diff` commands, simply replace `diff` with `difftool` to use the default `difftool` set on the system:

```bash
git difftool
```

To use a specific `difftool` (ex. `nvimdiff`), run the following command:

```bash
git difftool --tool="nvimdiff"
```

As mentioned previously, the other switches for `git difftool` are essetnially identical to `git diff`.

As an example, to view the differences between the working tree and all staged files, the user would run:

```bash
git difftool --staged
```

Refer to the [section on "git diff"](git-diff.md#using-git-diff-to-view-changes) for information on other switches.

Configuring the default tool for "git-difftool"
-----------------------------------------------

Once a user knows their preferred `difftool`, they may want to set that configuration globally, so that they don't need to specify it each time with `git difftool --tool`.

To set the global `difftool` for `git difftool`, run the following command:

```bash
git config --global diff.tool nvimdiff
```

To set the global `difftool` for resolving conflicts, run the following command:

```bash
git config --global merge.tool nvimdiff
```

Now, when `git difftool` is run, it will automatically know to use `nvimtool`. However, `git difftool` will still prompt the user for permission to open the tool every time the command is run. To set the configuration so that `git difftool` doesn't prompt the user, and just opens the `difftool`, run the following command:

```bash
git config --global difftool.prompt false
```

By running `git config --global`, these configs are set in the `~/.gitconfig` file. For more information, refer to the section on ["git-config"](git-config.md#global-configs-for-git-difftool).

Getting text to wrap when using "vimdiff" or "nvimdiff"
-------------------------------------------------------

By default, `vimdiff` and `nvimdiff` do not wrap text, even if a user's Vim or Neovim is configured to do so. While `:set wrap` does work, as it would in `vim` or `nvim`, the option only takes effect on a single pane.


In order to wrap text in `vimdiff` or `nvimdiff` for all panes in a window, run the following command:

```vim
:windo set wrap
```

To remove the wrapping option for all panes, run the following command:

```vim
:windo set nowrap
```

References
----------

- [StackOverflow - Configuring diff tool with .gitconfig](https://stackoverflow.com/questions/6412516/configuring-diff-tool-with-gitconfig)
    - Reference on how to configure a default `difftool`
- [SuperUser - How do I enable word wrap in all open splits?](https://superuser.com/questions/307331/how-do-i-enable-word-wrap-in-all-open-splits)
    - Reference for getting text to wrap when using `vimdiff` and `nvimdiff`
