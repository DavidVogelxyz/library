Manipulating variables in Python
================================

[Back to the "Python syntax guide" home page](README.md)

Length
------

### Length of a string

To get the length of a string, use the `len()` method:

```python
string = "hello"
length = len(string)

print(f"length = {length}")
```

Prints:

```
length = 5
```

### Length of an array

To get the length of (number of elements in) an array, use the `len()` method:

```python
arr = [1, 2, 3]
length = len(arr)

print(f"length = {length}")
```

Prints:

```
length = 3
```

Splitting
---------

To split a string on a given character, use the `split()` method:

```python
string = "10x20x30"
nums = string.split("x")

for num in nums:
    print(f"{num}")
```

Prints:

```
10
20
30
```

This can be simplified to just:

```python
string = "10x20x30"

for num in string.split("x"):
    print(f"{num}")
```

Also prints:

```
10
20
30
```

Removing a substring from a string
----------------------------------

### Keeping a substring

To keep a substring from a string by removing anything that doesn't match the pattern, write the following syntax:

```python
string = "hello"
char = "l"
new = ""

for c in string:
    if char == c:
        new += c

print(f"new string = {new}")
```

Prints:

```
new string = ll
```

As demonstrated, this can be useful when attempting to count the number of occurrences of a specific character.

### Removing a substring

To remove a substring from a string by removing all matches of a pattern, write the following syntax:

```python
string = "hello"
char = "l"

new = string.replace(char, "")

print(f"new string = {new}")
```

Prints:

```
new string = heo
```

Absolute value
--------------

To get the absolute value of a number, use the `abs()` method:

```python
num = -5
new = abs(num)

print(f"new = {new}")
```

Prints:

```
new = 5
```
