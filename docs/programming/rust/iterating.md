Iterating in Rust
=================

[Back to the "Rust syntax guide" home page](README.md)

For loop
--------

To iterate effectively in Rust, implement the built-in Rust `for` loop:

```rust
let string = "hello";

for c in string.chars() {
    println!("{}", c)
}
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

```rust
let arr = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let count = arr.len();
let n = 2;
let mut i = 0;

while i < count {
    println!("{}", arr[i]);
    i += n;
}
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

```rust
let arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100];
let count = arr.len();
let needle = 55;

for i in 0.. count {
    if arr[i] <= needle && needle <= arr[i+1] {
        println!("{} is between {} and {}.", needle, arr[i], arr[i+1]);
        break;
    }
}
```

Prints:

```
55 is between 50 and 60.
```
