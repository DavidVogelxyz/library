Reading input in Bash
=====================

[Back to the "Bash syntax guide" home page](README.md)

There are a few ways to read input in Bash.

Table of contents
-----------------

- [Read](#read)
    - [Using the read command to assign variables from an input file](#using-read-to-assign-variables-from-an-input-file)
    - [Using the read command to assign variables into an array](#using-read-to-assign-variables-into-an-array)
    - [Using the read command to split a string](#using-read-to-split-a-string)
    - [Using the read command to assign variables from the output of a command](#using-read-to-assign-variables-from-the-output-of-a-command)
        - [Using a here string to read the complete output of a command](#using-a-here-string-to-read-the-complete-output-of-a-command)
        - [Using process substitution to read the streamed output of a command](#using-process-substitution-to-read-the-streamed-output-of-a-command)
- [Readarray](#readarray)
- [References](#references)

Read
----

The `read` command can be used to assign variables from a variety of inputs.

### Using `read` to assign variables from an input file

To read an input file and assign variables, redirect the file into a `while read` loop:

```bash
while read -r var1 var2; do
    echo "var1 = $var1"
    echo "var2 = $var2"
done < "file.txt"
```

Imagine `file.txt` contains:

```
a b
c d
```

The `while read` loop would read `file.txt` and print:

```
var1 = a
var2 = b
var1 = c
var2 = d
```

If `file.txt` contains:

```
a b c
d e f
```

Then the same `while read` loop would print:

```
var1 = a
var2 = b c
var1 = d
var2 = e f
```

`read` will split into as many variables as were provided. If `read` has more content than variables, it will ignore the split and assign the remaining content to the last available variable.

By default, `read` splits on whitespace (` `).

If `file.txt` contains:

```
a,b
c,d
```

Then the same `while read` loop would print:

```
var1 = a,b
var2 =
var1 = c,d
var2 =
```

To split on some other value, set the "IFS" (inter-field separator).

This can be done by setting `IFS` as an environment variable for the `read` command:

```bash
while IFS="," read -r var1 var2; do
    echo "var1 = $var1"
    echo "var2 = $var2"
done < "file.txt"
```

This `while read` loop would split on `,`, and would parse the comma-separated `file.txt` as:

```
var1 = a
var2 = b
var1 = c
var2 = d
```

### Using `read` to assign variables into an array

`read` can also assign variables as elements of an array:

```bash
while read -r -a letters; do
    echo "${letters[1]}"
done < "file.txt"
```

Imagine `file.txt` contains:

```
a b
c d
```

The `while read` loop would read `file.txt` and print:

```
b
d
```

### Using `read` to split a string

It is also possible to split a string with `read`:

```bash
string="20-30"

IFS="-" read -r left right <<< "$string"

echo "left = $left"
echo "right = $right"
```

Prints:

```
left = 20
right = 30
```

However, Bash will operate faster if splitting is done through [parameter expansion](parameter-expansion.md#splitting-with-parameter-expansion). Using `read` to split is only advisable when parameter expansion is too cumbersome. An example of a cumbersome string could be `20,30,40,50,60` -- this string would be easier to split using `read`; though, it would run slower.

Notice that the `while read` loop from [using the read command to assign variables from an input file](#using-read-to-assign-variables-from-an-input-file) redirected the file input with `<`, but this `read` command redirects `$string` with `<<<`. The `<<<` is a variant of a redirect called a "here string", and it allows a variable to be read into `read`. The value (in this case, `$string`) undergoes tilde expansion, parameter and variable expansion, command substitution, arithmetic expansion, and quote removal before being passed to the `read` command.

### Using `read` to assign variables from the output of a command

There are two ways to read the output of a command into `read`: here strings, and process substitution.

Here strings wait for the command to complete before `read` starts to process the command's output, while process substitution allows the command's output to be streamed to `read`.

#### Using a here string to read the complete output of a command

Using a "here string", it is also possible to use `read` to assign a variable from the output of a command:

```bash
read -r output <<< "$(grep "a" file.txt)"
echo "$output"
```

Imagine `file.txt` contains:

```
a b
c d
```

The `read` command would read the output of `grep` and print:

```
a b
```

What is noteworthy about the "here string" (`<<<`) is that the command (in this case, `grep`) must complete before `read` begins to process the command's `STDOUT`. In a case where the command will complete quickly, or does not output much to memory, this isn't a big deal. But, for larger files or slower commands, this can be inadvisable.

#### Using process substitution to read the streamed output of a command

Sometimes, it is more efficient to stream the output of a command. In these cases, use process substitution:

```bash
while read -r output; do
    echo "output"
done < <(grep "a" file.txt)
```

In contrast to a "here string", the `read` command will receive streamed output from the command (in this case, `grep`). Therefore, the command does not have to complete before the `read` command will begin to process the command's `STDOUT`.

Readarray
---------

Read a file into an array:

```bash
readarray -t arr < "file.txt"
```

References
----------

- [YouTube - YouSuckAtProgramming - Episode 73 - Command vs. Process substitution in Bash](https://www.youtube.com/watch?v=f3eIK5xk4vg):
    - Dave Eddy's video on "command vs. process substitution"
- [GNU - Bash reference manual - Redirections](https://www.gnu.org/software/bash/manual/bash.html#Redirections-1):
    - Information on the different variations of "redirection"
