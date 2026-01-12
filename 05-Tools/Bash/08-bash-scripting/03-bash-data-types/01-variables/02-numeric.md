---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Numeric Variables in Bash

## Summary

Bash treats all variables as strings by default, but provides mechanisms for integer arithmetic through `declare -i`, arithmetic expansion `$((...))`, and the `let` command. Bash does NOT support floating-point arithmetic natively - for decimals, external tools like `bc` or `awk` are required. Understanding these integer operations is crucial for counters, calculations, and numeric comparisons in scripts.

## Detailed Explanation

### Integer Declaration

```bash
# Standard assignment (stored as string, evaluated as integer when needed)
count=42

# Explicit integer declaration
declare -i number
number=42
number="10 + 5"              # Evaluates to 15 (arithmetic context)

# Without declare -i
regular="10 + 5"
echo "$regular"              # Output: 10 + 5 (literal string)

# With declare -i
declare -i math
math="10 + 5"
echo "$math"                 # Output: 15

# Integer attribute persists
declare -i x=5
x=x+3                        # x is now 8 (no $ needed in arithmetic)
x="2 * 4"                    # x is now 8
```

### Arithmetic Expansion $((...))

```bash
# Basic arithmetic
result=$((5 + 3))            # 8
result=$((10 - 4))           # 6
result=$((6 * 7))            # 42
result=$((20 / 4))           # 5
result=$((17 % 5))           # 2 (modulo/remainder)
result=$((2 ** 10))          # 1024 (exponentiation)

# Using variables
a=10
b=3
sum=$((a + b))               # 13
diff=$((a - b))              # 7
product=$((a * b))           # 30
quotient=$((a / b))          # 3 (integer division!)
remainder=$((a % b))         # 1

# Compound expressions
result=$(( (a + b) * 2 ))    # 26
result=$(( a > b ? a : b ))  # 10 (ternary operator)

# Increment and decrement
count=5
((count++))                  # 6 (post-increment)
((count--))                  # 5 (post-decrement)
((++count))                  # 6 (pre-increment)
((--count))                  # 5 (pre-decrement)
((count += 10))              # 15
((count -= 5))               # 10
((count *= 2))               # 20
((count /= 4))               # 5
```

### The let Command

```bash
# Basic usage
let result=5+3               # 8
let "result = 5 + 3"         # 8 (quotes allow spaces)

# Multiple assignments
let a=5 b=10 c=a+b           # a=5, b=10, c=15

# Increment/decrement
let count++
let count--
let "count += 5"

# Comparison (returns exit status)
let "5 > 3"                  # Exit status 0 (true)
let "5 < 3"                  # Exit status 1 (false)

# Use in conditionals
if let "count > 10"; then
    echo "Count exceeds 10"
fi
```

### Integer Division and Modulo

```bash
# Integer division truncates (no rounding)
echo $((7 / 2))              # 3 (not 3.5)
echo $((7 / 3))              # 2 (not 2.33)
echo $((-7 / 2))             # -3

# Modulo for remainder
echo $((7 % 2))              # 1
echo $((10 % 3))             # 1
echo $((15 % 5))             # 0

# Check if number is even or odd
number=42
if (( number % 2 == 0 )); then
    echo "Even"
else
    echo "Odd"
fi

# Get last digit
echo $((12345 % 10))         # 5
```

### Number Bases

```bash
# Decimal (default)
dec=42

# Octal (prefix with 0)
oct=052                      # 42 in decimal
echo $((oct))                # 42

# Hexadecimal (prefix with 0x or 0X)
hex=0x2A                     # 42 in decimal
echo $((hex))                # 42

# Binary (Bash 2.04+, prefix with 0b)
# Note: Not all Bash versions support 0b
bin=2#101010                 # 42 in decimal (base#number syntax)

# Any base from 2 to 64
base2=$((2#1010))            # 10 in decimal
base8=$((8#52))              # 42 in decimal
base16=$((16#2A))            # 42 in decimal

# Convert decimal to hex (using printf)
printf "%x\n" 42             # 2a
printf "%X\n" 42             # 2A

# Convert decimal to octal
printf "%o\n" 42             # 52
```

### Floating-Point with bc

```bash
# bc for floating-point arithmetic
result=$(echo "5.5 + 3.2" | bc)          # 8.7
result=$(echo "10 / 3" | bc -l)          # 3.33333333333333333333

# Scale (decimal places)
result=$(echo "scale=2; 10 / 3" | bc)    # 3.33

# Complex calculations
result=$(echo "scale=4; sqrt(2)" | bc -l) # 1.4142

# Using variables
a=5.5
b=2.3
sum=$(echo "$a + $b" | bc)               # 7.8
product=$(echo "$a * $b" | bc)           # 12.65

# Comparison with bc
if (( $(echo "$a > $b" | bc -l) )); then
    echo "$a is greater than $b"
fi
```

### Floating-Point with awk

```bash
# awk for floating-point
result=$(awk "BEGIN {print 5.5 + 3.2}")      # 8.7
result=$(awk "BEGIN {printf \"%.2f\", 10/3}") # 3.33

# Using shell variables in awk
a=5.5
b=2.3
sum=$(awk -v x="$a" -v y="$b" 'BEGIN {print x + y}')

# Math functions
sqrt=$(awk "BEGIN {print sqrt(2)}")          # 1.41421
sin=$(awk "BEGIN {print sin(3.14159/2)}")    # ~1
log=$(awk "BEGIN {print log(10)}")           # 2.30259
```

### Random Numbers

```bash
# Built-in $RANDOM (0-32767)
echo $RANDOM                 # Random number 0-32767

# Random in range [0, N)
max=100
echo $((RANDOM % max))       # 0-99

# Random in range [min, max]
min=10
max=50
echo $((RANDOM % (max - min + 1) + min))   # 10-50

# Seed the random generator
RANDOM=42                    # Reproducible sequence

# Better randomness with /dev/urandom
random_num=$(od -An -tu4 -N4 /dev/urandom | tr -d ' ')
echo $((random_num % 100))   # 0-99 with better distribution

# shuf for random selection
echo $(shuf -i 1-100 -n 1)   # Random 1-100
```

### Practical Examples

```bash
#!/bin/bash

# Counter loop
for ((i=1; i<=10; i++)); do
    echo "Iteration $i"
done

# Sum of numbers
sum=0
for num in 1 2 3 4 5; do
    ((sum += num))
done
echo "Sum: $sum"             # 15

# Factorial
factorial() {
    local n=$1
    local result=1
    for ((i=2; i<=n; i++)); do
        ((result *= i))
    done
    echo $result
}
factorial 5                  # 120

# Percentage calculation
total=250
completed=175
percent=$(( (completed * 100) / total ))
echo "Progress: ${percent}%"  # 70%

# File size in human-readable format
bytes=1536000
kb=$((bytes / 1024))
mb=$((bytes / 1024 / 1024))
echo "${kb} KB or ${mb} MB"
```

## Interview Questions

### Q1: Does Bash support floating-point arithmetic natively?
**A:** No. Bash only supports integer arithmetic. For floating-point operations, use external tools like `bc` (e.g., `echo "5.5 + 3.2" | bc`) or `awk` (e.g., `awk 'BEGIN {print 5.5 + 3.2}'`).

### Q2: What is the difference between `$((expression))` and `let`?
**A:** Both perform arithmetic. `$((...))` is arithmetic expansion that returns a value, usable in assignments or command substitution. `let` is a command that evaluates expressions and returns exit status. `$((...))` is preferred in modern scripts.

### Q3: How does integer division work in Bash?
**A:** Integer division truncates toward zero, discarding any decimal portion. `$((7 / 2))` equals `3`, not `3.5`. Use `bc` or `awk` for precise division.

### Q4: What does `declare -i` do?
**A:** It declares a variable with the integer attribute. Any string assigned to it is evaluated as an arithmetic expression. For example, after `declare -i x`, assigning `x="5+3"` stores `8`, not the literal string.

### Q5: How do you generate a random number in a specific range?
**A:** Use `$((RANDOM % (max - min + 1) + min))`. For range 1-100: `$((RANDOM % 100 + 1))`. Note that `$RANDOM` has limited range (0-32767) and isn't cryptographically secure.

### Q6: How do you work with hexadecimal numbers in Bash?
**A:** Prefix with `0x`: `hex=0x2A`. For conversion, use `printf "%x" decimal` (to hex) or `$((0xFF))` (hex to decimal). Also works with `base#number` syntax: `$((16#FF))`.

### Q7: What is the modulo operator and when would you use it?
**A:** The modulo operator `%` returns the remainder of integer division. Common uses: checking even/odd (`n % 2`), cycling through values, extracting digits, and implementing wrap-around logic.

### Q8: How do you increment a variable in Bash?
**A:** Multiple ways: `((count++))`, `((count+=1))`, `let count++`, `count=$((count+1))`, or with `declare -i count; count=count+1`. The `((...))` forms are most common and readable.
