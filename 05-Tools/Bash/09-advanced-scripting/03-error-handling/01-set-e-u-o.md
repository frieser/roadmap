---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# set -e -u -o pipefail (Strict Mode)

## Summary

Bash strict mode (`set -euo pipefail`) makes scripts fail fast on errors instead of continuing silently. `-e` exits on command failure, `-u` errors on undefined variables, and `-o pipefail` makes pipelines fail if any command fails. This combination catches bugs early and is considered best practice for production scripts.

## Detailed Explanation

### The Three Options

```bash
#!/bin/bash
set -e          # Exit immediately if a command exits with non-zero
set -u          # Treat unset variables as errors
set -o pipefail # Pipeline returns failure if any command fails

# Combined (most common)
set -euo pipefail
```

### set -e (errexit)

```bash
#!/bin/bash
set -e

echo "Starting"
false           # This command fails (exit code 1)
echo "This never runs"  # Script exits before this

# Commands that "fail" but shouldn't exit:
# Use || true to allow failure
potentially_failing_command || true

# Or use conditionals
if ! command; then
    echo "Command failed, continuing..."
fi
```

### set -u (nounset)

```bash
#!/bin/bash
set -u

echo "$UNDEFINED_VAR"   # Error: UNDEFINED_VAR: unbound variable

# Providing defaults
echo "${VAR:-default}"  # Use "default" if VAR unset
echo "${VAR:=default}"  # Set VAR to "default" if unset
echo "${VAR:?Error message}"  # Exit with error if unset

# Check before using
if [[ -n "${VAR:-}" ]]; then
    echo "$VAR"
fi
```

### set -o pipefail

```bash
#!/bin/bash
set -o pipefail

# Without pipefail: only last command's exit status matters
false | true | true    # Exit code: 0 (true succeeded)

# With pipefail: pipeline fails if ANY command fails
set -o pipefail
false | true | true    # Exit code: 1 (false failed)

# Real example
cat missing_file.txt | grep "pattern" | wc -l
# Without pipefail: wc succeeds, no error shown
# With pipefail: cat fails, script exits
```

### Complete Strict Mode Template

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# IFS=$'\n\t' - safer word splitting
# Default IFS includes space, which can cause issues

# Example of stricter script
main() {
    local file="${1:?Usage: $0 <filename>}"
    
    if [[ ! -f "$file" ]]; then
        echo "Error: $file not found" >&2
        exit 1
    fi
    
    process "$file"
}

main "$@"
```

### When set -e Doesn't Exit

```bash
# Commands in conditions don't trigger exit
if false; then
    echo "no"
fi

# Commands before || or &&
false || echo "Recovered"
true && echo "Continues"

# Commands in subshells (unless inherit_errexit)
(false)  # May not exit main script

# Enable for subshells (Bash 4.4+)
shopt -s inherit_errexit
```

### Disabling Temporarily

```bash
set -euo pipefail

# Disable -e temporarily
set +e
risky_command
result=$?
set -e

# Check result manually
if [[ $result -ne 0 ]]; then
    echo "Command failed with $result"
fi
```

## Interview Questions

**Q: What does `set -euo pipefail` do?**
**A:** `-e` exits on command failure, `-u` errors on undefined variables, `-o pipefail` fails pipelines on any command failure. Together, they make scripts fail-fast and catch bugs early.

**Q: How do you allow a command to fail without exiting the script?**
**A:** Use `command || true` to ignore failure, or `if ! command; then handle; fi` for explicit handling. You can also temporarily disable with `set +e`.

**Q: Why is `set -o pipefail` important?**
**A:** Without it, `cmd1 | cmd2 | cmd3` only reports cmd3's exit status. If cmd1 fails, you won't know. With pipefail, the pipeline fails if any command fails.
