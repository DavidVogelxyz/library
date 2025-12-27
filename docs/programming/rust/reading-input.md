Reading input in Rust
=====================

[Back to the "Rust syntax guide" home page](README.md)

There are a few ways to read input in Rust.

read_to_string
--------------

With `read_to_string()`:

```rust
use std::fs;

let lines = fs::read_to_string("test.txt").expect("File should be readable.");

println!("{}", lines)
```
