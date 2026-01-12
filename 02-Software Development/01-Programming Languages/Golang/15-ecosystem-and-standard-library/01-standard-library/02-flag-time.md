# Flag and Time Packages

## Summary
The `flag` package provides a built-in way to parse command-line arguments, allowing you to build CLIs that accept flags like `-port=8080` or `-verbose`. The `time` package is the standard library's solution for measuring, displaying, and manipulating time, durations, and dates, handling the complexities of timezones and monotonic clocks automatically.

## Detailed Explanation

### 1. The `flag` Package
Go favors the Plan 9 style of flags (`-flag`) over GNU style (`--flag`), though both work with the standard library (with a single hyphen).

#### Defining Flags
You define flags *before* calling `flag.Parse()`.
```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	// Method 1: Return a pointer
	wordPtr := flag.String("word", "foo", "a string")
	
	// Method 2: Bind to existing variable
	var numb int
	flag.IntVar(&numb, "numb", 42, "an int")
	
	// Method 3: Boolean flag (defaults to false)
	boolPtr := flag.Bool("fork", false, "a bool")

	// Parse arguments from os.Args[1:]
	flag.Parse()

	fmt.Println("word:", *wordPtr)
	fmt.Println("numb:", numb)
	fmt.Println("fork:", *boolPtr)
	
	// Positional arguments (leftovers)
	fmt.Println("tail:", flag.Args())
}
```
**Usage**: `go run main.go -word=opt -numb=7 arg1 arg2`

### 2. The `time` Package
Time is complex, but Go abstracts it cleanly.

#### Core Types
*   **`time.Time`**: An instant in time (nanosecond precision). Always use values, not pointers.
*   **`time.Duration`**: An elapsed time (int64 nanoseconds).
    ```go
    d := 10 * time.Second
    fmt.Println(d.Milliseconds())
    ```

#### Formatting & Parsing
Go uses a unique reference time layout: **Mon Jan 2 15:04:05 MST 2006** (1 2 3 4 5 6).
```go
t := time.Now()
// Format
fmt.Println(t.Format("2006-01-02 15:04:05")) // YYYY-MM-DD HH:MM:SS

// Parse
parsedTime, _ := time.Parse(time.RFC3339, "2023-10-01T12:00:00Z")
```

#### Timers and Tickers
*   **`time.Sleep(d)`**: Pauses the goroutine.
*   **`time.NewTicker(d)`**: Delivers ticks repeatedly on a channel (for periodic tasks).
*   **`time.After(d)`**: Returns a channel that receives once after duration (useful for timeouts in `select`).

#### Monotonic Clocks
Go's `time.Now()` contains a monotonic clock reading for measuring duration. This means `t2.Sub(t1)` is accurate even if the system wall clock changes (e.g., NTP update) between `t1` and `t2`.

## Interview Questions

**Q: Why does Go use "2006-01-02" for date formatting?**
**A:** It represents the specific reference date: January 2nd, 2006 at 3:04:05 PM (MST). The numbers correspond to the sequence 1, 2, 3, 4, 5, 6, 7 (01=Month, 02=Day, 03=Hour, 04=Min, 05=Sec, 06=Year, 07=Zone). It's intended to be easier to remember than abstract codes like `%Y-%m-%d`.

**Q: How do you implement a timeout for a function execution using the `time` package?**
**A:** You use `time.After` inside a `select` statement.
```go
select {
case res := <-resultChan:
    handle(res)
case <-time.After(3 * time.Second):
    fmt.Println("timed out")
}
```

**Q: What is the difference between `flag.String` and `flag.StringVar`?**
**A:** `flag.String` defines a flag and returns a **pointer** (`*string`) to where the value will be stored. `flag.StringVar` takes a pointer to an **existing string variable** as an argument and stores the flag value there. The latter is useful if you want to group configuration variables in a struct.
