Walking a map in Bash
=====================

[Back to the "Bash syntax guide" home page](README.md)

Simple walk
-----------

To walk a map in Bash, write the following syntax:

```bash
arr=(
    "123"
    "456"
    "789"
)

rows_total="${#arr[@]}"

# Read through rows (vertically)
for ((row=0; row < rows_total; row++)); do
    curr_row=(${arr[$row]})
    cols_total="${#curr_row}"

    # Read through cols (horizontally)
    for ((col=0; col < cols_total; col++)); do
        char="${curr_row:$col:1}"
        echo "char = $char"
    done
done
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

To walk a map in Bash, and review the characters, write the following syntax:

```bash
arr=(
    "###"
    "###"
    "###"
)

rows_total="${#arr[@]}"

# Read through rows (vertically)
for ((row=0; row < rows_total; row++)); do
    curr_row=(${arr[$row]})
    cols_total="${#curr_row}"

    # Read through cols (horizontally)
    for ((col=0; col < cols_total; col++)); do
        char="${curr_row:$col:1}"
        count=0

        if [[ "$char" == "#" ]]; then
            for row_offset in {-1..1}; do
                for col_offset in {-1..1}; do
                    # Skip the "current character"
                    if ((row_offset == 0)) && ((col_offset == 0)); then
                        continue
                    fi

                    adj_row="$((row + row_offset))"
                    adj_col="$((col + col_offset))"

                    # Skip if `adj_row` is off map
                    if ((adj_row < 0)) || ((adj_row >= rows_total)); then
                        continue
                    fi

                    # Skip if `adj_col` is off map
                    if ((adj_col < 0)) || ((adj_col >= cols_total)); then
                        continue
                    fi

                    adj_value="${arr[$adj_row]:$adj_col:1}"

                    # If `adj_value` is a "#", increase `count`
                    if [[ "$adj_value" == "#" ]]; then
                        count="$((count + 1))"
                    fi
                done
            done

            # If `$count` is 8 (or greater), print coordinates
            if ((count >= 8)); then
                echo "Found a character surrounded by 8 \"#\":"
                echo "line = $((row + 1))"
                echo "col = $((col + 1))"
            fi
        fi
    done
done
```

Prints:

```
Found a character surrounded by 8 "#":
line = 2
col = 2
```
