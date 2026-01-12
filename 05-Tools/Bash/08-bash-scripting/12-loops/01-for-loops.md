---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# For Loops

## Summary

Bash for loops iterate over lists of items, ranges, or command output. They're essential for batch processing, file operations, and automation. Understanding the different forms - C-style, list iteration, and range - enables efficient scripting.

## Detailed Explanation

### List Iteration

```bash
# Iterate over words
for fruit in apple banana cherry; do
    echo "$fruit"
done

# Iterate over array
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Iterate over files (glob)
for file in *.txt; do
    echo "Processing $file"
done

# Iterate over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done
```

### Range Iteration (Brace Expansion)

```bash
# Number range
for i in {1..10}; do
    echo "$i"
done

# With step
for i in {0..100..10}; do
    echo "$i"   # 0, 10, 20, ... 100
done

# Character range
for letter in {a..z}; do
    echo "$letter"
done

# Leading zeros
for i in {01..10}; do
    echo "$i"   # 01, 02, ... 10
done
```

### C-Style For Loop

```bash
# Classic C syntax
for ((i = 0; i < 10; i++)); do
    echo "$i"
done

# Multiple variables
for ((i = 0, j = 10; i < 10; i++, j--)); do
    echo "i=$i, j=$j"
done

# Infinite loop
for ((;;)); do
    echo "Forever"
    sleep 1
done
```

### Iterating Over Lines

```bash
# WRONG: Word splitting issues
for line in $(cat file.txt); do
    echo "$line"    # Breaks on spaces
done

# RIGHT: Use while read
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Or process substitution
while IFS= read -r line; do
    echo "$line"
done < <(command)
```

### Practical Examples

```bash
# Process all files in directory
for file in /path/to/files/*; do
    if [[ -f "$file" ]]; then
        echo "File: $file"
    fi
done

# Rename files
for file in *.txt; do
    mv "$file" "${file%.txt}.md"
done

# Download multiple URLs
urls=("http://a.com" "http://b.com")
for url in "${urls[@]}"; do
    curl -O "$url"
done

# Parallel processing (background jobs)
for file in *.log; do
    process "$file" &
done
wait    # Wait for all background jobs

# Batch operations with index
files=(*.pdf)
for ((i = 0; i < ${#files[@]}; i++)); do
    echo "Processing ${files[i]} ($((i+1))/${#files[@]})"
done
```

### Loop Control

```bash
# break - exit loop
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        break
    fi
    echo "$i"
done

# continue - skip iteration
for i in {1..10}; do
    if [[ $((i % 2)) -eq 0 ]]; then
        continue    # Skip even numbers
    fi
    echo "$i"
done
```

### Nested Loops

```bash
# Matrix iteration
for i in {1..3}; do
    for j in {1..3}; do
        echo "($i, $j)"
    done
done

# Break from nested loop
for i in {1..5}; do
    for j in {1..5}; do
        if [[ $j -eq 3 ]]; then
            break 2   # Break both loops
        fi
    done
done
```

### Common Patterns

```bash
# Retry pattern
for attempt in {1..5}; do
    if command; then
        break
    fi
    echo "Attempt $attempt failed, retrying..."
    sleep 1
done

# Menu selection
options=("Option 1" "Option 2" "Quit")
for opt in "${options[@]}"; do
    echo "$opt"
done
```

## Interview Questions

**Q: What is the difference between `for i in {1..10}` and `for ((i=1; i<=10; i++))`?**
**A:** Brace expansion `{1..10}` generates a list at parse time and can't use variables. C-style `((i=1; i<=10; i++))` evaluates at runtime and supports variables for dynamic ranges.

**Q: How do you iterate over lines in a file safely?**
**A:** Use `while IFS= read -r line; do ... done < file`. The `for line in $(cat file)` approach breaks on whitespace. `IFS=` preserves leading spaces, `-r` prevents backslash interpretation.

**Q: How do you process files in parallel?**
**A:** Use `&` to background each iteration: `for f in *.txt; do process "$f" & done; wait`. The `wait` ensures all background jobs complete before continuing.
