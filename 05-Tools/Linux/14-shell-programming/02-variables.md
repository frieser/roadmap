---
tags: ['linux', 'roadmap']
---

# Shell Variables (User-defined, System, Parameter Expansion)

## Summary
Shell variables are named locations in memory used to store data that can be accessed and manipulated by the shell and scripts. They are central to scripting, allowing for dynamic behavior, environment configuration, and data processing. Bash variables are untyped (primarily strings) but can be treated as integers in specific contexts.

## Detailed Explanation

### 1. User-Defined Variables
Users can create their own variables for use within a shell session or script.

#### Assignment and Referencing
- **Assignment**: Use the `=` operator without spaces.
- **Referencing**: Use the `$` prefix or `${}` for clarity.

```bash
name="John"       # Correct assignment
# name = "John"   # ERROR: Shell looks for command 'name'
echo $name        # Output: John
echo "${name}ny"  # Output: Johnny (braces prevent ambiguity)
```

#### Local vs. Environment Variables
- **Local Variables**: Only available in the current shell.
- **Environment Variables**: Available to the current shell and all child processes/commands.
- **`export`**: Promotes a local variable to an environment variable.

```bash
CITY="New York"
export CITY       # Now available to scripts/commands started from this shell
```

### 2. System and Environment Variables
The system pre-defines several environment variables to store configuration and state.

| Variable | Description |
| :--- | :--- |
| `PATH` | List of directories the shell searches for executable files. |
| `HOME` | The current user's home directory. |
| `USER` | The name of the logged-in user. |
| `SHELL` | Path to the current user's default shell. |
| `PWD` | The current working directory. |
| `PS1` | The primary prompt string (defines the look of the command prompt). |

### 3. Special Variables
These variables provide information about the script's execution environment.

- `$0`: Name of the script being executed.
- `$1`, `$2`, ... `$n`: Positional parameters (arguments passed to the script).
- `$#`: Number of positional parameters.
- `$@`: All positional parameters (quoted individually: `"$1" "$2"...`).
- `$*`: All positional parameters (quoted as a single string: `"$1 $2..."`).
- `$?`: Exit status of the last executed command (0 for success, non-zero for failure).
- `$$`: Process ID (PID) of the current shell.
- `$!`: PID of the last background command.

### 4. Parameter Expansion
Parameter expansion allows for sophisticated manipulation of variable values.

#### Default Values
- `${VAR:-default}`: Use `default` if `VAR` is unset or null, but don't assign it.
- `${VAR:=default}`: Use `default` and **assign** it to `VAR` if it was unset or null.
- `${VAR:?error}`: Display `error` and exit if `VAR` is unset or null.

#### Substring and Length
- `${#VAR}`: Returns the length of the string.
- `${VAR:offset:length}`: Extracts a substring.

```bash
text="Hello World"
echo ${#text}        # Output: 11
echo ${text:6:5}     # Output: World
```

#### Pattern Removal
- `${VAR#pattern}`: Remove shortest match of pattern from the beginning.
- `${VAR##pattern}`: Remove longest match of pattern from the beginning.
- `${VAR%pattern}`: Remove shortest match of pattern from the end.
- `${VAR%%pattern}`: Remove longest match of pattern from the end.

```bash
path="/home/user/script.sh"
echo ${path#*/}      # Output: home/user/script.sh
echo ${path##*/}     # Output: script.sh (Handy for 'basename')
echo ${path%/*}      # Output: /home/user (Handy for 'dirname')
```

#### Search and Replace
- `${VAR/pattern/string}`: Replace first match.
- `${VAR//pattern/string}`: Replace all matches.

```bash
msg="I like cats and cats"
echo ${msg/cats/dogs}  # Output: I like dogs and cats
echo ${msg//cats/dogs} # Output: I like dogs and dogs
```

## Interview Questions

1. **What is the difference between `$@` and `$*`?**
   - When double-quoted, `"$@"` expands each argument as a separate word (`"arg1" "arg2"`), preserving spaces within arguments. `"$*"` expands all arguments into a single word separated by the first character of `IFS` (`"arg1 arg2"`).

2. **How do you check if a variable is empty or unset in a script?**
   - Using the `-z` flag in a conditional: `if [ -z "$VAR" ]; then echo "Empty"; fi`. Alternatively, use `${VAR:?error message}` to terminate if it's empty.

3. **How do you make a variable "read-only"?**
   - Use the `readonly` command: `readonly MY_VAR="fixed_value"`. Any attempt to change it later will result in an error.

4. **How can you get the directory name and filename from a full path using only parameter expansion?**
   - Filename: `${FULL_PATH##*/}` (removes everything up to the last slash).
   - Directory: `${FULL_PATH%/*}` (removes everything from the last slash onwards).

5. **What does `export` actually do?**
   - It marks the variable to be passed to the environment of subsequently executed child processes. Without `export`, a variable is only local to the current shell process.
