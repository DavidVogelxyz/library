Walking a map in Rust
=====================

Simple walk
-----------

To walk a map in Rust, write the following syntax:

```rust
let arr = [
    "123",
    "456",
    "789"
];

let grid = arr
    .iter()
    .map(|line| line
        .trim()
        .chars()
        .collect::<Vec<_>>())
    .collect::<Vec<_>>();

let rows_total = grid.len();

// Read through rows (vertically)
for row in 0.. rows_total {
    let cols_total = grid[row].len();

    // Read through the cols (horizontally)
    for col in 0.. cols_total {
        let char = grid[row][col];

        println!("char = {}", char)
    }
}
```

Prints:

```
char = 1
char = 2
char = 3
char = 4
char = 5
char = 6
char = 7
char = 8
char = 9
```

Walk and review
---------------

To walk a map in Python, and review the characters, write the following syntax:

```rust
let arr = [
    "###",
    "###",
    "###"
];

let grid = arr
    .iter()
    .map(|line| line
        .trim()
        .chars()
        .collect::<Vec<_>>())
    .collect::<Vec<_>>();

let rows_total = grid.len();

// Read through rows (vertically)
for row in 0.. rows_total {
    let cols_total = grid[row].len();

    // Read through the cols (horizontally)
    for col in 0.. cols_total {
        let char = grid[row][col];
        let mut count = 0;

        if char == '#' {
            for row_offset in -1..=1 {
                for col_offset in -1..=1 {
                    // Skip the "current character"
                    if row_offset == 0 && col_offset == 0 {
                        continue
                    }

                    let adj_row = row as i32 + row_offset;
                    let adj_col = col as i32 + col_offset;

                    // Skip if `adj_row` is off map
                    if adj_row < 0 || adj_row >= grid.len() as i32 {
                        continue
                    }
                    // Skip if `adj_col` is off map
                    if adj_col < 0 || adj_col >= grid[adj_row as usize].len() as i32 {
                        continue
                    }

                    let adj_value = grid[adj_row as usize][adj_col as usize];

                    // If `adj_value` is a "#", increase `count`
                    if adj_value == '#' {
                        count += 1
                    }
                }
            }

            if count >= 8 {
                println!("Found a character surrounded by 8 \"#\":");
                println!("line = {}", row + 1);
                println!("col = {}", col + 1);
            }
        }
    }
}
```

Prints:

```
Found a character surrounded by 8 "#":
line = 2
col = 2
```
