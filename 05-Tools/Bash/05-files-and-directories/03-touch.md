---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# touch

## Summary

The `touch` command creates empty files or updates the access/modification timestamps of existing files. It's commonly used to create placeholder files, update timestamps for make systems, and ensure files exist before operations.

## Detailed Explanation

### Basic Usage

```bash
# Create empty file (or update timestamp if exists)
touch newfile.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Create files with patterns
touch log_{1..5}.txt
# Creates: log_1.txt log_2.txt log_3.txt log_4.txt log_5.txt
```

### Timestamp Options

```bash
# -a: Change access time only
touch -a file.txt

# -m: Change modification time only
touch -m file.txt

# -t: Set specific timestamp (YYYYMMDDhhmm.ss)
touch -t 202401151200.00 file.txt

# -d: Use date string
touch -d "2024-01-15 12:00:00" file.txt
touch -d "yesterday" file.txt
touch -d "2 days ago" file.txt

# -r: Use another file's timestamp
touch -r reference.txt target.txt
```

### Common Use Cases

```bash
# Create file only if it doesn't exist
touch -c file.txt    # -c: don't create if not existing

# Create lock file
touch /var/run/myapp.lock

# Trigger make rebuild by updating source
touch src/main.c

# Create placeholder
touch README.md
```

### In Scripts

```bash
#!/bin/bash

# Ensure file exists
touch "$LOG_FILE"

# Create with proper handling
config_file="$HOME/.myapp/config"
mkdir -p "$(dirname "$config_file")"
touch "$config_file"

# Conditional creation
[[ -f "$file" ]] || touch "$file"
```

## Interview Questions

**Q: What happens when you touch an existing file?**
**A:** It updates the access and modification timestamps to the current time. The file content remains unchanged. Use `-c` to prevent creating new files.

**Q: How do you set a specific timestamp on a file?**
**A:** Use `-t YYYYMMDDhhmm.ss` or `-d "date string"`. For example: `touch -d "2024-01-15" file.txt`.
