# Linux
---
tags: ['linux', 'roadmap']
---

## Summary
In Linux, **links** are pointers to files or directories. There are two types: **Hard Links**, which are additional names for the same inode (pointing directly to the data), and **Soft Links** (Symbolic Links), which are shortcuts pointing to a file path. Understanding the relationship between filenames, inodes, and data blocks is key to mastering file management.

## Detailed Explanation

### The Inode Concept
An **inode** (index node) is a data structure on a filesystem that stores metadata about a file (size, owner, permissions, timestamps) and pointers to the actual data blocks on the disk. Crucially, the inode does **not** store the filename. The filename is just an entry in a directory that maps a string to an inode number.

### Hard Links
A hard link is essentially a second (or third) name for an existing inode.
- **Shared Inode**: Both the original filename and the hard link point to the exact same inode number.
- **Data Persistence**: If you delete the original filename, the data remains accessible through the hard link. The data is only deleted when the "link count" on the inode reaches zero.
- **Limitations**:
    - Cannot link directories (to prevent filesystem loops).
    - Cannot span across different filesystems (since inode numbers are unique only within a specific filesystem).

### Soft Links (Symbolic Links)
A soft link is a special file that contains the path to another file or directory.
- **Unique Inode**: A soft link has its own unique inode number and its own data blocks (storing the path string).
- **Dangling Links**: If the source file is deleted or moved, the soft link remains but points to a non-existent path, becoming "broken".
- **Flexibility**:
    - Can link to directories.
    - Can cross filesystem boundaries.

### Bash Examples

#### 1. Creating and Inspecting Links
```bash
# Create a sample file
echo "Hello World" > original.txt

# Create a Hard Link
ln original.txt hard_link.txt

# Create a Soft Link
ln -s original.txt soft_link.txt

# Inspect Inodes (the first column is the inode number)
ls -li
# Output:
# 123456 -rw-r--r-- 2 user user 12 Jan 10 12:00 hard_link.txt
# 123456 -rw-r--r-- 2 user user 12 Jan 10 12:00 original.txt
# 789012 lrwxrwxrwx 1 user user 12 Jan 10 12:00 soft_link.txt -> original.txt
```

#### 2. Deleting the Source
```bash
# Delete the original
rm original.txt

# Hard link still works!
cat hard_link.txt  # Output: Hello World

# Soft link is broken!
cat soft_link.txt  # Output: cat: soft_link.txt: No such file or directory
```

## Interview Questions

### 1. What happens to a hard link if the original file is moved to a different directory on the same filesystem?
The hard link continues to work perfectly. Since a hard link points to the inode number and not the path, moving the original file (which just updates its directory entry) doesn't affect the link between the hard link's name and the inode.

### 2. Can you create a hard link to a directory? Why or why not?
Generally, no. Modern Linux systems restrict hard linking directories to prevent infinite loops in the filesystem structure (e.g., a directory hard-linked to its parent), which would break tools like `find` or `du`. Soft links are used for directories instead.

### 3. How can you identify if two filenames are hard links to the same file?
By checking their inode numbers using `ls -i`. If the inode numbers are identical and they reside on the same filesystem, they are hard links to the same data. You can also see the link count in `ls -l` (the number after the permissions).

### 4. What is a "dangling" or "broken" symlink?
A dangling symlink occurs when the target file or directory that the symlink points to is deleted or moved. The symlink file itself still exists, but the path it contains no longer leads to a valid file.

### 5. Why can't hard links cross filesystem boundaries?
Inode numbers are local to each filesystem (partition). Inode 100 on `/dev/sda1` is completely different from Inode 100 on `/dev/sdb1`. Since a hard link points directly to an inode number, it cannot point to an inode on a different disk/partition. Soft links solve this by pointing to a path string instead.
