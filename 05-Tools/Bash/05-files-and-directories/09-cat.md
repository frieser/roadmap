---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# cat (Concatenate)

## Summary

The `cat` command concatenates and displays file contents. Despite its name suggesting concatenation, it's most commonly used to quickly view file contents. For large files, use `less` or `head/tail` instead. Cat is also useful for creating files and combining multiple files.

## Detailed Explanation

### Basic Usage

```bash
# Display file contents
cat file.txt

# Display multiple files
cat file1.txt file2.txt

# Concatenate files into new file
cat file1.txt file2.txt > combined.txt
```

### Common Options

```bash
# -n: Number all lines
cat -n file.txt
#      1  First line
#      2  Second line

# -b: Number non-empty lines
cat -b file.txt

# -s: Squeeze multiple blank lines into one
cat -s file.txt

# -A: Show all (non-printing characters)
cat -A file.txt
# Shows tabs as ^I, line endings as $
```

### Creating Files

```bash
# Create file with content
cat > newfile.txt << 'EOF'
Line 1
Line 2
Line 3
EOF

# Append to file
cat >> existingfile.txt << 'EOF'
More content
EOF

# Create from keyboard (Ctrl+D to end)
cat > notes.txt
Type your notes here
^D
```

### Practical Examples

```bash
# View config file
cat /etc/hostname

# Combine log files
cat access_*.log > all_access.log

# Add header to file
cat header.txt data.csv > report.csv

# Insert line numbers before processing
cat -n script.sh | grep -E "^\s*[0-9]+.*error"

# Create file with line numbers
nl -ba file.txt > numbered.txt   # nl is better for numbering
```

### Alternatives for Large Files

```bash
# DON'T use cat for:
cat largefile.txt    # Dumps entire file to terminal

# DO use:
less largefile.txt   # Paginated viewing
head -20 file.txt    # First 20 lines
tail -20 file.txt    # Last 20 lines
tail -f logfile.txt  # Follow log updates
```

## Interview Questions

**Q: When should you NOT use cat?**
**A:** For large files (use `less` or `head/tail`), for reading into a loop (use `while read < file`), and in "useless use of cat" antipatterns like `cat file | grep pattern` (use `grep pattern file`).

**Q: What is "Useless Use of Cat" (UUOC)?**
**A:** Using `cat` unnecessarily in a pipeline. Example: `cat file.txt | grep pattern` should be `grep pattern file.txt`. It adds an extra process for no benefit.
