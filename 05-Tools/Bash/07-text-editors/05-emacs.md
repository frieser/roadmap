---
tags: ['emacs', 'editor', 'lisp', 'linux', 'tools', 'roadmap']
---

# Emacs

## Summary

**Emacs** is a highly extensible, customizable text editor created by Richard Stallman in 1976. Unlike modal editors like vi/vim, Emacs uses modifier keys (Ctrl, Alt/Meta) for commands. Often described as "an operating system disguised as an editor," Emacs offers email, file management, shell integration, and more through its Lisp-based extension system.

## Detailed Explanation

### Emacs Philosophy

| Concept | Description |
|----------|-------------|
| **Extensible** | Everything is Lisp code, fully customizable |
| **Self-Documenting** | Built-in help for every command |
| **Platform** | Complete development environment |
| **Non-Modal** | No modes (unlike vi/vim) |
| **Interactive** | Commands often ask for input |

### Emacs vs Other Editors

| Feature | Emacs | Vim | Nano |
|---------|--------|------|-------|
| **Extension Language** | Emacs Lisp | VimL | None |
| **Learning Curve** | Very Steep | Steep | Easy |
| **Customization** | Unlimited | High | Limited |
| **Built-in Features** | Email, IRC, PDF, games | Editing only | Editing only |
| **Modes** | Yes (major modes) | Yes (editing modes) | No |
| **Startup Time** | Slow | Fast | Very Fast |
| **Memory Usage** | High | Medium | Low |

### Basic Emacs Commands

#### Starting Emacs

```bash
emacs                    # Start GUI version
emacs -nw               # Start terminal version
emacs filename.txt        # Open specific file
emacs +10 filename.txt   # Open at line 10
emacs file1 file2        # Open multiple files
```

#### File Operations

| Shortcut | Action |
|-----------|--------|
| `C-x C-f` | Find file (open) |
| `C-x C-s` | Save file |
| `C-x C-w` | Write file (save as) |
| `C-x C-c` | Quit Emacs |
| `C-x C-v` | Visit another file |
| `C-x b` | Switch buffer |

**Note:** `C-` = Ctrl, `M-` = Alt/Meta

#### Navigation

| Shortcut | Action |
|-----------|--------|
| `C-f` | Forward one character |
| `C-b` | Backward one character |
| `C-n` | Next line |
| `C-p` | Previous line |
| `C-a` | Beginning of line |
| `C-e` | End of line |
| `M-f` | Forward one word |
| `M-b` | Backward one word |
| `C-v` | Next page |
| `M-v` | Previous page |
| `M-<` | Beginning of buffer |
| `M->` | End of buffer |

#### Editing

| Shortcut | Action |
|-----------|--------|
| `C-d` | Delete character |
| `M-d` | Delete word |
| `C-k` | Kill (cut) to end of line |
| `C-w` | Kill region (cut selection) |
| `M-w` | Copy region |
| `C-y` | Yank (paste) |
| `C-@` | Set mark |
| `C-x C-x` | Exchange point and mark |
| `C-x h` | Mark entire buffer |
| `C-w` | Kill region between mark and point |

#### Search and Replace

| Shortcut | Action |
|-----------|--------|
| `C-s` | Incremental search forward |
| `C-r` | Incremental search backward |
| `M-%` | Query-replace (interactive) |
| `M-x replace-string` | Replace all |
| `M-x replace-regexp` | Regex replace |

#### Undo and Redo

| Shortcut | Action |
|-----------|--------|
| `C-/` | Undo |
| `C-x u` | Undo (alternative) |
| `C-g C-/` | Redo |

#### Buffers and Windows

| Shortcut | Action |
|-----------|--------|
| `C-x b` | Switch buffer |
| `C-x k` | Kill buffer |
| `C-x C-b` | List buffers |
| `C-x 2` | Split window horizontally |
| `C-x 3` | Split window vertically |
| `C-x o` | Other window (switch) |
| `C-x 1` | Delete other windows |
| `C-x 0` | Delete current window |
| `C-x ^` | Other frame |

### Emacs Modes

```lisp
;; Emacs has major and minor modes
;; Major mode: Specific to file type
;; Minor mode: Additional features

;; Common major modes:
text-mode             ;; Plain text
fundamental-mode       ;; Default mode
python-mode           ;; Python files
js-mode              ;; JavaScript
markdown-mode         ;; Markdown
org-mode             ;; Org-mode (notes, outlines)

;; Common minor modes:
auto-fill-mode        ;; Word wrap
flyspell-mode         ;; Spell checking
linum-mode           ;; Line numbers
company-mode         ;; Autocomplete
```

### Emacs Configuration (.emacs)

```lisp
;; ~/.emacs or ~/.emacs.d/init.el

;; Disable startup message
(setq inhibit-startup-screen t)

;; Enable line numbers
(global-linum-mode t)

;; Show column number
(column-number-mode t)

;; Show matching parentheses
(show-paren-mode t)

;; Set tab width
(setq-default indent-tabs-mode nil)
(setq tab-width 4)

;; Highlight current line
(global-hl-line-mode t)

;; Remove toolbar
(tool-bar-mode -1)

;; Remove menu bar (optional)
(menu-bar-mode -1)

;; Disable scroll bar
(scroll-bar-mode -1)

;; Set theme
(load-theme 'tango)

;; Package management (Emacs 24.4+)
(require 'package)
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/"))
(package-initialize)

;; Install packages automatically
(package-refresh-contents)
(package-install-selected-packages)

;; Example: Install useful packages
(unless (package-installed-p 'magit)
  (package-install 'magit))

(unless (package-installed-p 'helm)
  (package-install 'helm))

;; Keybindings
(global-set-key (kbd "C-c C-c") 'compile)
(global-set-key (kbd "C-c C-g") 'magit-status)
```

### Essential Emacs Packages

```lisp
;; Popular packages (use M-x package-install <package>)

;; Version control
magit              ;; Git interface (M-x magit-status)

;; Auto-completion
company            ;; Modern completion
auto-complete       ;; Alternative completion

;; Project management
projectile         ;; Project navigation
helm               ;; Fuzzy finder (alternative to anything)
ido-mode           ;; Built-in fuzzy completion

;; Development
flycheck           ;; Syntax checking
yasnippet          ;; Code snippets
emacs-lsp-mode     ;; Language Server Protocol
docker              ;; Docker integration

;; Org-mode
org-mode           ;; Notes, planning, documents (built-in)
```

### Org Mode

```org
;; Org mode is built-in and powerful

;; Create org file: C-x C-f notes.org

;; Basic outline:
* Top level heading
** Sub heading
*** Sub-sub heading

;; Todo items
* TODO [ ] Task to do
* TODO [X] Completed task

;; Tags and priorities
* IMPORTANT [#A] Task with priority
* Task :work:urgent:

;; Code blocks
#+BEGIN_SRC python
def hello():
    print("Hello, World!")
#+END_SRC

;; Execute code: C-c C-c (in src block)

;; Export to other formats
;; C-c C-e h: Export to HTML
;; C-c C-e o: Export to ODT
;; C-c C-e p: Export to PDF
```

### Emacs Lisp Programming

```lisp
;; Define a function
(defun my-insert-date ()
  "Insert current date at point."
  (interactive)
  (insert (format-time-string "%Y-%m-%d")))

;; Bind to key
(global-set-key (kbd "C-c d") 'my-insert-date)

;; Use: Press C-c d to insert date

;; Define a command
(defun my-clean-buffer ()
  "Remove trailing whitespace and extra blank lines."
  (interactive)
  (delete-trailing-whitespace)
  (save-buffer))

;; Advanced example: Sort lines
(defun my-sort-lines ()
  "Sort lines in region."
  (interactive)
  (sort-lines nil (region-beginning) (region-end)))
```

### Dired Mode (File Manager)

```emacs
;; Open directory listing: C-x d

;; Navigation:
n       ;; Next file
p       ;; Previous file
RET     ;; Open file/directory
d       ;; Mark for deletion
x       ;; Execute deletions
m       ;; Mark file
u       ;; Unmark
g       ;; Refresh directory
+       ;; Create directory

;; Operations:
C       ;; Copy
R       ;; Rename
Z       ;; Compress

;; Dired features:
;; - Powerful file operations
;; - Recursive operations
;; - Integration with shell commands
;; - Mark files by pattern
```

### Magit (Git Interface)

```emacs
;; Install: M-x package-install magit

;; Open git status: M-x magit-status

;; Magit sections:
;; Unstaged changes
;; Staged changes
;; Stashes
;; Unpulled commits
;; Unpushed commits

;; Keybindings:
s       ;; Stage
u       ;; Unstage
k       ;; Discard
c       ;; Commit
P       ;; Push
F       ;; Pull
b       ;; Branch
l       ;; Log

;; Magit workflow is menu-driven
;; Press ? at any point for help
```

### Emacs Built-in Features

```emacs
;; Calculator
M-x calc

;; Calendar
M-x calendar

;; Games
M-x tetris
M-x snake
M-x gomoku

;; Shell
M-x shell            ;; Integrated shell
M-x eshell           ;; Emacs Lisp shell

;; Email
M-x rmail            ;; Read mail
M-x sendmail         ;; Compose mail

;; IRC
M-x erc              ;; Emacs IRC client

;; Directory traversal
C-x d                ;; Dired (file browser)

;; Spell checking
M-x ispell
M-x flyspell-mode

;; Comparison
M-x ediff             ;; File diff
```

### Emacs vs Spacemacs vs Doom Emacs

| Variant | Description |
|---------|-------------|
| **Stock Emacs** | Pure Emacs, configure manually |
| **Spacemacs** | Pre-configured, Vim-like keybindings (spacebar leader) |
| **Doom Emacs** | Fast, pre-configured, modular, uses Evil (Vim emulation) |

```emacs
;; Doom Emacs installation
git clone --depth 1 https://github.com/hlissner/doom-emacs ~/.config/emacs
~/.config/emacs/bin/doom install
```

### Practical Workflows

#### Writing Code

```emacs
;; 1. Open file
C-x C-f app.py

;; 2. Enable programming features
M-x flycheck-mode       ;; Syntax checking
M-x company-mode        ;; Autocomplete

;; 3. Write code

;; 4. Check errors (flycheck shows them)

;; 5. Execute code in shell
M-x shell
python app.py
```

#### Taking Notes (Org Mode)

```emacs
;; 1. Create org file
C-x C-f notes.org

;; 2. Create structure
* Project Meeting [2024-01-15]
** Attendees
- John
- Jane
** Action Items
*** TODO Write documentation
*** TODO Deploy to staging
*** DONE Set up CI/CD

;; 3. Add TODOs with C-c C-t

;; 4. Track progress
;; C-c C-t on TODO to cycle states:
;; TODO -> DONE

;; 5. Export
;; C-c C-e o (to ODT) or h (to HTML)
```

#### Git Workflow with Magit

```emacs
;; 1. Open magit status
M-x magit-status

;; 2. Stage changes
s (on file or section)

;; 3. Commit
c - c (type message)
RET (commit)

;; 4. Push
P - u (to upstream)

;; Magit handles git operations through menus
;; Very intuitive once learned
```

### Emacs Resources

```emacs
;; Built-in help
C-h t      ;; Tutorial (essential for beginners)
C-h r      ;; Emacs manual
C-h f      ;; Describe function
C-h k      ;; Describe key
C-h v      ;; Describe variable

;; Online resources
emacs.org              ;; Official site
github.com/emacs-mirror/emacs
emacswiki.org          ;; Community wiki
github.com/sachac/.emacs.d  ;; Sample configs
```

### Emacs for Developers

```emacs
;; Why developers love Emacs:
;; - LSP integration (language-server-mode)
;; - Powerful refactoring tools
;; - Integrated debugging (gdb-mode)
;; - Project management (projectile)
;; - Live code execution
;; - REPL integration
;; - Magit for Git
;; - Org-mode for documentation
;; - Completely customizable

;; Common dev packages:
;; - irony: C/C++ completion
;; - jedi: Python completion
;; - tide: TypeScript/JavaScript completion
;; - rustic: Rust development
;; - go-mode: Go development
```

## Interview Questions

**Q: What makes Emacs different from vi/vim?**
**A:** Emacs uses modifier keys (Ctrl, Alt) instead of modes, has no modal editing, and is written in Emacs Lisp making it infinitely extensible. Vi/vim is modal and uses separate modes for navigation vs. typing. Emacs is more like an entire operating system with built-in email, Git, file manager, etc.

**Q: What is Org Mode in Emacs?**
**A:** Org mode is a powerful built-in major mode for note-taking, project planning, document authoring, and literate programming. It uses plain text with lightweight markup, supports outlines, TODO lists, tables, code blocks, and can export to multiple formats (HTML, PDF, ODT).

**Q: How do you save and exit in Emacs?**
**A:** Save: `Ctrl+X`, `Ctrl+S` (write file). Exit: `Ctrl+X`, `Ctrl+C` (quit). Emacs prompts to save if file modified. Save and exit in sequence: `Ctrl+X, Ctrl+S, Ctrl+X, Ctrl+C`.

**Q: What is the Emacs Lisp (Elisp) and why is it important?**
**A:** Emacs Lisp is the extension language of Emacs. Nearly all Emacs functionality is written in Elisp, and you can extend Emacs by writing your own Elisp functions. This makes Emacs the most customizable editor available.

**Q: What are Magit and why is it popular?**
**A:** Magit is a Git interface for Emacs that many users prefer over command-line git. It presents Git operations in a menu-driven, visual format making staging, committing, branching, and pushing more intuitive than git commands.

**Q: How does Emacs handle multiple files?**
**A:** Emacs uses **buffers** (in-memory file representations) and **windows** (screen splits). Open files create buffers (`C-x C-f`), switch between them (`C-x b`), split windows (`C-x 2` for horizontal, `C-x 3` for vertical), and switch windows (`C-x o`).
