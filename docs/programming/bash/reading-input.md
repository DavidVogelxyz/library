Reading input in Bash
=====================

There are a few ways to read input in Bash.

Read
----

A `while read` loop:

```bash
while read -r var1 var2; do
    echo "var1 = $var1"
    echo "var2 = $var2"
done < "file.txt"
```

Setting an IFS (inter-field separator) for a `while read` loop:

```bash
while IFS="," read -r var1 var2; do
    echo "var1 = $var1"
    echo "var2 = $var2"
done < "file.txt"
```

Readarray
---------

Read a file into an array:

```bash
readarray -t arr < "file.txt"
```
