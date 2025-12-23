Conditional expressions
=======================

Bash allows for various types of conditional expressions.

If/then statements
------------------

Matching
--------

To perform matching in Bash, write a `case` statement:

```bash
string="$1"

case "$string" in
    "1")
        echo "one"
        ;;
    "2")
        echo "two"
        ;;
    *)
        echo "hello"
        ;;
esac
```

Passing `1` as the argument prints `one`, passing `2` prints `two`, and passing any other argument prints `hello`.

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

### Verifying a hex string

To verify a hex string, write the following syntax:

```bash
string="$1"

if [[ "$string" =~ ^0x[0-9a-f]+$ ]]; then
    echo "string = $string"
elif [[ "$string" =~ ^[0-9a-f]+$ ]]; then
    echo "string = 0x${string}"
else
    echo "\"$string\" is not hex"
fi
```

Passing either `7` or `0x7` as the argument prints `string = 0x7`, but passing `hello` prints `"hello" is not hex`.
