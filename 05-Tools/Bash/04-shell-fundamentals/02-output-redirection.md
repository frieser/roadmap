---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Output Redirection

## Summary

Output redirection in Bash allows you to send a command's standard output (stdout) to a file instead of the terminal. The `>` operator creates or overwrites a file, while `>>` appends to it. Output redirection is essential for logging, saving command results, and building data pipelines.

## Detailed Explanation

### Basic Output Redirection

```bash
# Redirect stdout to file (overwrite)
echo "Hello, World!" > output.txt

# Redirect stdout to file (append)
echo "New line" >> output.txt

# Overwrite creates file if it doesn't exist
ls -la > directory_listing.txt

# Append adds to existing content
date >> log.txt
echo "Script completed" >> log.txt
```

### Redirection Operators

| Operator | Description |
|----------|-------------|
| `>` | Redirect stdout, overwrite file |
| `>>` | Redirect stdout, append to file |
| `>|` | Force overwrite (ignore noclobber) |
| `1>` | Explicit stdout redirect (same as `>`) |

### The noclobber Option

```bash
# Prevent accidental overwriting
set -o noclobber

# Now this fails if file exists
echo "test" > existing.txt
# bash: existing.txt: cannot overwrite existing file

# Force overwrite with >|
echo "test" >| existing.txt    # Works

# Disable noclobber
set +o noclobber
```

### Redirecting to /dev/null

```bash
# Discard output completely
command > /dev/null

# Common use: silent commands
if grep -q "pattern" file.txt > /dev/null; then
    echo "Found"
fi

# /dev/null is the "bit bucket" - discards everything
cat largefile.txt > /dev/null    # Reads file, discards output
```

### Creating and Truncating Files

```bash
# Create empty file (or truncate existing)
> newfile.txt

# Same as
: > newfile.txt
true > newfile.txt
truncate -s 0 newfile.txt

# Check file is empty
[[ ! -s file.txt ]] && echo "File is empty"
```

### Practical Examples

```bash
# Save command output for later
ps aux > processes.txt

# Build a file with multiple commands
{
    echo "Report generated: $(date)"
    echo "========================"
    df -h
    echo ""
    free -m
} > system_report.txt

# Log with timestamps
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> app.log
}
log "Application started"
log "Processing data..."

# Capture command in variable vs file
output=$(ls -la)     # Captures in variable
ls -la > output.txt  # Saves to file
```

### Process Substitution for Output

```bash
# Redirect output to a process (advanced)
# Some commands need a filename, not stdin
diff <(sort file1.txt) <(sort file2.txt)

# Write to multiple files simultaneously
echo "Hello" | tee file1.txt file2.txt file3.txt

# tee also writes to stdout
ls -la | tee listing.txt    # Shows output AND saves to file
```

## Interview Questions

**Q: What is the difference between `>` and `>>`?**
**A:** `>` overwrites the file (creates if doesn't exist, truncates if it does). `>>` appends to the file (creates if doesn't exist, adds to end if it does). Use `>` for fresh output, `>>` for logs.

**Q: How do you prevent accidental file overwriting?**
**A:** Use `set -o noclobber` to make `>` fail if the file exists. Override with `>|` when you intentionally want to overwrite. This protects against typos like `> important.txt`.

**Q: What is `/dev/null` and when would you use it?**
**A:** `/dev/null` is a special file that discards all data written to it. Use it to suppress unwanted output: `command > /dev/null` hides stdout. Commonly used in scripts to silence commands.

**Q: How do you redirect output to both a file and the terminal?**
**A:** Use `tee`: `command | tee output.txt` displays output and saves it. For append mode: `command | tee -a output.txt`. For stderr too: `command 2>&1 | tee output.txt`.
