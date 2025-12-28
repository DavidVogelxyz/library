Getopts in Bash
===============

[Back to the "Bash syntax guide" home page](README.md)

Built into Bash is the `getopts` option, which makes it easy to set command line options in a script.

To explain `getopts`, this guide will refer to the [ansible_encrypt](https://github.com/DavidVogelxyz/bin-linux/blob/master/bin-linux/ansible_encrypt) script, found within the [bin-linux](https://github.com/DavidVogelxyz/bin-linux) repo.

To understand what the command line usage of `ansible_encrypt` looks like, refer to the following valid command:

```bash
ansible_encrypt -p password-file -s "test" encrypt file.txt
```

Table of contents
-----------------

- [The main function](#the-main-function)
    - [Handling command line options](#handling-command-line-options)
    - [Running the remainder of the script](#running-the-remainder-of-the-script)
- [References](#references)

The `main` function
-----------------

First, consider the `main` function of the script:

```bash
main() {
    check_for_ansible

    while getopts "hp:s:" opt; do
        case "$opt" in
            h)
                usage
                exit 0;;
            p)
                if ! [[ -f "$OPTARG" ]]; then
                    error "[ERROR]: \`$OPTARG\` is not a file. Exiting now."
                fi

                VAULT_PASS_FILE="$OPTARG"
                ;;
            s)
                if [[ -z "$OPTARG" ]] || [[ "$OPTARG" =~ ^- ]] ; then
                    error "[ERROR]: To encrypt the file, the salt must be a valid string. Exiting now."
                fi

                SALT="$OPTARG"
                ;;
            *)
                usage >&2
                exit 1;;
        esac
    done

    # Shifts the arguments such that `$1` represents the first argument after options
    shift $((OPTIND - 1))

    MODE="$1"

    if [[ -z "$MODE" ]]; then
        usage >&2
        exit 1
    fi

    case "$MODE" in
        encrypt|decrypt)
            preflight_checks

            # Shift again, so that `$@` only contains potential files
            shift

            for arg in "$@"; do
                INPUT_FILE="$arg"
                check_input_file || continue
                "$MODE" || continue
                echo "[INFO]: \`$INPUT_FILE\` was successfully ${MODE}ed!"
            done;;
        *)
            error "[ERROR]: \"$MODE\" must be either \"encrypt\" or \"decrypt\". Exiting now.";;
    esac
}

main "$@"
```

The most important note from this snippet is that `main` is called as `main "$@"` -- this is crucial, as it passes all arguments into the `main` function. Without `$@`, the script will not be passed all of the command line options!

### Handling command line options

The `main` function will first run the `check_for_ansible` function, which will exit the script if it detects that Ansible is not installed on the system. Then, the script begins to process the command line options provided:

```bash
while getopts "hp:s:" opt; do
    case "$opt" in
        h)
            usage
            exit 0;;
        p)
            if ! [[ -f "$OPTARG" ]]; then
                error "[ERROR]: \`$OPTARG\` is not a file. Exiting now."
            fi

            VAULT_PASS_FILE="$OPTARG"
            ;;
        s)
            if [[ -z "$OPTARG" ]] || [[ "$OPTARG" =~ ^- ]] ; then
                error "[ERROR]: To encrypt the file, the salt must be a valid string. Exiting now."
            fi

            SALT="$OPTARG"
            ;;
        *)
            usage >&2
            exit 1;;
    esac
done
```

To configure `getopts`, write a `while` loop that defines the allowed options (`hp:s:`):
- any letter provided will be set as the variable `opt`:
    - this is simply a convention, and `opt` can be given any name
- any letter provided by itself will not accept an argument:
    - therefore, `h` does not accept an argument
- any letter provided with a `:` will accept an argument:
    - therefore, `p:` and `s:` will accept argument
    - the argument provided will be defined by default as `OPTARG`

Inside the `while` loop, match `opt` with a `case` statement:
- if the `h` option is provided, then the script will run `usage` and exit with a return code of `0`
- if the `p` option is provided, then the script will check if a file exists at the provided path:
    - if `OPTARG` is not a file that exists, the script will run `error`:
        - `error` prints the provided message and exits with a return code of `1`
    - if `OPTARG` is a file that exists, the script will set `VAULT_PASS_FILE` as the provided file path
- if the `s` option is provided, then the script will check the provided salt:
    - if `OPTARG` is empty, or if `OPTARG` is another command line option, then the script will run `error`:
        - `error` prints the provided message and exits with a return code of `1`
    - if `OPTARG` is a valid string, the script will set `SALT` as the provided salt
- if no valid argument (`*`) is provided, then the script will run `usage`, redirect the output to `STDERR`, and exit with a return code of `1`

### Running the remainder of the script

After handling the command line options with `getopts`, the `main` function will then run the rest of the function:

```bash
# Shifts the arguments such that `$1` represents the first argument after options
shift $((OPTIND - 1))

MODE="$1"

if [[ -z "$MODE" ]]; then
    usage >&2
    exit 1
fi

case "$MODE" in
    encrypt|decrypt)
        preflight_checks

        # Shift again, so that `$@` only contains potential files
        shift

        for arg in "$@"; do
            INPUT_FILE="$arg"
            check_input_file || continue
            "$MODE" || continue
            echo "[INFO]: \`$INPUT_FILE\` was successfully ${MODE}ed!"
        done;;
    *)
        error "[ERROR]: \"$MODE\" must be either \"encrypt\" or \"decrypt\". Exiting now.";;
esac
```

It is critical to `shift` after processing all command line options. To understand why, consider the usage example:

```bash
ansible_encrypt -p password-file -s "test" encrypt file.txt
```

When the script first runs, the first argument (`$1`) is the `-p` option. Once the command line options have been processed by `getopts`, `-p` remains as the first argument.

To resolve this, and standardize the behavior after processing the command line options, `shift` by the value of `OPTIND - 1`. In the case of the usage example, the `OPTIND` would be `5`. Therefore, the `shift` command should be instructed to shift by `4`. Now, the script will interpret the first argument as `encrypt`, which is the desired behavior.

After shifting, the script can correctly set `MODE` with the first argument, which is `encrypt`. After confirming that `MODE` is set, the script moves onto a `case` statement, matching `MODE` with either `encrypt` or `decrypt`.

Note that after `MODE` is matched, the script runs `preflight_checks`, and then runs `shift` again. As the comment states, this is done so that `$@` contains only potential files for encryption or decryption. Then, a `for` loop can be used to iterate through the remaining argument, and will perform the specified `MODE` against those files.

References
----------

- [YouTube - YouSuckAtProgramming - Crash-Course in using `getopts` to parse Command Line Arguments in Bash!](https://www.youtube.com/watch?v=wTvqFQydZQs):
    - Dave Eddy's video on using `getopts`
- [GitHub - DavidVogelxyz - bin-linux - ansible_encrypt](https://github.com/DavidVogelxyz/bin-linux/blob/master/bin-linux/ansible_encrypt):
    - Example script that makes use of `getopts`
