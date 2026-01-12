---
tags: ['linux', 'roadmap']
---

# Vim Editor (Modes, Basic Navigation, Editing, Saving)

## Summary
Vim (Vi IMproved) is a highly efficient, modal text editor based on the classic `vi` editor. It is designed for use both from a command-line interface and as a standalone application in a graphical user interface. Vim is known for its steep learning curve but offers immense productivity through its command-based navigation and editing system, allowing users to perform complex text manipulations without leaving the keyboard.

## Detailed Explanation

### **Vim Modes**
The most critical concept in Vim is its **modal** nature. Unlike traditional editors where every keypress inserts a character, Vim interprets keys differently depending on the active mode.

1.  **Normal Mode (Default)**:
    *   This is the mode Vim starts in.
    *   Used for navigation and executing commands.
    *   Press `Esc` to return to Normal Mode from any other mode.
2.  **Insert Mode**:
    *   Used for typing text into the file.
    *   Entered from Normal Mode by pressing `i` (insert before cursor), `a` (append after cursor), or `o` (open new line below).
3.  **Visual Mode**:
    *   Used for selecting blocks of text.
    *   Entered from Normal Mode by pressing `v` (character-wise), `V` (line-wise), or `Ctrl+v` (block-wise).
4.  **Command-line Mode**:
    *   Used for entering complex commands like saving, quitting, or searching.
    *   Entered from Normal Mode by typing `:` (command), `/` (search forward), or `?` (search backward).

### **Basic Navigation (Normal Mode)**
Vim uses the home row for navigation to keep hands in a neutral position:
*   `h`: Move left
*   `j`: Move down
*   `k`: Move up
*   `l`: Move right

**Advanced Movement**:
*   `w`: Jump forward to the start of the next word.
*   `b`: Jump backward to the start of the previous word.
*   `0`: Jump to the beginning of the line.
*   `$`: Jump to the end of the line.
*   `gg`: Go to the first line of the document.
*   `G`: Go to the last line of the document.
*   `{line_number}G`: Go to a specific line number.

### **Editing Commands**
*   `x`: Delete the character under the cursor.
*   `dd`: Delete (cut) the current line.
*   `yy`: Yank (copy) the current line.
*   `p`: Paste the yanked/deleted text after the cursor.
*   `u`: Undo the last action.
*   `Ctrl+r`: Redo the last undone action.
*   `r`: Replace a single character.

### **Saving and Exiting (Command Mode)**
All these commands start with a `:` from Normal Mode:
*   `:w`: Write (save) the file.
*   `:q`: Quit (fails if there are unsaved changes).
*   `:wq` or `:x`: Save and quit.
*   `:q!`: Quit without saving changes.

## Interview Questions

1.  **Q: What is the primary difference between Vim and a standard text editor like Notepad?**
    **A:** Vim is a modal editor, meaning keys have different functions depending on the mode (Normal, Insert, Visual, etc.), whereas standard editors are typically modeless (typing always inserts text).

2.  **Q: How do you enter "Insert Mode" to begin typing text, and how do you return to "Normal Mode"?**
    **A:** You enter Insert Mode by pressing `i`, `a`, or `o`. You return to Normal Mode by pressing the `Esc` key.

3.  **Q: What command would you use to delete 5 lines of text starting from the cursor?**
    **A:** You would type `5dd` in Normal Mode.

4.  **Q: How do you search for a specific word in a file?**
    **A:** In Normal Mode, type `/` followed by the word you want to find and press `Enter`. Use `n` to jump to the next occurrence and `N` for the previous.

5.  **Q: If you have made changes to a file but want to discard them and exit, what command do you use?**
    **A:** Use the command `:q!` in Command-line Mode.
