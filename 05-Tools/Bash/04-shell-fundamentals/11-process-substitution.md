---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Process Substitution

## Summary

**Process substitution** allows a command's output to be used as a file, enabling commands that expect filenames to process dynamic data. The syntax `<(command)` creates a readable pseudo-file, and `>(command)` creates a writable one. This is a Bash-specific feature essential for comparing outputs, parallel processing, and working with programs that only accept file arguments.

## Detailed Explanation

### Basic Syntax

```bash
# Input process substitution: <(command)
# Creates a file-like object containing command output
diff <(command1) <(command2)

# Output process substitution: >(command)
# Creates a file-like object that pipes input to command
command > >(tee output.log)
```

### Input Process Substitution

```bash
# Compare outputs of two commands
diff <(ls dir1) <(ls dir2)

# Compare sorted files without creating temp files
diff <(sort file1.txt) <(sort file2.txt)

# Feed command output to program expecting file
wc -l <(grep "error" logfile.txt)

# Source dynamic script content
source <(curl -s https://example.com/script.sh)

# Multiple process substitutions
paste <(cut -f1 file1.csv) <(cut -f2 file2.csv)
```

### How It Works

```bash
# Process substitution creates a /dev/fd/N file descriptor
echo <(ls)
# /dev/fd/63

# The command runs in a subshell, output available via fd
cat <(echo "Hello from process substitution")

# It's not a real file - just a named pipe
file <(ls)
# /dev/fd/63: symbolic link to pipe:[12345]
```

### Output Process Substitution

```bash
# Write to multiple destinations
echo "Hello" | tee >(cat -n > numbered.txt) >(wc -l > count.txt)

# Log and display simultaneously
./script.sh > >(tee stdout.log) 2> >(tee stderr.log >&2)

# Process output while saving it
command | tee >(grep "error" > errors.txt) >(grep "warn" > warnings.txt)
```

### Comparison with Alternatives

```bash
# Without process substitution (using temp files)
sort file1.txt > /tmp/sorted1.txt
sort file2.txt > /tmp/sorted2.txt
diff /tmp/sorted1.txt /tmp/sorted2.txt
rm /tmp/sorted1.txt /tmp/sorted2.txt

# With process substitution (cleaner)
diff <(sort file1.txt) <(sort file2.txt)

# Without (using pipes - limited)
sort file1.txt | diff - <(sort file2.txt)
# Only stdin can be piped, not both inputs
```

### Practical Examples

```bash
# Compare directory contents
diff <(ls -la /dir1) <(ls -la /dir2)

# Compare remote and local files
diff <(ssh server "cat /etc/config") /etc/config

# Join sorted files without temp files
join <(sort file1.txt) <(sort file2.txt)

# Feed multiple files to program expecting them
paste <(seq 1 10) <(seq 11 20) <(seq 21 30)

# Read config from command
source <(echo 'export VAR="value"')

# MySQL with password from file (avoiding command line)
mysql --defaults-file=<(cat << EOF
[client]
password=secret
EOF
)
```

### Caveats

```bash
# Process substitution runs in subshell
while read line; do
    count=$((count + 1))
done < <(cat file.txt)
echo $count    # This works (not a pipe)

# But this doesn't (pipe creates subshell)
cat file.txt | while read line; do
    count=$((count + 1))
done
echo $count    # count is 0 or unset

# Not available in POSIX sh
# Only Bash, Zsh, and some others support it

# Check shell support
echo <(ls) 2>/dev/null && echo "Supported" || echo "Not supported"
```

## Interview Questions

**Q: What is the difference between process substitution and command substitution?**
**A:** Command substitution `$(cmd)` captures output as a string. Process substitution `<(cmd)` creates a file-like object that programs can read from. Use command substitution for strings/variables, process substitution when a command needs a filename.

**Q: When would you use process substitution over pipes?**
**A:** When a command needs multiple file inputs (like `diff`), or when you need to read from process output in the main shell (avoiding subshell variable scope issues with pipes), or when a program doesn't read stdin and requires a filename.

**Q: Is process substitution portable?**
**A:** No, it's a Bash-specific feature (also in Zsh and some Ksh versions). Scripts using `#!/bin/sh` or running on systems with limited shells (like Alpine's ash) won't support it.

**Q: How do you compare outputs of two commands?**
**A:** Use `diff <(command1) <(command2)`. For example, `diff <(sort file1) <(sort file2)` compares sorted versions without temp files. This is cleaner than creating intermediate files.
