Conditional expressions
=======================

[Back to the "Python syntax guide" home page](README.md)

Python allows for various types of conditional expressions.

If/then statements
------------------

Matching
--------

To perform matching in Python, write a `match` statement:

```python
string = "1"

match string:
    case "1":
        print("one")
    case "2":
        print("two")
    case _:
        print("hello")
```

Defining `string` as `1` prints `one`; `2` prints `two`; and, defining `string` as any other value prints `hello`.

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
arr = [1, 2, 3]

if arr.__contains__(1):
    print("arr contains 1")

if arr.__contains__(4):
    print("arr contains 4")
```

Prints:

```
arr contains 1
```

### Verifying a hex string

To verify a hex string, write the following syntax:

```python
string = "7"

try:
    value = int(string, 16)
    print(f"value = 0x{value}")
except Exception as e:
    print(f"\"{string}\" is not hex -- {e}")
```

Defining `string` as either `7` or `0x7` prints `value = 0x7`; but, defining `string` as `hello` prints `"hello" is not hex -- invalid literal for int() with base 16: 'hello'`.
