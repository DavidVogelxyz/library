Iterating in Python
===================

For loop
--------

To iterate effectively in Python, implement the built-in Python `for` loop:

```python
string = "hello"

for c in string:
    print(f"{c}")
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

To increment by any number besides 1, implement a `while` loop:

```python
arr = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
count = len(arr)
n = 2
i = 0

while i < count:
    print(f"{arr[i]}")
    i += n
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

It is also possible to decrement an index -- the opposite of `i += n` is `i -= n`.

Indexing into an array by adding to an index
--------------------------------------------

To index into an array by adding to an index, implement a `while` loop:

```python
arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
count = len(arr)
needle = 55
i = 0

while i < count:
    if arr[i] <= needle and needle <= arr[i+1]:
        print(f"{needle} is between {arr[i]} and {arr[i+1]}.")
        break

    i += 1
```

Prints:

```
55 is between 50 and 60.
```
