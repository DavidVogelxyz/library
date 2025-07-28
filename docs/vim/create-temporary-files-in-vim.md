# Creating Temporary Files in Vim

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)

## Introduction

The obvious answer is to "just open Vim" by running:

```
vim
```

But, what if the user wants a scratch file of a certain file type?

To do this, the use can run a command with the following syntax:

```
vim $(mktemp).<FILE_EXTENSION>
```

As an example, imagine a user wanted to jot some notes in a scratch Markdown file. To do this, they would run the following command:

```
vim $(mktemp).md
```

Alternatively, if a user wanted to quickl write a scratch Bash script, they would run the following:

```
vim $(mktemp).sh
```
