---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Repeat Commands (History)

## Summary

Bash maintains a **history** of executed commands that can be searched, recalled, and modified. The `history` builtin and various keyboard shortcuts enable quickly repeating or modifying previous commands. Mastering history features dramatically improves productivity by eliminating redundant typing.

## Detailed Explanation

### Basic History Commands

```bash
# Show command history
history

# Show last N commands
history 20

# Execute command by number
!42              # Run command #42

# Execute last command
!!               # Run previous command

# Execute last command starting with string
!ssh             # Run last command starting with "ssh"

# Execute last command containing string
!?pattern?       # Run last command containing "pattern"
```

### History Expansion

```bash
# Last command's arguments
echo hello world
echo !!          # → echo echo hello world

# Last argument of previous command
cd /var/log/nginx
cat !$           # → cat /var/log/nginx (same as !:$)

# First argument
ls -la /home
cd !^            # → cd -la (first arg, probably not wanted)

# All arguments
ls -la /home
cp !* /backup    # → cp -la /home /backup

# Specific argument
!:2              # Second argument
!:2-4            # Arguments 2 through 4
```

### Interactive History Search

```bash
# Reverse search (most useful!)
Ctrl+R           # Start typing to search
                 # Press Ctrl+R again for older matches
                 # Enter to execute, Ctrl+G to cancel

# Forward search (if enabled)
Ctrl+S           # Search forward (may need stty -ixon)

# Navigate history
↑ / Ctrl+P      # Previous command
↓ / Ctrl+N      # Next command
```

### History Modifiers

```bash
# Quick substitution
echo hello world
^hello^goodbye   # → echo goodbye world

# More general substitution
!:s/old/new      # Substitute once
!:gs/old/new     # Substitute globally

# Remove extension
echo /path/to/file.txt
echo !$:r        # → echo /path/to/file

# Get extension
echo !$:e        # → echo .txt

# Get head (directory)
echo !$:h        # → echo /path/to

# Get tail (filename)
echo !$:t        # → echo file.txt
```

### History Configuration

```bash
# Add to ~/.bashrc

# History size
HISTSIZE=10000         # In-memory history
HISTFILESIZE=20000     # On-disk history

# Ignore duplicates and whitespace
HISTCONTROL=ignoreboth  # Same as ignoredups:ignorespace

# Ignore specific commands
HISTIGNORE="ls:cd:pwd:exit:clear"

# Add timestamps
HISTTIMEFORMAT="%F %T "

# Append to history (don't overwrite)
shopt -s histappend

# Save history immediately (not just on exit)
PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}history -a"
```

### Useful History Tricks

```bash
# Run command without saving to history
 ls -la          # Note leading space (needs HISTCONTROL=ignorespace)

# Delete specific entry
history -d 42

# Clear all history
history -c

# View without executing (add :p)
!ssh:p           # Print, don't execute

# Edit before executing
fc               # Opens last command in $EDITOR
fc -l            # List recent commands
```

### Multi-Session History

```bash
# Share history between terminal sessions
# Add to ~/.bashrc

# Append to file immediately
shopt -s histappend
PROMPT_COMMAND="history -a; history -n"

# Or for immediate sharing:
PROMPT_COMMAND="history -a; history -c; history -r"
```

## Interview Questions

**Q: How do you repeat the last command in Bash?**
**A:** Use `!!`. You can also use the up arrow to recall it and press Enter. `sudo !!` is a common pattern to rerun the last command with sudo.

**Q: What is Ctrl+R used for?**
**A:** Reverse incremental search through command history. Type a pattern and Bash shows the most recent matching command. Press Ctrl+R again for older matches, Enter to execute, or Ctrl+G to cancel.

**Q: How do you prevent a command from being saved to history?**
**A:** Start the command with a space (requires `HISTCONTROL=ignorespace` or `ignoreboth`). You can also delete it after with `history -d <number>`.

**Q: What does `!$` represent?**
**A:** The last argument of the previous command. Very useful for operating on the same file: `cat /etc/config.yaml` then `vim !$` to edit the same file.
