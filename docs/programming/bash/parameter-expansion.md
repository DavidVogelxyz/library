Parameter expansion
===================

[Back to the "Bash syntax guide" home page](README.md)

Parameter expansion in Bash allows a programmer to access information about a variable that is often retrieved in other languages by using various methods.

Table of contents
-----------------

- [Length](#length)
    - [Length of a string](#length-of-a-string)
    - [Length of an array](#length-of-an-array)
- [Splitting](#splitting)
    - [Splitting with parameter expansion](#splitting-with-parameter-expansion)
- [Removing a substring from a string](#removing-a-substring-from-a-string)
    - [Keeping a substring](#keeping-a-substring)
    - [Removing a substring](#removing-a-substring)
- [Absolute value](#absolute-value)
- [Single quotes vs double quotes](#single-quotes-vs-double-quotes)
- [Uppercase and lowercase](#uppercase-and-lowercase)
- [References](#references)

Length
------

### Length of a string

To get the length of a string, perform parameter expansion with the `#` character:

```bash
string="hello"
length="${#string}"

echo "length = $length"
```

Prints:

```
length = 5
```

### Length of an array

To get the length of (number of elements in) an array, perform parameter expansion with the `#` character on the array:

```bash
arr=(1 2 3)
length="${#arr[@]}"

echo "length = $length"
```

Prints:

```
length = 3
```

Splitting
---------

### Splitting with parameter expansion

Split with parameter expansion:

```bash
string="20-30"

left="${string%%-*}"
right="${string##*-}"

echo "left = $left"
echo "right = $right"
```

Prints:

```
left = 20
right = 30
```

When assigning `left`, Bash interprets `%%` as "read from right to left (backwards) until the last instance of the pattern" (`-*`). Then, Bash assigns `left` as "everything that is **NOT** that pattern" (in this case, `20`).

Similarly, when assigning `right`, Bash interprets `##` as "read from left to right (fowards) until the last instance of the pattern" `*-`. Then, Bash assigns `right` as "everything that is **NOT** that pattern" (in this case, `30`).

It is also possible to [split with the read command](reading-input.md#using-read-to-split-a-string); though, Bash will operate faster if splitting is done with parameter expansion. Using `read` to split is only advisable when parameter expansion is too cumbersome. An example of a cumbersome string could be `20,30,40,50,60` -- this string would be easier to split using `read`; though, it would run slower.

Removing a substring from a string
----------------------------------

### Keeping a substring

To keep a substring from a string by removing anything that doesn't match the pattern, write the following syntax:

```bash
string="hello"
char="l"

new="${string//[^$char]}"

echo "new string = $new"
```

Prints:

```
new string = ll
```

As demonstrated, this can be useful when attempting to count the number of occurrences of a specific character.

### Removing a substring

To remove a substring from a string by removing all matches of a pattern, write the following syntax:

```bash
string="hello"
char="l"

new="${string//[$char]}"

echo "new string = $new"
```

Prints:

```
new string = heo
```

Absolute value
--------------

Bash does not have an "absolute value" method, as other languages do. Instead, perform parameter expansion to remove any `-` by "reading from left to right" (`#`) and matching the first instance of the pattern (`-`):

```bash
arr=(1 2 -1 -2)

for ((i=0; i < "${#arr[@]}"; i++)); do
    echo "${arr[$i]#-}"
done
```

Prints:

```
1
2
1
2
```

Single quotes vs double quotes
------------------------------

Often, it is practical to make use of double quotes when quoting variables, in order to ensure that parameters are properly expanded.

However, there are some instances where it is imperative that the parameter is **NOT** expanded. In these cases, use single quotes.

An example of this may be a script that executes the following command:

```bash
ssh <TARGET> 'kill -9 $(pidof sleep)'
```

This command will `ssh` into `<TARGET>` in order to run `kill -9 $(pidof sleep)`. If double quotes are used in this command, then `$(pidof sleep)` will expand on the host system, and that expansion will be run on the target machine. That is likely **NOT** the intention of the person running the script, as the expansion will not result in the PID of the `sleep` process running on `<TARGET>`.

However, if single quotes are used, then the command `kill -9 $(pidof sleep)` will be expanded on the target machine, and will correctly expand to the PID of the `sleep` process on the target machine.

Uppercase and lowercase
-----------------------

To convert a string to uppercase, write the following syntax:

```bash
string="hello"

echo "string = ${string@U}"
```

Prints:

```
HELLO
```

To convert a string to lowercase, write the following syntax:

```bash
string="HELLO"

echo "string = ${string@L}"
```

Prints:

```
hello
```

References
----------

- [GNU - Bash reference manual - Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion-1)
    - Information on "parameter expansion" in Bash
- [YouTube - YouSuckAtProgramming - Episode 21 - Escaping and Wrangling quotes in Bash](https://www.youtube.com/watch?v=5DD7YrOgmq4):
    - Dave Eddy's video on "quotes, in Bash"
