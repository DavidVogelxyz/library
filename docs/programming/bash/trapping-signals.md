Trapping signals
================

[Back to the "Bash syntax guide" home page](README.md)

Bash allows for signal trapping in scripts, allowing the script to exit properly, even in the case of interruption or a pre-mature end to the process.

Table of contents
-----------------

- [Demonstrating the trap command](#demonstrating-trap)
- [Order of operations](#order-of-operations)
    - [Enabling the trap command too late](#enabling-trap-too-late)
    - [Enabling the trap command too early](#enabling-trap-too-early)
- [Trapping other useful signals](#trapping-other-useful-signals)
    - [Trapping multiple signals](#trapping-multiple-signals)
    - [Trapping the debug signal](#trapping-debug)
    - [Trapping the SIGINFO signal](#trapping-siginfo)
- [References](#references)

Demonstrating `trap`
--------------------

Consider the following script:

```bash
cleanup() {
    echo "Running cleanup operations..."
    rm -v test.log
    echo "Cleanup completed!"
}

trap cleanup exit

ping 127.0.0.1 2>&1 | tee test.log

sleep 10

echo "debug"
```

This scipt will:
- define the `cleanup` function
- enable `trap` to run `cleanup` when the script ends:
    - `exit` is a pseudo-signal that captures all process exits (including `SIGINT` and `SIGTERM`):
        - however, `exit` does **NOT** capture `SIGKILL`:
            - this is because a `SIGKILL` ends the process without allowing the process to exit
- ping the `localhost`:
    - redirect `STDERR` to `STDOUT` (`2>&1`)
    - pipe `STDOUT` to `tee test.log`
- on `SIGINT` (`Ctrl-C`), `sleep` for 10 seconds
- print `debug` to screen

This script demonstrates a few key points:
- `trap` only traps signals that are sent ***after*** `trap` runs
- `ping` technically violates this, as `ping` will continue to run until it is sent `SIGINT`:
    - in this way, `ping` essentially has a built-in `trap` that first captures any signal sent to the `ping` sub-process
- `2>&1 | tee test.log` has a dual function:
    - it allows for `STDOUT` and `STDERR` to print to the screen, as normal
    - but, it also allows for a log file to be created (`test.log`)
    - if `STDOUT` was redirected with `>`, then the log file would be created; but, no output would print to screen
- while `sleep` is also an external call, `sleep` is provided an argument for the runtime:
    - therefore, it does **NOT** trap `SIGINT` in the same way as `ping`

When run, the script will enable `trap` and then begin to ping `localhost`. It would ping until the user sends `SIGINT` with `Ctrl-C`. At that point, `sleep` would run for 10 seconds:
- if the user sends another `SIGINT` before `sleep` completes, then `trap` would capture that signal and run `cleanup`:
    - in this case, `echo "debug"` would **NOT** run
- if the user lets `sleep` complete, then the script would continue:
    - in this case, `echo "debug"` **would** run
    - the script would "complete", and the `exit` signal would be captured by `trap`, which would then call `cleanup`

Order of operations
-------------------

The behavior of the previous script has been described in detail in order to contrast against the following scripts. The next sections explain why it is important to understand the order of operations in a script, such that `trap` is called exactly when it should.

### Enabling `trap` too late

Consider the slighty altered script:

```bash
cleanup() {
    echo "Running cleanup operations..."
    rm -v test.log
    echo "Cleanup completed!"
}

ping 127.0.0.1 2>&1 | tee test.log

sleep 10

echo "debug"

trap cleanup exit
```

The only difference in this script, as opposed to the [original script](#demonstrating-trap), is that `trap` only runs ***after*** `ping` and `sleep`. The script's behavior would change accordingly, and would:
- define the `cleanup` function
- ping the `localhost`
- on `SIGINT` (`Ctrl-C`), `sleep` for 10 seconds
- print `debug` to screen
- enable `trap` to run `cleanup` when the script ends

So, contrasted against the first script, this script would define `cleanup` and begin to ping `localhost`. It would ping until the user sends `SIGINT` with `Ctrl-C`. At that point, `sleep` would run for 10 seconds:
- if the user sends another `SIGINT` before `sleep` completes, then the script would exit:
    - in this case, neither `echo "debug"` nor `trap` would be called:
        - therefore, `cleanup` will never run
- if the user lets `sleep` complete, then the script would continue:
    - in this case, `echo "debug"` **would** run
    - then, `trap` would be enabled
    - the script would "complete", and the `exit` signal would be captured by `trap`, which would then call `cleanup`

Therefore, it is important to enable `trap` early enough that it correctly captures the exit signal. If `trap` is enabled too late, it's possible that `trap` is never enabled, resulting in a failed cleanup operation.

### Enabling `trap` too early

Consider one more example:

```bash
cleanup() {
    echo "Running cleanup operations..."
    rm -v test.log
    echo "Cleanup completed!"
}

trap cleanup exit

sleep 10

ping 127.0.0.1 2>&1 | tee test.log

echo "debug"
```

As opposed to the [original script](#demonstrating-trap), `sleep` runs before `ping`. The script would instead:
- define the `cleanup` function
- enable `trap` to run `cleanup` when the script ends
- `sleep` for 10 seconds
- ping the `localhost`
- on `SIGINT` (`Ctrl-C`), print `debug` to screen

So, contrasted against the first script, this script would define `cleanup`, enable `trap`, and begin to sleep for 10 seconds:
- if the user sends `SIGINT` before `sleep` completes, then `trap` would capture that signal and run `cleanup`:
    - in this case, `cleanup` would attempt to remove `test.log`, which does not yet exist because `ping` was never called
- if the user lets `sleep` complete, then the script would continue:
    - in this case, `ping` would then run until receiving `SIGINT`
    - then, `echo "debug"` would print "debug" to the screen
    - the script would "complete", and the `exit` signal would be captured by `trap`, which would then call `cleanup`:
        - this call of `cleanup` would correctly remove the `test.log` file

Therefore, it is important to enable `trap` at the proper time. If `trap` is enabled too early, it's possible that `trap` runs `cleanup` in situations where it is not necessary, resulting in a failed cleanup operation.

Trapping other useful signals
-----------------------------

As previously described in [Demonstrating the trap command](#demonstrating-trap), trapping the `exit` pseudo-signal allows for any signal that exits the process to be captured with a single argument.

### Trapping multiple signals

To capture only `SIGINT` and `SIGTERM`, with the same `trap` call, provide them both as arguments:

```bash
trap <COMMAND> SIGINT SIGTERM
```

Alternatively, `trap` can be set with multiple calls, in order to trap different signals and provide different outcomes:

```bash
trap <COMMAND_1> SIGINT
trap <COMMAND_2> SIGTERM
```

### Trapping `debug`

Another useful signal is `debug`:

```bash
trap <COMMAND> debug
```

Once enabled, `trap <COMMAND> debug` will run `<COMMAND>` after every future function call. In this way, it behaves similarly to a script that is executed with `bash -x <SCRIPT>` (or, a script that has `set -x` enabled).

### Trapping `SIGINFO`

Another useful signal is `SIGINFO`:

```bash
trap <COMMAND> SIGINFO || exit 1
```

Once enabled, `trap <COMMAND> SIGINFO` will run `<COMMAND>` anytime `SIGINFO` (`Ctrl-T`) is sent.

Note the `|| exit 1` at the end of the `trap` call. This is important, because not all systems have `SIGINFO` -- most Linux distributions do **NOT** have `SIGINFO`. If the system does not have `SIGINFO`, the script will fail to set `trap <COMMAND> SIGINFO`, and will exit with return code `1`.

References
----------

- [YouTube - YouSuckAtProgramming - Episode 64 - Trapping signal with trap](https://www.youtube.com/watch?v=aXovP1sUtoE):
    - Dave Eddy's video on trapping signals with `trap`
- [Man page for trap](https://man7.org/linux/man-pages/man7/signal.7.html):
    - The `man` page for `trap`
- [Linux Journal - The Bash Trap Command](https://www.linuxjournal.com/content/bash-trap-command):
    - Additional information on the `exit` pseudo-signal
- [AskUbuntu - How to redirect output to screen as well as a file?](https://askubuntu.com/questions/38126/how-to-redirect-output-to-screen-as-well-as-a-file/38127#38127):
    - Information on `<COMMAND> 2>&1 | tee <FILE>`
