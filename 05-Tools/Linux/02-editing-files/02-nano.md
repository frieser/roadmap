#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
**GNU Nano** is a lightweight, command-line text editor for Unix-like systems, designed for simplicity and ease of use. Unlike Vim or Emacs, it provides a "modeless" editing experience where you can type directly into the buffer, and it displays a helpful shortcut menu at the bottom of the screen. It is the go-to editor for quick configuration changes and for users who prefer a straightforward terminal editing interface without a steep learning curve.

## Detailed Explanation

### What is Nano?
Originally created as a free replacement for the non-free **Pico** editor (part of the Pine email suite), Nano has become a staple in most Linux distributions. Its primary appeal is its accessibility; most operations are triggered by holding the `Ctrl` key (represented as `^` in the UI) followed by a letter.

### Basic Usage

#### Opening and Creating Files
To open an existing file or create a new one, use:
```bash
nano filename.txt
```
To open a file at a specific line number (useful for debugging):
```bash
nano +line_number filename.txt
```

#### Essential Shortcuts
Nano uses the `Ctrl` (indicated by `^`) and `Alt` (indicated by `M-` for "Meta") keys.

| Shortcut | Action | Description |
| :--- | :--- | :--- |
| `Ctrl + G` | **Get Help** | Displays the internal help manual. |
| `Ctrl + O` | **Write Out** | Saves the current file (asks for filename). |
| `Ctrl + X` | **Exit** | Closes the editor (prompts to save if changed). |
| `Ctrl + K` | **Cut Text** | Deletes the current line and stores it in the buffer. |
| `Ctrl + U` | **Uncut Text** | Pastes the contents of the cut buffer. |
| `Ctrl + W` | **Where Is** | Searches for a string or regex. |
| `Ctrl + \` | **Replace** | Replaces a string or regex. |
| `Ctrl + J` | **Justify** | Justifies the current paragraph. |
| `Ctrl + C` | **Cur Pos** | Shows the current cursor position (line/column). |
| `Alt + U` | **Undo** | Reverts the last action. |
| `Alt + E` | **Redo** | Reapplies a reverted action. |

### Navigation
While arrow keys work as expected, professional use often involves:
- `Ctrl + A`: Jump to the beginning of the line.
- `Ctrl + E`: Jump to the end of the line.
- `Ctrl + Y`: Page up.
- `Ctrl + V`: Page down.
- `Alt + \`: Jump to the top of the file.
- `Alt + /`: Jump to the bottom of the file.

### Editing in Go Context
While Go developers often use IDEs (VS Code, GoLand) or Vim, Nano is excellent for quick fixes on remote servers.

#### Syntax Highlighting
Nano supports syntax highlighting. For Go, ensure you have a `go.nanorc` file included in your `~/.nanorc`. Most modern distributions include this by default in `/usr/share/nano/`.

```bash
# Example: Enabling Go syntax highlighting in ~/.nanorc
include "/usr/share/nano/go.nanorc"
```

#### Quick Go Fixes
If you need to change a constant or a simple logic branch in a Go file directly on a production or staging server:
```bash
nano main.go
# Search for the line with Ctrl+W
# Edit the text
# Save with Ctrl+O, then Exit with Ctrl+X
```

### Advanced Features
- **Multiple Buffers**: Open multiple files with `nano -F file1 file2` and switch between them using `Alt + <` and `Alt + >`.
- **Soft Wrap**: Use `nano -S` to enable smooth scrolling instead of jumping to the next line.
- **Auto-indent**: Use `nano -i` to automatically indent new lines based on the previous line's indentation (critical for Go code).

## Interview Questions

**Q: How do you save changes and exit Nano?**
**A:** Press `Ctrl + O` (Write Out) to save the file, then `Ctrl + X` (Exit) to close the editor. If you just press `Ctrl + X` with unsaved changes, Nano will prompt you whether to save before exiting.

**Q: What is the difference between `Ctrl + K` and a standard "Delete"?**
**A:** `Ctrl + K` (Cut) deletes the entire current line and stores it in a temporary "cut buffer." You can then move the cursor and use `Ctrl + U` (Uncut) to paste that line elsewhere, making it a quick way to move lines of code.

**Q: How can you search for a specific word in a large file using Nano?**
**A:** Use the "Where Is" command by pressing `Ctrl + W`. Type the search term and press Enter. To find the next occurrence of the same term, press `Alt + W`.

**Q: Does Nano support undo/redo functionality?**
**A:** Yes. `Alt + U` is used for Undo and `Alt + E` is used for Redo. These were added in more recent versions of GNU Nano and are essential for correcting mistakes without retyping.

**Q: How would you enable line numbers in Nano?**
**A:** You can start Nano with the `-l` flag (`nano -l file.txt`) or toggle them while inside the editor using `Alt + N`.
