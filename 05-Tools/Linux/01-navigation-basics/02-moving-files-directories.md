#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
The `mv` (move) command in Linux is used to move files and directories from one location to another or to rename them. Unlike `cp`, which creates a duplicate, `mv` effectively "cuts and pastes" the source to the destination. It is a fundamental tool for organizing filesystems and managing file names.

## Detailed Explanation

The `mv` command is part of the GNU Coreutils and is one of the most frequently used commands in the Linux terminal. Its behavior depends on whether the destination is an existing directory or a new filename.

### Syntax
```bash
mv [OPTIONS] SOURCE DEST
mv [OPTIONS] SOURCE... DIRECTORY
```

### 1. Renaming Files and Directories
If you provide two filenames, and the second one does not exist as a directory, `mv` renames the source to the destination.
```bash
# Rename file.txt to new_name.txt
mv file.txt new_name.txt

# Rename a directory
mv old_dir/ new_dir/
```

### 2. Moving Files to a Directory
If the destination is an existing directory, the source files are moved into that directory while keeping their original names.
```bash
# Move a file into a folder
mv report.pdf documents/

# Move multiple files at once
mv file1.txt file2.txt image.png backup_folder/
```

### 3. Common Options
- `-i` (**Interactive**): Prompts for confirmation before overwriting an existing file.
  ```bash
  mv -i source.txt existing_dest.txt
  ```
- `-f` (**Force**): Overwrites existing files without prompting, even if they are read-only.
- `-n` (**No-clobber**): Prevents overwriting any existing file at the destination.
- `-u` (**Update**): Moves the file only if the source is newer than the destination or if the destination doesn't exist.
- `-v` (**Verbose**): Displays the name of each file as it is moved.
  ```bash
  mv -v *.jpg images/
  # Output: 'photo1.jpg' -> 'images/photo1.jpg'
  ```
- `-b` (**Backup**): Creates a backup of existing destination files before overwriting them (usually appends a `~`).

### 4. Technical Nuance: Inodes and Filesystems
- **Same Filesystem**: When moving a file within the same partition, `mv` simply updates the directory entry to point to the existing **inode**. The data itself isn't moved, making the operation instantaneous regardless of file size.
- **Across Filesystems**: When moving between different partitions or disks, `mv` must physically copy the data to the new location and then delete the original source.

## Interview Questions

### 1. How does the `mv` command differ when moving a file within the same filesystem versus across different filesystems?
On the same filesystem, `mv` only renames the file's entry in the directory structure, keeping the same inode and not moving actual data. Across filesystems, it performs a copy followed by a delete, which takes time proportional to the file size.

### 2. Which option would you use to prevent `mv` from overwriting an existing file at the destination?
You can use the `-n` (no-clobber) option to skip moving if the destination already exists, or the `-i` (interactive) option to be prompted before overwriting.

### 3. How can you rename a directory in Linux?
You use the `mv` command: `mv old_directory_name new_directory_name`. If `new_directory_name` does not exist, the directory is renamed.

### 4. What happens if you run `mv file1.txt file2.txt dir1/` if `dir1` does not exist?
The command will fail with an error because when moving multiple sources, the final argument MUST be an existing directory.

### 5. How do you move only the files that are newer than their counterparts in the destination directory?
Use the `-u` (update) flag: `mv -u source_dir/* dest_dir/`.
