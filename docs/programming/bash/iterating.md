Iterating in Bash
=================

C-style for loop
----------------

To iterate effectively in Bash, write a "C-style" for loop:

```bash
arr=(1 2 3)
count="${#arr[@]}"

for ((i=0; i < count; i++)); do
    echo "${arr[i]}"
done
```

Prints:

```
1
2
3
```

Iterating over characters in a string
-------------------------------------

To iterate over characters in a string, write a "C-style" for loop and use parameter expansion:

```bash
string="hello"
count="${#string}"

for ((i=0; i < count; i++)); do
    echo "${string:i:1}"
done
```

Prints:

```
h
e
l
l
o
```

Increment by a numbers besides 1
--------------------------------

To increment by any number besides 1, write the following "C-style" for loop:

```bash
arr=(0 1 2 3 4 5 6 7 8 9 10)
count="${#arr[@]}"
n=2

for ((i=0; i < count; i+=n)); do
    echo "${arr[i]}"
done
```

Prints:

```
0
2
4
6
8
10
```
