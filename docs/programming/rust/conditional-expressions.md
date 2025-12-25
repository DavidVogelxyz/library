Conditional expressions
=======================

Rust allows for various types of conditional expressions.

If/then statements
------------------

Matching
--------

To perform matching in Python, write a `match` statement:

```rust
let string = "1";

match string {
    "1" => println!("one"),
    "2" => println!("two"),
    _ => println!("hello"),
}
```

Defining `string` as `1` prints `one`; `2` prints `two`; and, defining `string` as any other value prints `hello`.

Conditional constructs
----------------------

### String contains a substring

To conditionally assess whether a string contains a substring, write the following syntax:

```rust
let string = "rust";

if string.contains("r") {
    println!("\"rust\" contains \"r\"");
}

if string.contains("y") {
    println!("\"rust\" contains \"y\"");
}
```

Prints:

```
"rust" contains "r"
```

### Array contains an element

To conditionally assess whether an array contains an integer element, write the following syntax:

```rust
let arr = [1, 2, 3];

if arr.contains(&1) {
    println!("arr contains 1");
}

if arr.contains(&4) {
    println!("arr contains 4");
}
```

Prints:

```
arr contains 1
```

To conditionally assess whether an array contains a string element, write the following syntax:

```rust
let arr = ["1", "2", "3", "hello"];

if arr.contains(&"hello") {
    println!("arr contains \"hello\"");
}

if arr.contains(&"4") {
    println!("arr contains 4");
}
```

Prints:

```
arr contains "hello"
```

### Verifying a hex string

To verify a hex string, write the following syntax:

```rust
let mut string = "7";

string = string
    .strip_prefix("0x")
    .or_else(|| string.strip_prefix("0X"))
    .unwrap_or(string);

match i32::from_str_radix(string, 16) {
    Ok(value) => {
        println!("value = 0x{:x}", value);
    }

    Err(e) => {
        println!("\"{}\" is not hex -- {}", string, e);
    }
}
```

Defining `string` as either `7` or `0x7` prints `value = 0x7`; but, defining `string` as `hello` prints `"hello" is not hex -- invalid digit found in string`.
