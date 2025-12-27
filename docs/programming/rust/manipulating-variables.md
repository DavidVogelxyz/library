Manipulating variables in Rust
==============================

[Back to the "Rust syntax guide" home page](README.md)

Length
------

### Length of a string

To get the length of a string, use the `len()` method:

```rust
let string = "hello";
let length = string.len();

println!("length = {}", length)
```

Prints:

```
length = 5
```

### Length of an array

To get the length of (number of elements in) an array, use the `len()` method:

```rust
let arr = [1, 2, 3];
let length = arr.len();

println!("length = {}", length)
```

Prints:

```
length = 3
```

Splitting
---------

To split a string on a given character, use the `split()` method:

```rust
let string = "10x20x30";
let nums = string.split("x");

for num in nums {
    println!("{}", num)
}
```

Prints:

```
10
20
30
```

This can be simplified to just:

```rust
let string = "10x20x30";

for num in string.split("x") {
    println!("{}", num)
}
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

```rust
let string = "hello";
let char = 'l';
let mut new = String::new();

for c in string.chars() {
    if c == char {
        new += &c.to_string();
    }
}

println!("new string = {}", new)
```

Prints:

```
new string = ll
```

As demonstrated, this can be useful when attempting to count the number of occurrences of a specific character.

### Removing a substring

To remove a substring from a string by removing all matches of a pattern, write the following syntax:

```rust
let string = "hello";
let char = 'l';
let mut new = String::new();

for c in string.chars() {
    if c != char {
        new += &c.to_string();
    }
}

println!("new string = {}", new)
```

Prints:

```
new string = heo
```

Absolute value
--------------

To get the absolute value of a number, use the `abs()` method:

```rust
let num: i32 = -5;
let new = num.abs();

println!("new = {}", new)
```

Prints:

```
new = 5
```
