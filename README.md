# Aeryxium's Arch Linux Shell Style Guidelines

## Introduction
What follows is my *personal* sh/bash/dash/zsh/etc style guidelines for scripting on Arch Linux. This is not intended to dictate what anyone else should do, or even what I *must* do; rather it's purpose is to serve as a repository for my own personal thoughts on style to be used by me as a reference. Having said that, I like to think my personal view are sane and the arguments valid, and would therefore be a reasonable starting point for anyone looking to find a good starting point for their own scripting.

I'll also propose the caveat that I tend not to write portable scripts; rather they tend to be designed for my system and if someone requires POSIX-compatibility, these guidelines may fall flat. Given that I rarely worry about POSIX-compatability. I have a tendancy to take advantage of the syntax available to me.

* [Bug Avoidance](#bug-avoidance)
   * [Shellcheck](#shellcheck)
   * [Shell Options](#shell-options)
   * [Return and Exit Codes](#return-and-exit-codes)
   * [Variable Expansion and Quoting](#variable-expansion-and-quoting)
   * [Variable Declaration and Assignment](#variable-declaration-and-assignment)
   * [Function Declaration](#function-declaration)
* [Output](#output)
   * [File Descriptors](#file-descriptors)
   * [Coloured Output](#coloured-output)
* [Formatting](#formatting)
   * [File Layout](#file-layout)
   * [Command Substitution](#command-substitution)
   * [Indentation and Alignment](#indentation-and-alignment)
   * [Line Length](#line-length)
   * [Pipelines](#pipelines)
* [Comments](#comments)
   * [File Headers](#file-headers)
   * [Function Header](#function-header)
   * [Section Breaks](#section-breaks)
   * [Folding](#folding)
   * [Other](#other)
* [Naming Conventions](#naming-conventions)
   * [File Names and Exensions](#file-names-and-exensions)
   * [Function Names](#function-names)
   * [Variable Names](#variable-names)
* [Loops and Conditionals](#loops-and-conditionals)
   * [Test Command](#test-command)
   * [Arithmetic Test](#arithmetic-test)
   * [If Statements](#if-statements)
   * [For Statements](#for-statements)
   * [While Statements](#while-statements)
   * [Case Statements](#case-statements)
* [References](#references)

## Bug Avoidance
#### Shellcheck
Use shellcheck to validate all scripts. Attempt to resolve issues rather than disable shell checking.

#### Shell Options
At a minimum, enable inheriting ERR trap (`-E`), exit on errors (`-e`), error on empty variable expansion (`-u`), and fail on pipe errors (`-o pipefail`). Also consider returning null on globbing failure (`shopt nullglob`) or error on globbing failure (`shopt faillglob`).

    set +Eeu -o pipefail
    shopt nullglob

#### Return and Exit Codes
Functions and scripts should return a status code if they are intended to be sourced, or an exit code if they are supposed to be executed.

#### Variable Expansion and Quoting
Quote all variables except in arithmetic test and omit the `$` except when necessary in arithmetic test. Use curly braces only when necessary. When expanding filenames using a relative path, use `./*` instead of `*`.

#### Variable Declaration and Assignment
Variables not intented to be used outside of a fuction should be declared `local`, variables intended to not be changed should be declared `readonly`. In either case, declaration and assignment should be on separate lines.

    local var1 # local variable
    readonly var2 # readonly variable
    local -r var3 # local and readonly

## Output
#### File Descriptors
Use proper file descriptors (i.e. errors should output to STDERR).

    echo "ERROR: Something didn't work." >&2

#### Coloured Output
Before using colours, query terminal capability and file descriptors to confirm compatability:

    # Verify terminal capability
    if tput colors &>/dev/null; then
        # Check STDIN
        if [ -t 1 ]; then
            # set normal colours here
        # Check STDERR
        if [ -t 2 ]; then
            # set error colours here
        fi
    fi

## Formatting
#### File Layout
Shell script files should begin with the file header, licensing if required, `set` and `shopt` statements setting options, declaration of global constants, functions, and then remainder of code. For bigger scripts, consider ending with a `main()` function called with `main "$@"` as the last line.

#### Command Substitution
For clarity, avoid backticks and prefer `$(command)` for command substitution.

#### Indentation and Alignment
Use 8-space tabs for indentation. Don't mix tabs and spaced for aligning indentation. Spaces may be user for alignment after non-whitespace characters, but should still be avoided unless it brings extra clarity. Alignment just "to look pertty" should be avoided.

#### Line Length
Lines under 80 characters is ideal. Lines over 100 characters should be examed for simplification. Lines should not exceed 120 characters.

#### Pipelines and Logical Operators
To manage line length, pipelines and logical operators should be split if they don't fit on a single line. Use an escaped newline with the pipe or logical operator at the start of the next line for clarity. Always split one per line even if multiples would fit on one line in a reasonable length for clarity:

    command1 arg1 arg2 arg3 arg4 arg5 \
        | command2 \
        | command3

## Comments
#### File Headers
All files should contain a short header with some basic information in the format of:

    #!/usr/bin/shell

    # /absolute/file/path
    #
    # Short description; idealy a single line or sentence.
    # Caveats: If any.

    # Copyright if required
    # Licensing if required

In the header, `shell` is replaced with the actual shell the script is for. If there are no caveats (i.e. a specific version of the noted shell), the final line is omitted. In cases where the script may be ported to other systems besides Arch, use `/usr/bin/env shell` instead.

A license or copyright notice should be included for any script that may at some point be distributed or used by others and not merely as a reference. My preferred license is MIT for any copyleft licensing. It should immediately follow the file header.

#### Function Header
All functions should contain a short header immediately before the actual function definiiton in the format of:

    # functionname - Short description
    # Arguments:
    #     Description of arguments one per line

In the function header, the `Arguments:` section is always included even if none are taken with the word `None` on the description line to ensure clarity.

#### Section Breaks
Section breaks should only be needed in very long, complex scripts. Consider breaking them into multiple files instead. Sections are noted by a `# --` comment. Increase the number of `--` for each subsequent nested section if required:

    # -- Main Section Name
    # ---- Secondary Section Name
    # ------ Tertiary Section Name

#### Folding
Folding should be avoided. If folding appears necessary, consider breaking things up into multiple files instad. In case folding functionality is required, use `{{{` to designate the start of a fold and `}}}` to designate the close, using the same rules as with section breaks above:

    # -- Folding Section Name {{{

    # ---- Folding Subsection Name {{{
    # ---- }}}

    # ---- Folding Subsection Name {{{
    # ---- }}}

    # -- }}}

#### Other
Use comments where appropriate to clarify obscure or confusing code. Comments shouldn't be used to explain obvious blocks, but when in doubt a comment is preferred to not. TODO comments should include a short note of what needs to be done. In the case of code being simultaneously devleoped by others, include an identifier indicating who is responsible for the TODO.

## Naming Conventions
#### File Names and Exensions
Filenames sould be in `snake_case`. Prefer no extension for files intended to be executed, or an extension indicating which interpreter they use for files intended to be sourced. For example:
* .sh for POSIX
* .bash for bash
* .dash for dash
* .zsh for zsh

In cases of convention and/or portability, bash scripts may contain the `.sh` extension instead of `.bash`.

#### Function Names
Use `snake_case` for function names which are designed for use by the user (i.e. "public" functions), and `_leading_snake_case` for function names which are designed for use solely for use by other functions (i.e. "private" or "helper" functions). For libraries, set namespaces with `::`.

    function somenamespace::some_function() {

#### Variable Names
Prefer `snake_case` for variable names and `SCREAMING_SNAKE_CASE` for constants. Constants should be declared `readonly`. Variable names should always be descriptive even when simple.

## Loops and Conditionals
#### Test Command
Double square bracket test notation is preferred to either `test` or `[ ]`, even when the latter would suffice, to protect against accidental globbing or expansion and to maintain consistency. Always space the content of the test from the brackets with a single space. Explicitly test for empty or non-empty strings with `-n` and `-z` as opposed to relying on testing logic. Expand empty variables to null to avoid errors caused by set `-u`

    if [[ -n "${somevar:-}" ]]; then
        command
    fi

#### Arithmetic Test
Use arithmetic test `(( ))` when dealing with numbers only, no quoting should be used, and variables are already dereferenced so avoid `$` unless needed.

    if (( somevar > 0 )); then

#### If Statements
Prefer `if...then` as opposed to relying on the exit status with `&&` and `||` to avoid the pitfall of chained exit codes. The only acceptable usage of exit codes is when chaining multiple commands that all depend on success. Keep `; then` on same line as preceding condition, even if conditions are split across multiple lines. `else` should be avoided at all cost; try to use better logical flow instead.

    command1 && command2 && command3
    if [[ "$somevar" = "true" &&
            "$somevar2" = "false" ]]; then
    if true; then

#### For Statements
Keep `; do` on the same line as preceeding condition, even if conditions are split across multiple lines.

    for file in "${file_list[@]}"; do

#### While Statemets
Keep `; do` on the same line as preceeding condition, even if conditions are split across multiple lines. Don't pipe into `while`, use process substitution instead.

    while read line; do
        command2
    done < <(command1)


#### Case Statements
Unless all matches for a given statement fit on one line, put the commands for each match on a new line. The ending `;;` should also be on its own line in this case.

    case "$(command)" in
        1)
            do_something
            ;;
        *)
            do_something_else
            ;;
    esac

## References
https://www.gnu.org/software/bash/manual/html_node/index.html

http://zsh.sourceforge.net/Doc/Release/index.html

https://man7.org/linux/man-pages/man1/bash.1.html

https://linux.die.net/man/1/zsh

https://mywiki.wooledge.org/BashPitfalls
