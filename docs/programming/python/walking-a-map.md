Walking a map in Python
=======================

Simple walk
-----------

To walk a map in Python, write the following syntax:

```python
arr = [
    "123",
    "456",
    "789"
]

rows_total = len(arr)
row = 0

# Read through rows (vertically)
while row < rows_total:
    cols_total = len(arr[row])
    col = 0

    # Read through the cols (horizontally)
    while col < cols_total:
        char = arr[row][col]

        print(f"char = {char}")

        col += 1

    row += 1
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

```python
arr = [
    "###",
    "###",
    "###"
]

rows_total = len(arr)
row = 0

# Read through rows (vertically)
while row < rows_total:
    cols_total = len(arr[row])
    col = 0

    # Read through the cols (horizontally)
    while col < cols_total:
        char = arr[row][col]
        count = 0

        if char == "#":
            # `range(-1, 2)` defines `[-1, 0, 1]`
            for row_offset in range(-1, 2):
                # `range(-1, 2)` defines `[-1, 0, 1]`
                for col_offset in range(-1, 2):
                    # Skip the "current character"
                    if row_offset == 0 and col_offset == 0:
                        continue

                    adj_row = row + row_offset
                    adj_col = col + col_offset

                    # Skip if `adj_row` is off map
                    if adj_row < 0 or adj_row >= rows_total:
                        continue

                    # Skip if `adj_col` is off map
                    if adj_col < 0 or adj_col >= cols_total:
                        continue

                    adj_value = arr[adj_row][adj_col]

                    # If `adj_value` is a "#", increase `count`
                    if adj_value == "#":
                        count += 1

            if count >= 8:
                print("Found a character surrounded by 8 \"#\":")
                print(f"line = {row + 1}")
                print(f"col = {col + 1}")

        col += 1

    row += 1
```

Prints:

```
Found a character surrounded by 8 "#":
line = 2
col = 2
```
