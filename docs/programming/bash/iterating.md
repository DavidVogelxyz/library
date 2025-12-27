Iterating in Bash
=================

[Back to the "Bash syntax guide" home page](README.md)

C-style for loop
----------------

To iterate effectively in Bash, implement a "C-style" `for` loop:

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

To iterate over characters in a string, implement a "C-style" `for` loop along with parameter expansion:

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

Incrementing by any number besides 1
------------------------------------

To increment by any number besides 1, implement a "C-style" `for` loop:

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

It is also possible to decrement an index. The opposite of `i++` is `i--`, and the opposite of `i+=n` is `i-=n`.

Indexing into an array by adding to an index
--------------------------------------------

To index into an array by adding to an index, implement a "C-style" `for` loop:

```bash
arr=(10 20 30 40 50 60 70 80 90 100)
count="${#arr[@]}"
needle="55"

for ((i=0; i < count; i++)); do
    if ((arr[i] <= needle && needle <= arr[i+1])); then
        echo "$needle is between ${arr[i]} and ${arr[i+1]}."
        break
    fi
done
```

Prints:

```
55 is between 50 and 60.
```
