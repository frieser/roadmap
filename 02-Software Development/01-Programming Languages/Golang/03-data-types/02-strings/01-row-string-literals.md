#Golang
---
---

## Summary

Raw string literals in Go are enclosed in backticks (`` ` ``) and preserve the exact content including newlines, tabs, and backslashes without interpretation. They are ideal for multi-line strings, regular expressions, SQL queries, JSON templates, and any content containing many special characters that would otherwise require escaping.

## Detailed Explanation

### **Basic Syntax**

```go
package main

import "fmt"

func main() {
    // Raw string literal with backticks
    raw := `This is a raw string literal`
    
    // Multi-line - newlines are preserved
    multiline := `Line 1
Line 2
Line 3`
    
    fmt.Println(raw)
    fmt.Println(multiline)
}
```

### **No Escape Sequence Processing**

```go
func main() {
    // Interpreted string: escapes processed
    interpreted := "Path: C:\\Users\\name\\file.txt"
    
    // Raw string: backslashes are literal
    raw := `Path: C:\Users\name\file.txt`
    
    fmt.Println(interpreted)  // Path: C:\Users\name\file.txt
    fmt.Println(raw)          // Path: C:\Users\name\file.txt
    
    // Tab and newline characters
    interpretedTab := "Column1\tColumn2\nRow1\tRow2"
    rawTab := `Column1\tColumn2\nRow1\tRow2`
    
    fmt.Println(interpretedTab)
    // Column1    Column2
    // Row1       Row2
    
    fmt.Println(rawTab)
    // Column1\tColumn2\nRow1\tRow2 (literal backslashes)
}
```

### **Common Use Cases**

#### Regular Expressions

```go
import "regexp"

func main() {
    // Without raw strings: escape nightmare
    patternInterpreted := "\\d{3}-\\d{3}-\\d{4}"
    
    // With raw strings: clean and readable
    patternRaw := `\d{3}-\d{3}-\d{4}`
    
    re := regexp.MustCompile(patternRaw)
    
    phone := "123-456-7890"
    fmt.Println(re.MatchString(phone))  // true
    
    // Complex regex
    emailPattern := `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
    emailRe := regexp.MustCompile(emailPattern)
    fmt.Println(emailRe.MatchString("test@example.com"))  // true
}
```

#### SQL Queries

```go
func main() {
    query := `
        SELECT 
            u.id,
            u.name,
            u.email
        FROM users u
        WHERE u.status = 'active'
            AND u.created_at > $1
        ORDER BY u.created_at DESC
        LIMIT 100
    `
    
    fmt.Println(query)
    // Executes with proper formatting preserved
}
```

#### JSON Templates

```go
func main() {
    jsonTemplate := `{
    "name": "{{.Name}}",
    "email": "{{.Email}}",
    "active": {{.Active}}
}`
    
    fmt.Println(jsonTemplate)
}
```

#### HTML Templates

```go
func main() {
    html := `<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>
<body>
    <h1>Welcome, {{.Name}}!</h1>
</body>
</html>`
    
    fmt.Println(html)
}
```

#### Configuration Files

```go
func main() {
    config := `
# Server Configuration
server:
  host: localhost
  port: 8080

# Database Configuration  
database:
  driver: postgres
  connection: "host=localhost port=5432 dbname=myapp"
`
    
    fmt.Println(config)
}
```

### **Preserving Whitespace**

```go
func main() {
    // All whitespace is preserved exactly
    indented := `
        This line is indented with 8 spaces.
    This line is indented with 4 spaces.
No indentation here.
`
    
    fmt.Println(indented)
    // Outputs with exact whitespace preserved
    
    // Common pattern: strip leading/trailing whitespace
    import "strings"
    cleaned := strings.TrimSpace(indented)
    fmt.Println(cleaned)
}
```

### **Limitations**

```go
func main() {
    // Cannot include backtick in raw string
    // invalid := `This has a ` backtick`  // Syntax error!
    
    // Solution 1: Use interpreted string
    withBacktick := "This has a ` backtick"
    
    // Solution 2: Concatenate
    withBacktick2 := `This has a ` + "`" + ` backtick`
    
    // Solution 3: Use string formatting
    withBacktick3 := fmt.Sprintf("This has a %c backtick", '`')
    
    fmt.Println(withBacktick)
    fmt.Println(withBacktick2)
    fmt.Println(withBacktick3)
}
```

### **Comparison: Raw vs Interpreted**

| Feature | Raw String (`` ` ``) | Interpreted String (`"`) |
| --- | --- | --- |
| Escape sequences | Not processed | Processed |
| Multi-line | Allowed | Not allowed |
| Backtick | Cannot include | Can include (`\``) |
| Newlines | Literal | Must use `\n` |
| Tabs | Literal | Must use `\t` |
| Backslashes | Literal | Must escape (`\\`) |

### **Practical Examples**

#### CLI Help Text

```go
func printHelp() {
    help := `
Usage: myapp [OPTIONS] COMMAND

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -c, --config    Path to config file

Commands:
    start           Start the server
    stop            Stop the server
    status          Show server status

Examples:
    myapp start --config /etc/myapp.yaml
    myapp status --verbose
`
    fmt.Print(help)
}
```

#### Heredoc-Style Strings

```go
func main() {
    // Similar to heredoc in other languages
    script := `#!/bin/bash
set -e

echo "Starting deployment..."
docker-compose up -d
echo "Deployment complete!"
`
    
    fmt.Println(script)
}
```

#### ASCII Art

```go
func main() {
    gopher := `
    ʕ◔ϖ◔ʔ
   /|    |\
  (_|    |_)
`
    fmt.Println(gopher)
}
```

### **Best Practices**

```go
// ✓ Good: Use raw strings for regex
var emailRegex = regexp.MustCompile(`^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$`)

// ✓ Good: Use raw strings for SQL
const selectUsers = `SELECT * FROM users WHERE status = $1`

// ✓ Good: Use raw strings for multi-line text
const helpMessage = `Usage: command [options]

Options:
  -h  Show help
  -v  Verbose output`

// ✗ Avoid: Using interpreted strings for regex (escape hell)
var badRegex = regexp.MustCompile("^[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,}$")

// ✗ Avoid: Using raw strings for simple single-line strings
var greeting = `Hello`  // Just use "Hello"
```

## Interview Questions

**Q: What is a raw string literal in Go?**
**A:** A raw string literal is enclosed in backticks (`` ` ``) and preserves all content literally without processing escape sequences. Newlines, tabs, and backslashes appear exactly as written. They're ideal for regex, SQL, multi-line text, and file paths.

**Q: What is the main limitation of raw string literals?**
**A:** You cannot include a backtick character inside a raw string literal. To include a backtick, you must either use an interpreted string (`"has ` + "`" + ` inside"`) or concatenate a raw string with an interpreted one containing the backtick.

**Q: When should you prefer raw strings over interpreted strings?**
**A:** Use raw strings for: (1) Regular expressions (no double-escaping backslashes), (2) SQL queries (readability), (3) Multi-line content (JSON, HTML, config templates), (4) File paths with backslashes, (5) Any content with many special characters.

**Q: Do raw strings process Unicode escape sequences like `\u0041`?**
**A:** No. Raw strings preserve content literally, so `\u0041` appears as the six characters `\`, `u`, `0`, `0`, `4`, `1` rather than the letter "A". For Unicode escapes, use interpreted strings or rune literals.
