---
title: Text Manipulation
tags: [devops, bash, linux, golang, regex]
---

# Text Manipulation

## Summary
Text manipulation is a fundamental skill in DevOps, essential for parsing logs, managing configuration files, and automating system tasks. It involves using specialized CLI tools (`grep`, `sed`, `awk`, etc.) and programming libraries (like Go's `strings` and `regexp`) to search, filter, and transform plain text data. Mastery of these tools allows engineers to extract actionable insights from vast amounts of system data and build robust automation scripts.

## Detailed Explanation

### Essential CLI Commands

In the Linux/Unix world, everything is a file, and most configurations/logs are plain text. The following "Big Three" and their companions form the core of text processing.

#### 1. Search & Filter: `grep`
Used to search for patterns within text using Regular Expressions (Regex).
- **Basic Search**: `grep "error" app.log`
- **Recursive Search**: `grep -r "connection failed" /var/log/`
- **Extended Regex (ERE)**: `grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}"` (Matches IPs)
- **Invert Match**: `grep -v "INFO"` (Shows everything except INFO lines)

#### 2. Stream Editing: `sed`
A non-interactive editor used for basic text transformations.
- **Substitution**: `sed 's/localhost/127.0.0.1/g' config.yaml` (Global replace)
- **Delete Lines**: `sed '5,10d' file.txt` (Delete lines 5 through 10)
- **In-place Edit**: `sed -i 's/foo/bar/g' file.txt`

#### 3. Field Processing: `awk`
A powerful pattern scanning and processing language, ideal for columnar data.
- **Print Specific Columns**: `awk '{print $1, $4}' access.log`
- **Conditional Printing**: `awk '$9 == 404 {print $7}' access.log` (Print paths of 404 errors)
- **Custom Delimiter**: `awk -F':' '{print $1}' /etc/passwd`

#### 4. Utility Tools
- **`cut`**: Extract specific fields (`cut -d',' -f1 data.csv`).
- **`sort` & `uniq`**: Sort lines and remove duplicates. Often used together: `sort log.txt | uniq -c | sort -nr` (Count unique occurrences).
- **`wc`**: Word/Line count (`wc -l` for line count).
- **`head` & `tail`**: View the beginning or end of files. `tail -f` is critical for real-time log monitoring.

---

### Text Processing in Go

Go provides a powerful standard library for text manipulation, making it an excellent choice for building DevOps tools.

#### 1. The `strings` Package
For simple, efficient string operations without regex overhead.
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    line := "INFO: 2026/01/09: Database connection established"
    
    // Check for substring
    if strings.Contains(line, "INFO") {
        // Split by delimiter
        parts := strings.Split(line, ": ")
        fmt.Println("Message:", parts[1])
    }
}
```

#### 2. The `regexp` Package
For complex pattern matching and data extraction.
```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "My email is devops@example.com"
    // Compile regex for email
    re := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
    
    match := re.FindString(text)
    fmt.Println("Found email:", match)
}
```

#### 3. Efficient Log Processing with `bufio`
When dealing with large files, `bufio.Scanner` is preferred over `os.ReadFile` to avoid loading the entire file into memory.
```go
package main

import (
    "bufio"
    "os"
    "strings"
)

func processLogs(filePath string) error {
    file, err := os.Open(filePath)
    if err != nil {
        return err
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        line := scanner.Text()
        if strings.Contains(line, "ERROR") {
            // Process the error line
        }
    }
    return scanner.Err()
}
```

## Interview Questions

**Q: How would you count the number of unique IP addresses in an Nginx access log?**
**A:** `awk '{print $1}' access.log | sort | uniq | wc -l`. If the log is very large, using a Go script with a map to store seen IPs might be more performant and memory-efficient.

**Q: What is the difference between `grep`, `sed`, and `awk`?**
**A:** `grep` is for **searching** (finding lines that match a pattern). `sed` is for **modifying** (substituting or deleting text in a stream). `awk` is for **processing fields** and generating reports (best for structured data like logs with columns).

**Q: How do you replace all occurrences of a string in multiple files recursively?**
**A:** `grep -rl "old_string" . | xargs sed -i 's/old_string/new_string/g'`. This finds the files containing the string and then passes them to `sed` for in-place replacement.

**Q: Why should you use `bufio.Scanner` instead of `ioutil.ReadFile` for log processing in Go?**
**A:** `ioutil.ReadFile` (or `os.ReadFile`) loads the entire file into memory at once. If you're processing a 10GB log file, it will likely crash the process. `bufio.Scanner` reads the file line-by-line (or in chunks), which is memory-efficient and allows processing of files larger than the available RAM.

**Q: Write a regex to match a valid IPv4 address.**
**A:** `^((25[0-5]|(2[0-4]|1\d|[1-9]|)\d)\.?\b){4}$` (or a simpler version for basic filtering: `^([0-9]{1,3}\.){3}[0-9]{1,3}$`).
