Printf
======

[Back to the "Bash syntax guide" home page](README.md)

Similar to C, Bash has the `printf` function built into the shell. Below are a few pecularities.

Table of contents
-----------------

- [Using the printf command to print the elements of an array](#using-printf-to-print-the-elements-of-an-array)
- [Using the printf command to format and print an array](#using-printf-to-format-and-print-an-array)
- [Using the printf command to pad and print lines](#using-printf-to-pad-and-print-lines)
- [Using the printf command to print any number of consecutive characters](#using-printf-to-print-any-number-of-consecutive-characters)
- [References](#references)

Using `printf` to print the elements of an array
------------------------------------------------

To print the elements in an array, separated by newlines, write the following syntax:

```bash
arr=(1 2 3)

printf "%s\n" "${arr[@]}"
```

Prints:

```
1
2
3
```

Two other ways to print the same array are as follows.

Writing a `for` loop:

```bash
arr=(1 2 3)

for num in "${arr[@]}"; do
    echo "$num"
done
```

Writing a "C-style" `for` loop:

```bash
arr=(1 2 3)

for ((i=0; i < ${#arr[@]}; i++)); do
    echo "${arr[i]}"
done
```

The benefit of `printf`, as opposed to a `for` loop calling `echo`, is that `printf` only requires 1 function call in order to print all the elements of the array. In contrast, the `for` loops will call `echo` for every element of the array. For arrays with few elements, this is largely insignificant; but, for arrays with many elements, this repeated invocation of `echo` requires far more computation.

Using `printf` to format and print an array
-------------------------------------------

Consider the output of the [previous example](#using-printf-to-print-the-elements-of-an-array):

```
1
2
3
```

Imagine a user wants to format the output so the elements are prefixed with `-`, like in the following snippet:

```
- 1
- 2
- 3
```

This is possible with `printf`. To do so, write the following syntax:

```bash
arr=(1 2 3)

printf -- "- %s\n" "${arr[@]}"
```

Using `printf` to pad and print lines
-------------------------------------

Sometimes, when writing a script, a user will want to pad the printed output, such that strings always appears with the same spacing regardless of the length of the strings.

To explain padding, refer to the [prox-dush](https://github.com/DavidVogelxyz/bin-proxmox/blob/master/bin-proxmox/prox-dush) script, found within the [bin-proxmox](https://github.com/DavidVogelxyz/bin-proxmox) repo.

Consider the following output of `prox-dush`:

```
SIZE     USAGE    VM ID            NAME OF VM
100G     20%      VM_ID_1          VM_NAME_1
50G      10%      VM_ID_2          VM_NAME_2
5G       1%       VM_ID_3          VM_NAME_3
```

Regardless of the values, it may be valuable to guarantee that the outputs always line up correctly within their columns.

This is possible with `printf`. To do so, write the following syntax for the headers:

```bash
printf "%-8s %-8s %-16s %s\n" "SIZE" "USAGE" "VM ID" "NAME OF VM"
```

The same can be done when looping through the entries in the table:

```bash
printf "%-8s %-8s %-16s %s\n" "${sizes[$id]}" "$((usage[$id] * 100 / total_usage))%" "$id" "${names[$id]}"
```

Using `printf` to print any number of consecutive characters
------------------------------------------------------------

`printf` can also be manipulated into printing any number of the same character. To do so, write the following syntax:

```bash
count=10
char="="
empty_line="$(printf "%*s\n" "$count")"
new_line="${empty_line// /$char}"
echo "$new_line"
```

Prints:

```
==========
```

This works by assigning `empty_line` to a string that is just whitespace (` `), padded to the value of `count`. Then, `new_line` is assigned to the value of `empty_line` after being expanded to replace all instances of the whitespace with the value of `char`.

This trick was utilized in the [daily-note](https://github.com/DavidVogelxyz/bin-linux/blob/master/bin-linux/daily-note) script, found within the [bin-linux](https://github.com/DavidVogelxyz/bin-linux) repo.

References
----------

- [GitHub - DavidVogelxyz - bin-proxmox - prox-dush](https://github.com/DavidVogelxyz/bin-proxmox/blob/master/bin-proxmox/prox-dush):
    - Example script that makes use of `printf` to pad printed text
- [GitHub - DavidVogelxyz - bin-linux - daily-note](https://github.com/DavidVogelxyz/bin-linux/blob/master/bin-linux/daily-note):
    - Example script that makes use of `printf` to print a number of consecutive characters
