---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Special Variables in Bash

## Summary

Bash provides numerous special variables (also called automatic or built-in variables) that contain information about the shell environment, script execution, positional parameters, and command results. These read-only or automatically-set variables are essential for script logic, error handling, argument processing, and accessing runtime information.

## Detailed Explanation

### Positional Parameters

```bash
#!/bin/bash
# Script called as: ./script.sh arg1 arg2 arg3

echo "Script name: $0"           # ./script.sh
echo "First argument: $1"        # arg1
echo "Second argument: $2"       # arg2
echo "Third argument: $3"        # arg3

echo "All arguments (\$*): $*"   # arg1 arg2 arg3
echo "All arguments (\$@): $@"   # arg1 arg2 arg3
echo "Number of arguments: $#"   # 3

# Difference between $* and $@
set -- "arg with spaces" "another arg"

# With double quotes - critical difference
for arg in "$*"; do
    echo "[\$*] $arg"            # [arg with spaces another arg] (one string)
done

for arg in "$@"; do
    echo "[\$@] $arg"            # [arg with spaces] then [another arg] (separate)
done

# Access arguments beyond $9
echo "${10}"                     # Braces required for 10+
echo "${15}"
```

### Exit Status Variables

```bash
# $? - Exit status of last command
ls /nonexistent 2>/dev/null
echo "Exit status: $?"           # 2 (error)

ls /tmp 2>/dev/null
echo "Exit status: $?"           # 0 (success)

# Using in conditionals
if command; then
    echo "Success (exit 0)"
else
    echo "Failed (exit $?)"
fi

# PIPESTATUS - Exit status of pipeline components
cat /etc/passwd | grep root | wc -l
echo "Pipeline statuses: ${PIPESTATUS[@]}"   # e.g., 0 0 0

false | true | false
echo "PIPESTATUS: ${PIPESTATUS[0]} ${PIPESTATUS[1]} ${PIPESTATUS[2]}"  # 1 0 1
```

### Process IDs

```bash
# $$ - Current shell's PID
echo "Current shell PID: $$"

# Useful for unique temp files
temp_file="/tmp/script.$$"
echo "Temp file: $temp_file"     # /tmp/script.12345

# $! - PID of last background job
sleep 100 &
echo "Background PID: $!"        # PID of sleep

# Wait for specific background job
sleep 5 &
bg_pid=$!
echo "Waiting for $bg_pid..."
wait $bg_pid
echo "Background job finished with status: $?"

# $PPID - Parent process ID
echo "Parent PID: $PPID"

# $BASHPID - Current subshell PID (different from $$)
echo "Shell PID: $$"
( echo "Subshell PID: $BASHPID" )  # Different PID
( echo "Subshell \$\$: $$" )       # Same as parent ($$)!
```

### Current State Variables

```bash
# $PWD - Current working directory
echo "Current directory: $PWD"

# $OLDPWD - Previous directory
cd /tmp
cd /var
echo "Previous: $OLDPWD"         # /tmp
cd -                             # Returns to $OLDPWD

# $HOME - User's home directory
echo "Home: $HOME"

# $USER / $LOGNAME - Current username
echo "User: $USER"

# $HOSTNAME - System hostname
echo "Hostname: $HOSTNAME"

# $HOSTTYPE, $OSTYPE, $MACHTYPE - System info
echo "Host type: $HOSTTYPE"      # e.g., x86_64
echo "OS type: $OSTYPE"          # e.g., linux-gnu
echo "Machine type: $MACHTYPE"   # e.g., x86_64-pc-linux-gnu

# $SECONDS - Seconds since shell started
echo "Shell running for $SECONDS seconds"

# $RANDOM - Random number 0-32767
echo "Random: $RANDOM"

# $LINENO - Current line number
echo "Line: $LINENO"
```

### String and Field Variables

```bash
# $IFS - Internal Field Separator (default: space, tab, newline)
echo "IFS: ${IFS@Q}"             # $' \t\n' (quoted representation)

# Changing IFS for parsing
data="one:two:three"
IFS=':' read -ra arr <<< "$data"
echo "${arr[@]}"                 # one two three

# Restore IFS
old_ifs="$IFS"
IFS=','
# ... do work
IFS="$old_ifs"

# Or use subshell to avoid restoration
(
    IFS=':'
    # Changes only affect subshell
)

# $REPLY - Default variable for read
read -p "Enter value: "
echo "You entered: $REPLY"
```

### Shell Options and Info

```bash
# $- - Current shell option flags
echo "Shell options: $-"         # e.g., himBHs

# Check specific options
[[ $- == *i* ]] && echo "Interactive shell"
[[ $- == *e* ]] && echo "Exit on error enabled"

# $BASH_VERSION - Bash version
echo "Bash version: $BASH_VERSION"      # e.g., 5.1.16(1)-release

# $BASH_VERSINFO - Version as array
echo "Major: ${BASH_VERSINFO[0]}"       # 5
echo "Minor: ${BASH_VERSINFO[1]}"       # 1
echo "Patch: ${BASH_VERSINFO[2]}"       # 16

# Version check
if (( BASH_VERSINFO[0] >= 4 )); then
    echo "Bash 4+ features available"
fi

# $SHLVL - Shell nesting level
echo "Shell level: $SHLVL"       # Increments with each nested shell

# $BASH_SUBSHELL - Subshell nesting level
echo "Subshell level: $BASH_SUBSHELL"
( echo "In subshell: $BASH_SUBSHELL" )  # 1
```

### Function and Source Context

```bash
# $FUNCNAME - Array of function call stack
outer() {
    inner
}

inner() {
    echo "Function: ${FUNCNAME[0]}"      # inner
    echo "Caller: ${FUNCNAME[1]}"        # outer
    echo "Full stack: ${FUNCNAME[@]}"    # inner outer main
}

outer

# $BASH_SOURCE - Array of source file names
echo "Current script: ${BASH_SOURCE[0]}"

# Get script directory reliably
script_dir="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
echo "Script directory: $script_dir"

# $BASH_LINENO - Line numbers of call stack
func() {
    echo "Called from line: ${BASH_LINENO[0]}"
}
func    # Shows line number where func was called
```

### Command-Related Variables

```bash
# $_ - Last argument of previous command
ls /tmp /var
echo "$_"                        # /var

# Also set to script path at startup
#!/bin/bash
echo "Script path: $_"           # Path used to invoke script

# $BASH_COMMAND - Currently executing command
trap 'echo "Executing: $BASH_COMMAND"' DEBUG

# COMP_* - Completion variables (for programmable completion)
# COMP_WORDS, COMP_CWORD, COMP_LINE, etc.
```

### History Variables

```bash
# $HISTFILE - History file path
echo "History file: $HISTFILE"   # ~/.bash_history

# $HISTSIZE - Commands in memory
echo "History size: $HISTSIZE"

# $HISTFILESIZE - Commands in file
echo "File size: $HISTFILESIZE"

# $HISTCMD - Current history number
echo "History number: $HISTCMD"

# !! - Previous command (not a variable, but related)
# !$ - Last argument of previous command
# !^ - First argument of previous command
```

### Practical Examples

```bash
#!/bin/bash

# Argument validation
if (( $# < 2 )); then
    echo "Usage: $0 <input> <output>" >&2
    exit 1
fi

input="$1"
output="$2"
shift 2   # Remove first two arguments, rest in $@

# Process remaining options
for opt in "$@"; do
    case "$opt" in
        --verbose) verbose=1 ;;
        --dry-run) dry_run=1 ;;
    esac
done

# Cleanup on exit
cleanup() {
    rm -f "/tmp/work.$$"
}
trap cleanup EXIT

# Error handling with line numbers
error() {
    echo "Error at line $1: $2" >&2
    exit 1
}
trap 'error $LINENO "$BASH_COMMAND"' ERR

# Process with progress
total=$#
current=0
for file in "$@"; do
    ((current++))
    echo "Processing $current/$total: $file"
done
```

## Interview Questions

### Q1: What is the difference between `$*` and `$@`?
**A:** When unquoted, they're the same. When double-quoted, `"$*"` expands to a single string with arguments joined by IFS, while `"$@"` expands to separate strings preserving each argument. Always use `"$@"` for iterating over arguments.

### Q2: What does `$?` contain?
**A:** The exit status of the last executed command. `0` typically means success, non-zero means failure. It's overwritten by each command, so save it immediately if needed later.

### Q3: How do you get the PID of the current script vs a background process?
**A:** `$$` gives the current shell's PID. `$!` gives the PID of the most recently started background process. Note: `$$` doesn't change in subshells; use `$BASHPID` for the actual subshell PID.

### Q4: What is `$PIPESTATUS` and when would you use it?
**A:** An array containing exit statuses of all commands in the last pipeline. Useful when you need to check if any stage of a pipeline failed, not just the last command (`$?`).

### Q5: How do you reliably get the directory containing the current script?
**A:** Use `"$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"`. `${BASH_SOURCE[0]}` is more reliable than `$0` when scripts are sourced.

### Q6: What does `$#` represent?
**A:** The number of positional parameters (arguments) passed to the script or function. It doesn't count `$0` (the script name).

### Q7: What is `$IFS` and why is it important?
**A:** Internal Field Separator - determines how Bash splits words. Default is space/tab/newline. Critical for parsing strings, reading files, and understanding word splitting behavior.

### Q8: How can you access the 10th argument to a script?
**A:** Use braces: `${10}`. Single digits work without braces (`$1`-`$9`), but `$10` is interpreted as `$1` followed by `0`. Always use `${n}` for clarity.
