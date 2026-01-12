---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Environment Variables vs Shell Variables

## Summary

Shell variables exist only in the current shell session and are not inherited by child processes. Environment variables are exported to the environment and inherited by all child processes (subshells, scripts, commands). Understanding this distinction is crucial for configuring applications, passing data between processes, and writing portable scripts.

## Detailed Explanation

### Shell Variables (Local)

```bash
# Shell variable - exists only in current shell
my_var="Hello"
count=42

# NOT available to child processes
my_var="secret"
bash -c 'echo "Child sees: $my_var"'    # Output: Child sees: (empty)

# Only visible in current shell session
echo $my_var                             # Output: secret
```

### Environment Variables (Exported)

```bash
# Create and export in one step
export MY_VAR="Hello World"

# Or export existing variable
my_var="Hello"
export my_var

# Now available to child processes
export GREETING="Hi there"
bash -c 'echo "Child sees: $GREETING"'   # Output: Child sees: Hi there

# Run command with temporary environment variable
MY_VAR="temp" ./script.sh                # MY_VAR only set for this command

# Export with declare
declare -x EXPORTED_VAR="value"
```

### Checking Variable Type

```bash
# List all environment variables
env
printenv
export -p

# List all shell variables (including environment)
set

# Check if variable is exported
declare -p MY_VAR
# Output: declare -x MY_VAR="value"  (the -x means exported)

# Check specific environment variable
printenv HOME           # Prints value if exists
echo $?                  # Exit status: 0 if found, 1 if not
```

### Common Environment Variables

```bash
# System/user paths
echo $HOME              # User's home directory
echo $PATH              # Executable search path
echo $PWD               # Current working directory
echo $OLDPWD            # Previous working directory

# User information
echo $USER              # Current username
echo $LOGNAME           # Login name
echo $UID               # User ID
echo $GROUPS            # User's groups

# Shell configuration
echo $SHELL             # User's default shell
echo $BASH              # Path to Bash binary
echo $BASH_VERSION      # Bash version string

# Locale and formatting
echo $LANG              # System locale
echo $LC_ALL            # Override all locale settings
echo $TZ                # Timezone

# Terminal
echo $TERM              # Terminal type
echo $COLUMNS           # Terminal width
echo $LINES             # Terminal height

# Editor preferences
echo $EDITOR            # Default text editor
echo $VISUAL            # Visual editor (GUI)
echo $PAGER             # Default pager (less, more)

# Temporary files
echo $TMPDIR            # Temporary directory
echo $TEMP              # Alternative temp directory
```

### Modifying PATH

```bash
# Add directory to beginning of PATH (searched first)
export PATH="/my/custom/bin:$PATH"

# Add directory to end of PATH (searched last)
export PATH="$PATH:/my/custom/bin"

# Add multiple directories
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"

# Temporary PATH for single command
PATH="/special/bin:$PATH" my_command

# In scripts, common pattern
script_dir="$(dirname "$0")"
export PATH="$script_dir:$PATH"
```

### Inheritance and Scope

```bash
# Demonstration of inheritance
export PARENT_VAR="from parent"
LOCAL_VAR="local only"

# Child process inherits exported variables only
(
    echo "Subshell PARENT_VAR: $PARENT_VAR"  # from parent
    echo "Subshell LOCAL_VAR: $LOCAL_VAR"    # (empty or inherited due to subshell)
)

# Actual child process
bash -c '
    echo "Child PARENT_VAR: $PARENT_VAR"     # from parent
    echo "Child LOCAL_VAR: $LOCAL_VAR"       # (empty)
'

# Child modifications don't affect parent
export MY_VAR="original"
bash -c 'export MY_VAR="modified"'
echo $MY_VAR                                  # Still "original"

# Subshell vs child process
# Subshell () inherits all variables but changes don't propagate back
# Child process only gets exported environment variables
```

### Unsetting Variables

```bash
# Remove shell variable
my_var="hello"
unset my_var
echo $my_var                 # (empty)

# Remove environment variable
export MY_ENV="value"
unset MY_ENV

# Remove export attribute but keep variable
export MY_VAR="value"
export -n MY_VAR             # No longer exported, still exists as shell var

# Check it's gone
declare -p MY_VAR 2>/dev/null || echo "Variable unset"
```

### Read-Only Variables

```bash
# Make variable read-only
readonly CONSTANT="unchangeable"
declare -r ALSO_CONSTANT="fixed"

# Attempting to change fails
CONSTANT="new value"         # Error: CONSTANT: readonly variable

# Read-only environment variable
readonly -x CONFIG_PATH="/etc/myapp"

# Cannot unset readonly variables
unset CONSTANT               # Error: cannot unset: readonly variable

# List readonly variables
readonly -p
```

### Setting Environment in Scripts

```bash
#!/bin/bash

# Variables defined in script are local to script by default
SCRIPT_VAR="local to script"

# Export to make available to commands run by script
export DATABASE_URL="postgres://localhost/db"

# Source another file to inherit its variables
source ./config.sh           # Variables from config.sh available here
. ./config.sh                # Equivalent shorthand

# Set environment for entire script
set -a                       # Auto-export all variables
VAR1="auto exported"
VAR2="also exported"
set +a                       # Stop auto-exporting
```

### Practical Patterns

```bash
#!/bin/bash

# Configuration with defaults
DB_HOST="${DB_HOST:-localhost}"
DB_PORT="${DB_PORT:-5432}"
DEBUG="${DEBUG:-false}"

# Required environment variable
: "${API_KEY:?Error: API_KEY environment variable is required}"

# Load from .env file if exists
if [[ -f .env ]]; then
    set -a
    source .env
    set +a
fi

# Temporary environment modification
(
    export TEMP_SETTING="value"
    ./command_that_needs_temp_setting
)
# TEMP_SETTING not visible here

# Pass secrets without command line exposure
# Bad: ./script.sh --password=secret  (visible in ps)
# Good: PASSWORD=secret ./script.sh   (safer)
```

### env Command

```bash
# Run command with modified environment
env VAR1=value1 VAR2=value2 ./script.sh

# Run command with empty environment
env -i ./script.sh

# Run with only specific variables
env -i PATH="$PATH" HOME="$HOME" ./script.sh

# Print all environment variables
env

# Print specific variable
env | grep ^PATH=

# Unset variable for command
env -u UNWANTED_VAR ./script.sh
```

## Interview Questions

### Q1: What is the difference between shell variables and environment variables?
**A:** Shell variables exist only in the current shell and are not passed to child processes. Environment variables are exported and inherited by all child processes. Use `export` to convert a shell variable to an environment variable.

### Q2: How do you make a variable available to child processes?
**A:** Use `export VAR=value` or `export VAR` after assignment. The variable becomes part of the environment and is inherited by all subsequently spawned child processes.

### Q3: If a child process modifies an environment variable, does the parent see the change?
**A:** No. Environment inheritance is one-way (parent to child). Child processes get a copy of the environment; modifications don't propagate back to the parent.

### Q4: What does `export -n` do?
**A:** It removes the export attribute from a variable without unsetting it. The variable remains as a shell variable but is no longer passed to child processes.

### Q5: How do you set an environment variable for just one command?
**A:** Prefix the command with the assignment: `VAR=value command`. The variable is set only for that command's environment and doesn't persist in the current shell.

### Q6: What is the purpose of the `env -i` command?
**A:** It runs a command with a completely empty environment, ignoring all inherited environment variables. Useful for testing scripts in clean environments or for security.

### Q7: How do you require that an environment variable be set before a script runs?
**A:** Use parameter expansion with error: `: "${VAR:?Error message}"`. This causes the script to exit with an error if VAR is unset or empty.

### Q8: What is the difference between a subshell `()` and a child process regarding variable inheritance?
**A:** A subshell `()` inherits all variables (shell and environment) from the parent but in a copy - changes don't affect the parent. A child process (like `bash -c`) only inherits exported environment variables, not shell variables.
