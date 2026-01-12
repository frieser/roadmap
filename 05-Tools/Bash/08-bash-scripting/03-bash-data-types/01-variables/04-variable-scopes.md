---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Variable Scopes in Bash

## Summary

Bash has three main variable scopes: global (default for all variables), local (inside functions using `local`), and environment (exported, inherited by child processes). Unlike many programming languages, Bash variables are global by default, which can lead to unintended side effects. Understanding scope is essential for writing maintainable functions and avoiding variable collision bugs.

## Detailed Explanation

### Global Scope (Default)

```bash
# All variables are global by default
count=0
name="global"

increment() {
    count=$((count + 1))     # Modifies global count
    name="modified"          # Modifies global name
}

echo "Before: count=$count, name=$name"   # 0, global
increment
echo "After: count=$count, name=$name"    # 1, modified

# This can cause bugs
process_file() {
    file="processing.txt"    # Accidentally overwrites global $file
    # ... do work
}

file="important.txt"
process_file
echo "File: $file"           # Oops! Now "processing.txt"
```

### Local Scope (Functions)

```bash
# Use 'local' to restrict variable to function
my_function() {
    local my_var="local value"
    local count=100
    echo "Inside: my_var=$my_var, count=$count"
}

my_var="global value"
count=0

my_function                  # Inside: my_var=local value, count=100
echo "Outside: my_var=$my_var, count=$count"   # global value, 0

# Local declaration syntax options
func() {
    local var1="value"       # Most common
    local var2 var3          # Declare multiple
    local -i num=42          # Local integer
    local -r const="fixed"   # Local readonly
    local -a arr=(1 2 3)     # Local array
}
```

### Local Variables in Nested Functions

```bash
outer() {
    local outer_var="outer"
    
    inner() {
        local inner_var="inner"
        echo "Inner sees outer_var: $outer_var"     # outer
        echo "Inner sees inner_var: $inner_var"     # inner
        outer_var="modified by inner"               # Modifies outer's local!
    }
    
    inner
    echo "After inner: outer_var=$outer_var"        # modified by inner
}

outer
echo "Global outer_var: $outer_var"                 # (empty - was local)
```

### Dynamic Scoping

```bash
# Bash uses dynamic scoping, not lexical scoping
# Inner functions see caller's local variables

level1() {
    local var="level1"
    level2
}

level2() {
    local var="level2"
    level3
}

level3() {
    # Sees level2's local var, not level1's
    echo "level3 sees: $var"       # level2
}

level1
```

### Function Parameters and Scope

```bash
# Function parameters are local (positional parameters)
greet() {
    # $1, $2, etc. are local to this function
    echo "Hello, $1!"
    
    # But named variables need explicit 'local'
    local message="Welcome"
    echo "$message $2"
}

# Global positional parameters are separate
set -- "global1" "global2"
greet "Alice" "Bob"
echo "Global \$1: $1"        # global1 (unchanged)
```

### Environment Scope

```bash
# Environment variables are inherited by child processes
export GLOBAL_CONFIG="/etc/app"

run_child() {
    local LOCAL_VAR="not inherited"
    export EXPORTED_VAR="inherited"
    
    # Child process sees GLOBAL_CONFIG and EXPORTED_VAR
    bash -c 'echo "Config: $GLOBAL_CONFIG, Exported: $EXPORTED_VAR, Local: $LOCAL_VAR"'
    # Output: Config: /etc/app, Exported: inherited, Local: (empty)
}

run_child
```

### Subshell Scope

```bash
# Subshells get copies of all variables (local and global)
var="original"
count=0

# Subshell - changes don't propagate back
(
    var="modified in subshell"
    count=100
    echo "Subshell: var=$var, count=$count"   # modified, 100
)
echo "Parent: var=$var, count=$count"         # original, 0

# Common subshell triggers
echo "hello" | while read line; do
    count=$((count + 1))    # This is in a subshell!
done
echo "Count: $count"        # Still 0! (Bash < 4.2)

# Fix with lastpipe or process substitution
shopt -s lastpipe           # Bash 4.2+
echo "hello" | while read line; do
    count=$((count + 1))
done
echo "Count: $count"        # Now 1
```

### Avoiding Subshell Scope Issues

```bash
# Problem: pipe creates subshell
count=0
cat file.txt | while read line; do
    ((count++))
done
echo $count    # 0 (subshell modification lost)

# Solution 1: Process substitution
count=0
while read line; do
    ((count++))
done < <(cat file.txt)
echo $count    # Correct count

# Solution 2: Here-string
count=0
while read line; do
    ((count++))
done <<< "$(cat file.txt)"
echo $count    # Correct count

# Solution 3: Redirect from file directly
count=0
while read line; do
    ((count++))
done < file.txt
echo $count    # Correct count
```

### Declare and Local Options

```bash
# declare inside function is local by default
func() {
    declare inner="local"       # Same as local
    declare -g global="global"  # Force global scope
    
    echo "inner: $inner"
}

func
echo "inner outside: $inner"    # (empty)
echo "global outside: $global"  # global

# Scope modifiers with declare
declare -g VAR="force global"   # Even inside function
local -n ref=other_var          # Local nameref
```

### Nameref and Scope

```bash
# Nameref allows reference to another variable's scope
set_value() {
    local -n ref=$1             # Reference to passed variable name
    ref="new value"
}

my_var="original"
set_value my_var
echo "$my_var"                  # new value

# Useful for returning values from functions
get_data() {
    local -n result=$1
    result="computed data"
}

output=""
get_data output
echo "$output"                  # computed data
```

### Best Practices

```bash
#!/bin/bash

# 1. Always use local in functions
process_data() {
    local input="$1"
    local temp
    local result
    
    temp="${input^^}"
    result="Processed: $temp"
    echo "$result"
}

# 2. Initialize variables at appropriate scope
declare -g GLOBAL_CONFIG="/etc/myapp.conf"

main() {
    local file_count=0
    local -a files=()
    
    # Use global config, modify local state
    source "$GLOBAL_CONFIG"
    
    for f in *.txt; do
        files+=("$f")
        ((file_count++))
    done
    
    echo "Found $file_count files"
}

# 3. Document expected globals
# Globals: CONFIG_PATH, DEBUG_MODE
initialize() {
    : "${CONFIG_PATH:?CONFIG_PATH must be set}"
    local debug="${DEBUG_MODE:-false}"
    # ...
}

# 4. Use namerefs for "output parameters"
compute() {
    local -n output_ref=$1
    local input=$2
    
    output_ref=$((input * 2))
}

result=0
compute result 21
echo "$result"                  # 42
```

## Interview Questions

### Q1: Are Bash variables global or local by default?
**A:** Global by default. Variables assigned anywhere (including inside functions) are global unless explicitly declared with `local`. This is a common source of bugs.

### Q2: How do you create a local variable in a function?
**A:** Use the `local` keyword: `local var="value"`. The variable is then scoped to that function and its called functions (dynamic scoping).

### Q3: What is dynamic scoping in Bash?
**A:** Functions see local variables from their callers, not from where they were defined. If function A calls B, and A has `local x`, then B can access and modify `x` even without its own declaration.

### Q4: Why does modifying a variable in a pipe not affect the original?
**A:** Pipes create subshells, and each pipeline segment runs in a separate subshell. Variables modified in subshells don't affect the parent. Fix with process substitution `< <(cmd)` or `shopt -s lastpipe`.

### Q5: What does `declare -g` do inside a function?
**A:** It forces the variable to be global scope even when used inside a function. Without `-g`, `declare` inside a function creates a local variable.

### Q6: What is a nameref and how does it relate to scope?
**A:** A nameref (`local -n` or `declare -n`) creates a reference to another variable by name. It allows functions to modify variables in the caller's scope, providing a way to return multiple values.

### Q7: What happens to local variables when a function ends?
**A:** They are destroyed. Local variables only exist during the function's execution and are not accessible afterward, unlike global variables which persist.

### Q8: How can you prevent a function from accidentally modifying global state?
**A:** Declare all variables inside functions with `local`. Use namerefs for intentional output. Consider using subshells `()` for complete isolation. Always initialize variables before use.
