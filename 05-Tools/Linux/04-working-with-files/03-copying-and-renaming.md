#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
Copying, moving, and renaming are fundamental file operations in Linux. While `cp` and `mv` handle basic local operations, `rsync` provides advanced synchronization capabilities, efficiency through delta-transfers, and remote support, making it the preferred tool for backups and large data transfers.

## Detailed Explanation

### 1. Copying Files and Directories (`cp`)
The `cp` command creates a duplicate of a source file or directory.

*   **Basic Copy**: `cp source.txt destination.txt`
*   **Recursive Copy**: `cp -r source_dir/ destination_dir/` (Required for directories).
*   **Archive Mode**: `cp -a source/ destination/` (Preserves links, permissions, and timestamps; equivalent to `-dR --preserve=all`).
*   **Interactive Mode**: `cp -i source destination` (Prompts before overwriting).
*   **Update Only**: `cp -u source destination` (Only copies if source is newer or destination is missing).
*   **Verbose**: `cp -v source destination` (Shows progress).

### 2. Moving and Renaming (`mv`)
The `mv` command is used both to move files to different locations and to rename them. In Linux, renaming is essentially moving a file to the same location with a different name.

*   **Rename a File**: `mv old_name.txt new_name.txt`
*   **Move to Directory**: `mv file.txt /path/to/directory/`
*   **Interactive**: `mv -i source destination` (Warns before overwriting).
*   **No Clobber**: `mv -n source destination` (Does not overwrite existing files).
*   **Update**: `mv -u source destination` (Moves only if source is newer).

### 3. Synchronizing Files (`rsync`)
`rsync` (Remote Sync) is a powerful utility for local and remote file synchronization. It uses a delta-transfer algorithm to send only the differences between files.

*   **Standard Usage**: `rsync -avz source/ destination/`
    *   `-a` (Archive): Preserves permissions, ownerships, and symlinks.
    *   `-v` (Verbose): Provides detailed output.
    *   `-z` (Compress): Compresses data during transfer.
*   **Delete Extraneous Files**: `rsync -av --delete source/ destination/` (Makes destination an exact mirror).
*   **Show Progress**: `rsync -avP source destination` (Combination of `--partial` and `--progress`).
*   **Dry Run**: `rsync --dry-run -av source destination` (Tests the command without making changes).
*   **Remote Transfer**: `rsync -avz local_file user@remote_host:/path/to/destination`

> **Note on Trailing Slashes**: In `rsync`, `source/` (with slash) copies the *contents* of the directory, while `source` (without slash) copies the directory *itself*.

## Interview Questions

### 1. What is the difference between `cp -r` and `cp -a`?
`cp -r` copies directories recursively but does not necessarily preserve file attributes like timestamps, ownership, or symbolic links (it may resolve links). `cp -a` (archive) is more comprehensive; it implies `-r` and preserves almost all metadata and link structures, making it ideal for backups.

### 2. How do you rename all `.txt` files to `.bak` in a directory?
Using a basic `mv` inside a loop:
```bash
for f in *.txt; do mv "$f" "${f%.txt}.bak"; done
```
Alternatively, using the `rename` utility (if available):
`rename 's/\.txt$/\.bak/' *.txt`

### 3. Why is `rsync` preferred over `cp` for large backups?
`rsync` is idempotent and efficient. It uses a delta-transfer algorithm to copy only changed parts of files, supports compression, and can resume interrupted transfers using the `--partial` flag. It also supports remote transfers over SSH.

### 4. What happens if you run `mv file.txt dir/` where `dir/` is an existing directory?
The file `file.txt` is moved inside the directory `dir/`, resulting in `dir/file.txt`. If `dir` did not exist, `file.txt` would be renamed to a new file named `dir`.

### 5. How can you ensure you don't accidentally overwrite files when copying?
Use the `-i` (interactive) flag with `cp` or `mv`. This will prompt the user for confirmation before overwriting any existing file at the destination.
