Conditional expressions
=======================

Bash allows for various types of conditional expressions.

If/then statements
------------------

Matching
--------

Conditional constructs
----------------------

### String contains a substring

To conditionally assess whether a string contains a substring, write the following syntax:

```bash
if [[ "bash" =~ "a" ]]; then
    echo "\"bash\" contains \"a\""
fi

if [[ "bash" =~ "o" ]]; then
    echo "\"bash\" contains \"o\""
fi
```

Prints:

```
"bash" contains "a"
```

### Array contains an element

To conditionally assess whether an array contains an element, write the following syntax:

```bash
arr=(1 2 3)

if [[ "${arr[@]}" =~ 1 ]]; then
    echo "array contains 1"
fi

if [[ "${arr[@]}" =~ 4 ]]; then
    echo "array contains 4"
fi
```

Prints:

```
array contains 1
```
