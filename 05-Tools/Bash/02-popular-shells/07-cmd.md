---
tags: ['cmd', 'shell', 'windows', 'tools', 'roadmap']
---

# CMD (Command Prompt)

## Summary

**CMD** (Command Prompt, cmd.exe) is the default command-line interpreter on Windows, descended from MS-DOS's COMMAND.COM. While limited compared to Unix shells, it remains essential for Windows system administration, batch files, and legacy compatibility. Understanding CMD is necessary when working in Windows environments, though PowerShell has become the preferred modern alternative.

## Detailed Explanation

### CMD vs Unix Shells

| Aspect | CMD | Bash |
|--------|-----|------|
| Path separator | `\` | `/` |
| Environment var | `%VAR%` | `$VAR` |
| Redirect | Same: `>`, `>>`, `<` | Same |
| Pipe | Same: `|` | Same |
| Comments | `REM` or `::` | `#` |
| Scripts | `.bat`, `.cmd` | `.sh` |
| Case sensitive | No | Yes |

### Basic CMD Commands

```batch
:: Navigation
cd \Users\name
dir                    :: List files (like ls)
dir /a                 :: Show hidden files

:: File operations
copy file1.txt file2.txt
move file.txt folder\
del file.txt           :: Delete file
mkdir newfolder
rmdir /s folder        :: Remove directory recursively

:: Environment variables
echo %PATH%
set MYVAR=value
set                    :: Show all variables

:: Help
help                   :: List commands
command /?             :: Help for specific command
```

### Batch Scripting Basics

```batch
@echo off
REM This is a comment
:: This is also a comment

REM Variables
set name=World
echo Hello, %name%!

REM Arguments
echo First arg: %1
echo All args: %*

REM Conditionals
if "%1"=="" (
    echo No arguments provided
) else (
    echo Argument: %1
)

REM Check if file exists
if exist myfile.txt (
    echo File found
)

REM Loops
for %%f in (*.txt) do (
    echo Processing %%f
)

REM Call another script
call other_script.bat

REM Exit with code
exit /b 0
```

### CMD Limitations

```batch
:: No floating-point arithmetic
set /a result=10/3    :: Result: 3 (integer only)

:: Limited string manipulation
:: No regex, no substring natively (workarounds exist)

:: No proper arrays
:: Must use numbered variables: set arr[0]=a

:: No functions (only labels and CALL)
:myfunction
    echo In function
    goto :eof

call :myfunction
```

### Useful CMD Tricks

```batch
:: Get current directory
echo %cd%

:: Get script location
echo %~dp0

:: Delayed expansion (for loops)
setlocal enabledelayedexpansion
set count=0
for %%f in (*) do (
    set /a count+=1
    echo !count!
)

:: Redirect stderr
command 2>error.log
command 2>&1          :: Merge stderr to stdout

:: Run as admin
runas /user:Administrator cmd
```

## Interview Questions

**Q: What is the difference between CMD and PowerShell?**
**A:** CMD is a simple command processor with limited scripting from DOS era. PowerShell is a modern object-oriented shell with access to .NET, better scripting, remoting, and structured data handling. Use PowerShell for new Windows automation; CMD for legacy compatibility.

**Q: How do you pass and access arguments in a batch file?**
**A:** Arguments are accessed as `%1`, `%2`, etc. `%*` gives all arguments. `%0` is the script name. Use `shift` to iterate through arguments. `%~dp0` gives the script's directory path.

**Q: What does `@echo off` do at the start of batch files?**
**A:** `echo off` prevents commands from being printed before execution (only output is shown). The `@` suppresses the echo command itself from appearing. Together they make batch scripts cleaner.
