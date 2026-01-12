---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Variable Best Practices in Bash

## Summary

Writing robust Bash scripts requires following established conventions for variable naming, quoting, initialization, and scope management. Poor variable handling is the source of most Bash bugs - from word splitting issues to injection vulnerabilities. This guide covers essential practices for professional shell scripting.

## Detailed Explanation

### Naming Conventions

```bash
# UPPERCASE: Environment variables and constants
export DATABASE_URL="postgres://localhost/db"
readonly MAX_RETRIES=3
declare -r CONFIG_PATH="/etc/myapp"

# lowercase or snake_case: Local/script variables
file_count=0
current_user="$USER"
temp_dir="/tmp/work"

# Avoid single letters except in loops
for i in {1..10}; do         # OK for loop counter
    process "$i"
done

# Descriptive names
# Bad
f="/path/to/file"
n=0
t=$(date)

# Good
input_file="/path/to/file"
retry_count=0
timestamp=$(date)

# Function-local variables: lowercase with local
process_file() {
    local file_path="$1"
    local line_count=0
    local -a results=()
}
```

### Always Quote Variables

```bash
# ALWAYS quote variable expansions
filename="my file.txt"

# Bad - word splitting and glob expansion
rm $filename           # Tries to remove "my" and "file.txt"
ls $dir/*              # Glob expands unexpectedly

# Good - preserves spaces and prevents glob
rm "$filename"         # Removes "my file.txt"
ls "$dir"/*            # Correct behavior

# Quote in conditionals
# Bad
if [ $var = "value" ]; then    # Fails if var is empty or has spaces

# Good
if [ "$var" = "value" ]; then  # Always works
if [[ $var = "value" ]]; then  # [[ handles unquoted vars, but quote anyway

# Quote in loops
# Bad
for file in $files; do         # Splits on whitespace

# Good
for file in "${files[@]}"; do  # Preserves array elements

# Quote command substitution
# Bad
result=$(command $arg)

# Good
result=$(command "$arg")
```

### Initialize Variables

```bash
#!/bin/bash

# Initialize at the top of scripts
count=0
output=""
declare -a files=()
declare -A config=()

# Use defaults for optional variables
log_level="${LOG_LEVEL:-info}"
config_file="${CONFIG_FILE:-/etc/default.conf}"
timeout="${TIMEOUT:-30}"

# Require essential variables
: "${API_KEY:?Error: API_KEY must be set}"
: "${DATABASE_URL:?Error: DATABASE_URL is required}"

# Check before use
if [[ -z "${input_file:-}" ]]; then
    echo "Error: input_file not set" >&2
    exit 1
fi
```

### Use set Options for Safety

```bash
#!/bin/bash
set -euo pipefail

# -e: Exit on error
# -u: Error on undefined variables
# -o pipefail: Catch pipeline failures

# With -u, undefined variables cause error
echo "$undefined_var"    # Script exits with error

# Use default values to handle optional vars with -u
optional="${OPTIONAL_VAR:-}"     # Empty string if unset
optional="${OPTIONAL_VAR-}"      # Same, works with -u

# Check if variable is set (with -u active)
if [[ -v MY_VAR ]]; then         # Bash 4.2+
    echo "MY_VAR is set"
fi

# Or use default pattern
if [[ -n "${MY_VAR:-}" ]]; then
    echo "MY_VAR is set and non-empty"
fi
```

### Use local in Functions

```bash
# BAD: Global variables in functions
process() {
    result="processed"           # Pollutes global namespace
    temp=$(mktemp)               # Global temp variable
}

# GOOD: Local variables in functions
process() {
    local result="processed"
    local temp
    temp=$(mktemp)
    
    echo "$result"
}

# Declare all variables at function start
parse_config() {
    local config_file="$1"
    local -a lines=()
    local line
    local key
    local value
    
    while IFS='=' read -r key value; do
        # Process
        :
    done < "$config_file"
}

# Return values via stdout, not global variables
# Bad
calculate() {
    RESULT=$((num1 + num2))      # Global side effect
}

# Good
calculate() {
    local num1="$1"
    local num2="$2"
    echo $((num1 + num2))        # Return via stdout
}
result=$(calculate 5 3)
```

### Use Braces for Clarity

```bash
# Always use braces for variable expansion
name="world"

# Bad - ambiguous
echo "$nameFile"          # Looking for $nameFile, not ${name}File

# Good - clear intent
echo "${name}File"        # worldFile
echo "${name}_suffix"     # world_suffix

# Required for arrays
arr=(one two three)
echo "${arr[0]}"          # First element
echo "${arr[@]}"          # All elements

# Required for parameter expansion
path="/path/to/file.txt"
echo "${path%.txt}"       # Remove suffix
echo "${path##*/}"        # Basename
```

### Avoid eval and Indirect Expansion

```bash
# DANGEROUS: eval with user input
user_input="value; rm -rf /"
eval "var=$user_input"           # Executes rm -rf /!

# SAFER: Use arrays or declare
# If you need dynamic variable names:

# Option 1: Associative arrays (preferred)
declare -A config
config["key1"]="value1"
config["key2"]="value2"
echo "${config[$key]}"

# Option 2: Nameref (Bash 4.3+)
set_value() {
    local -n ref="$1"
    ref="$2"
}
my_var=""
set_value my_var "new value"

# Option 3: declare with validation
name="valid_name"
if [[ $name =~ ^[a-zA-Z_][a-zA-Z0-9_]*$ ]]; then
    declare "$name=value"
fi
```

### Handle Empty and Special Values

```bash
# Check for empty
if [[ -z "$var" ]]; then
    echo "var is empty"
fi

if [[ -n "$var" ]]; then
    echo "var is not empty"
fi

# Distinguish empty from unset
if [[ -v var ]]; then           # Bash 4.2+
    echo "var is set (maybe empty)"
fi

# Handle values that look like options
filename="-dangerous"

# Bad
rm $filename             # rm interprets as option

# Good
rm -- "$filename"        # -- ends option parsing
rm "./$filename"         # Explicit path

# Handle special characters in filenames
find . -print0 | while IFS= read -r -d '' file; do
    echo "Processing: $file"
done
```

### Document Variables

```bash
#!/bin/bash
#
# Script: backup.sh
# Description: Backup specified directories
#

# =============================================================================
# Configuration (override via environment)
# =============================================================================
BACKUP_DIR="${BACKUP_DIR:-/var/backups}"    # Where to store backups
RETENTION_DAYS="${RETENTION_DAYS:-30}"      # Days to keep old backups
COMPRESS="${COMPRESS:-true}"                # Enable compression

# =============================================================================
# Constants (do not modify)
# =============================================================================
readonly SCRIPT_NAME="${0##*/}"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# =============================================================================
# Global Variables
# =============================================================================
declare -a source_dirs=()       # Directories to backup
declare -i exit_code=0          # Track overall success
declare -i file_count=0         # Number of files processed

# Function documents its expected globals
# Globals:
#   BACKUP_DIR - Destination directory
#   LOG_FILE - Log file path
# Arguments:
#   $1 - Source directory
# Returns:
#   0 on success, 1 on failure
backup_directory() {
    local source="$1"
    # ...
}
```

### Array Best Practices

```bash
# Declare explicitly
declare -a indexed_array=()
declare -A assoc_array=()

# Quote when expanding
files=("file one.txt" "file two.txt")
for file in "${files[@]}"; do       # Preserves elements
    echo "$file"
done

# Check if array is empty
if [[ ${#files[@]} -eq 0 ]]; then
    echo "No files"
fi

# Safe array building
results=()
while IFS= read -r line; do
    results+=("$line")
done < <(some_command)

# Pass arrays to functions
process_files() {
    local -a files=("$@")
    for file in "${files[@]}"; do
        echo "Processing: $file"
    done
}
process_files "${files[@]}"
```

## Interview Questions

### Q1: Why should you always quote variable expansions?
**A:** Unquoted variables undergo word splitting and pathname expansion (globbing). A filename with spaces becomes multiple arguments; `*` expands to all files. Quote to preserve literal value: `"$var"`.

### Q2: What is the convention for naming environment variables vs local variables?
**A:** Environment variables and constants use UPPERCASE (e.g., `PATH`, `CONFIG_FILE`). Local and script variables use lowercase or snake_case (e.g., `file_count`, `user_name`).

### Q3: How does `set -u` affect variable handling?
**A:** It causes the script to exit with an error when referencing undefined variables. Handle optional variables with defaults: `"${VAR:-default}"` or `"${VAR-}"` for empty string.

### Q4: Why should you use `local` in functions?
**A:** Bash variables are global by default. Without `local`, functions modify global state, causing bugs. `local` restricts variable scope to the function.

### Q5: When are braces required around variable names?
**A:** When followed by characters that could be part of the name (`${name}text`), for arrays (`${arr[0]}`), for parameter expansion (`${var#pattern}`), and for positional parameters 10+ (`${10}`).

### Q6: How do you safely handle filenames with special characters?
**A:** Always quote: `"$filename"`. Use `--` to end option parsing: `rm -- "$file"`. For find, use `-print0` with `read -d ''`. Avoid parsing `ls` output.

### Q7: What is a nameref and when would you use it?
**A:** A nameref (`local -n ref=varname`) creates a reference to another variable. Use for passing "output parameters" to functions, avoiding global variables or eval.

### Q8: How do you require an environment variable to be set?
**A:** Use parameter expansion with error: `: "${VAR:?Error message}"`. The script exits with the error message if VAR is unset or empty. Use `${VAR?msg}` if empty string is acceptable.
