Reading input in Python
=======================

There are a few ways to read input in Python.

Open, read, strip
-----------------

```python
with open("file.txt", "r") as file:
    arr = file.read().strip("\n\n")
```

This will open `file.txt` for reading (`r`), and will assign the contents to the variable `file`.

Then, iterate over the contents of `file` by reading (`read()`) and stripping (`strip()`) each "unit" of any character (in this case, `\n\n`).

Then, each "unit" is added to an array (`arr`).

Open, read, strip, split
------------------------

```python
with open("file.txt", "r") as file:
    arr = file.read().strip("\n\n").split("\n")
```

This will open `file.txt` for reading (`r`), and will assign the contents to the variable `file`.

Then, iterate over the contents of `file` by reading (`read()`) and stripping (`strip()`) each "unit" of any character (in this case, `\n\n`). Next, each "unit" will be split (`split()`) on a given character (in this case, `\n`).

Then, each "unit" is added to an array (`arr`).
