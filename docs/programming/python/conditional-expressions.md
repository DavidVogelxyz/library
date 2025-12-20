Conditional expressions
=======================

Python allows for various types of conditional expressions.

If/then statements
------------------

Matching
--------

Conditional constructs
----------------------

### String contains a substring

To conditionally assess whether a string contains a substring, write the following syntax:

```python
string = "python"

if string.__contains__("y"):
    print("\"python\" contains \"y\"")

if string.__contains__("x"):
    print("\"python\" contains \"x\"")
```

Prints:

```
"python" contains "y"
```

### Array contains an element

To conditionally assess whether an array contains an element, write the following syntax:

```python
arr = [
    1,
    2,
    3
]

if arr.__contains__(1):
    print("arr contains 1")

if arr.__contains__(4):
    print("arr contains 4")
```

Prints:

```
arr contains 1
```
