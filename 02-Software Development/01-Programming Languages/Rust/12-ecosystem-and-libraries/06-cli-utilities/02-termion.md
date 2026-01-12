# Termion
---
---

## Summary
Termion is a pure-Rust, bindless library for low-level terminal manipulation. Unlike higher-level libraries (like `crossterm` or `ncurses`), Termion focuses on being a lightweight wrapper around ANSI escape codes for Unix-like systems.

## Detailed Explanation

### Core Philosophy
Termion is "Unix-only" and "Pure Rust." It avoids C bindings, making it easy to build. It provides the building blocks (raw mode, cursor movement, colors, events) needed to build Text User Interfaces (TUIs) or interactive CLI tools.

### Key Features
*   **Raw Mode**: Switch the terminal to raw mode to process input byte-by-byte (essential for games/editors).
*   **ANSI Escape Codes**: Helpers to move the cursor, clear screens, and set colors.
*   **Async Input**: Iterators for handling key events.

### Use Cases
*   **Text Editors**: Building a Vim clone in Rust.
*   **CLI Games**: Snake, Tetris in the terminal.
*   **System Monitors**: Tools like `htop`.

### Code Example
*Dependencies: `termion`*

```rust
use std::io::{Write, stdout};
use termion::raw::IntoRawMode;
use termion::{color, clear, cursor};

fn main() {
    // Enter raw mode
    let mut stdout = stdout().into_raw_mode().unwrap();

    write!(stdout, "{}{}", clear::All, cursor::Goto(1, 1)).unwrap();
    write!(stdout, "{}Hello from Raw Mode!", color::Fg(color::Red)).unwrap();
    
    // Reset style
    write!(stdout, "{}", color::Fg(color::Reset)).unwrap();
    
    stdout.flush().unwrap();
}
```

## Interview Questions

1.  **Q: Why is Termion Unix-only?**
    *   **A:** Termion is built specifically around the Unix TTY architecture and standard ANSI escape codes. It does not abstract away the differences between Unix terminals and the Windows Console API. For cross-platform support (including Windows), libraries like `crossterm` are preferred.

2.  **Q: What is "Raw Mode" and why is it needed?**
    *   **A:** By default, terminals are in "Canonical Mode" (or Cooked Mode), where input is buffered until the user hits Enter. Raw Mode disables this buffering and local echoing, allowing the program to receive every keypress immediately. This is required for interactive apps like editors or games.

3.  **Q: How does Termion handle input events?**
    *   **A:** It provides an `events()` iterator on the Stdin stream (or `keys()` for simple key presses). This allows you to loop over input events cleanly.
