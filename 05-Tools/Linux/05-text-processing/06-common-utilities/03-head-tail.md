---
tags: ['linux', 'roadmap']
---

# head and tail commands

## Summary
The `head` and `tail` commands are essential Linux utilities used to view the beginning and end of text files, respectively. They are particularly useful for inspecting large files, monitoring logs in real-time, and piping data through complex command chains. By default, both commands display 10 lines, but they offer extensive options for fine-grained control over output size and behavior.

## Detailed Explanation

### The head Command
The `head` command outputs the first part (the "head") of one or more files. It is commonly used to verify the format of a file or to extract headers.

#### Common Options
- `-n [number]`: Specify the number of lines to show.
- `-c [number]`: Specify the number of bytes to show.
- `-q` (Quiet): Do not print headers when multiple files are provided.
- `-v` (Verbose): Always print headers with file names.

#### Examples
```bash
# Show the first 10 lines (default)
head file.txt

# Show the first 5 lines
head -n 5 file.txt

# Show all but the last 10 lines
head -n -10 file.txt
```

### The tail Command
The `tail` command outputs the last part (the "tail") of one or more files. It is most famous for its ability to monitor files as they grow.

#### Common Options
- `-n [number]`: Specify the number of lines to show.
- `-c [number]`: Specify the number of bytes to show.
- `-f` (Follow): Keep the file open and output appended data as the file grows.
- `-F`: Similar to `-f`, but handles file rotation (if the file is renamed and a new one created with the same name, `tail` will switch to the new file).
- `-n +[number]`: Output starting from line `number` until the end.

#### Examples
```bash
# Show the last 10 lines (default)
tail file.txt

# Monitor a log file in real-time
tail -f /var/log/syslog

# Show starting from line 20 to the end
tail -n +20 file.txt

# Show the last 50 lines and keep following
tail -n 50 -f app.log
```

### Combining head and tail
One of the most powerful patterns is combining both to extract a specific range of lines.

```bash
# Extract lines 10 through 20 of a file
# 1. head gets the first 20 lines
# 2. tail gets the last 11 lines of those 20 (which are 10-20)
head -n 20 file.txt | tail -n 11
```

## Interview Questions

**Q: How do you monitor a log file that is being rotated (renamed) in real-time?**
**A:** Use `tail -F logfile.log`. The capital `-F` flag (or `--follow=name --retry`) ensures that `tail` keeps trying to open the file by name even if it's moved or recreated, which is common in log rotation systems.

**Q: How can you display the first 50 bytes of a file?**
**A:** Use the `-c` option with `head`: `head -c 50 file.txt`.

**Q: What does the command `tail -n +1` do?**
**A:** It displays the entire file starting from the first line. The `+` prefix in `tail -n` tells the command to start at that specific line number rather than counting from the end.

**Q: How would you display lines 5 to 10 of a file?**
**A:** `sed -n '5,10p' file.txt` is one way, but using `head` and `tail`: `head -n 10 file.txt | tail -n 6`. (Lines 5, 6, 7, 8, 9, 10 make 6 lines).

**Q: How do you output the last 15 lines of multiple files while suppressing the header names?**
**A:** Use `tail -q -n 15 file1.txt file2.txt`. The `-q` (quiet) flag hides the headers that normally appear when processing multiple files.
