#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
An **inode** (Index Node) is a fundamental data structure in Unix-style filesystems that stores metadata about a filesystem object (file, directory, or symbolic link). Every object is assigned an inode number, which serves as a unique identifier within the filesystem. Critically, the inode contains almost everything about a file except for its name and the actual data content.

## Detailed Explanation

### What is an Inode?
In Linux, files are managed via inodes. When you access a file, the system looks up its inode to determine its location on the disk and verify access permissions.

#### Metadata Stored in an Inode
The inode contains the following attributes:
- **File Type**: Regular file, directory, socket, pipe, block device, character device, or symbolic link.
- **Permissions**: Read, write, and execute bits for Owner, Group, and Others.
- **Owner Information**: User ID (UID) and Group ID (GID).
- **File Size**: Size in bytes.
- **Timestamps**:
  - `atime`: Last access time.
  - `mtime`: Last modification time (content changed).
  - `ctime`: Last status change time (metadata or inode changed).
- **Link Count**: The number of hard links pointing to this inode.
- **Data Pointers**: Pointers to the disk blocks or extents where the actual content is stored.

#### What is NOT in an Inode?
- **Filename**: Filenames are stored in the data blocks of the **parent directory**, which maps filenames to inode numbers.
- **File Data**: The actual content is stored in separate data blocks.

### Working with Inodes (Bash Examples)

#### 1. Checking Inode Numbers
Use `ls -i` to see the inode number associated with a file.
```bash
# Create a file and check its inode
touch test_file.txt
ls -i test_file.txt
# Output example: 1234567 test_file.txt
```

#### 2. Detailed Metadata
The `stat` command provides a human-readable summary of the inode information.
```bash
stat test_file.txt
```

#### 3. Monitoring Inode Usage
Filesystems have a fixed number of inodes allocated at creation (except for modern filesystems like XFS or Btrfs that allocate them dynamically). Use `df -i` to check usage.
```bash
df -i
# Output shows IUsed, IFree, and IUse%
```

### Running Out of Inodes
It is possible to "run out of disk space" even if there are gigabytes of free blocks available. This happens when the filesystem exhausts its supply of inodes, typically due to a massive number of very small files.

```bash
# Scenario: Creating many empty files (simulation)
for i in {1..100000}; do touch "file_$i"; done
# Eventually, touch will fail with "No space left on device"
# even if df (without -i) shows plenty of free blocks.
```

### Inodes and Hard Links
A hard link is simply an additional directory entry pointing to the same inode number.
```bash
# Create a hard link
ln test_file.txt hard_link.txt

# Both have the same inode number and the link count increases
ls -i test_file.txt hard_link.txt
stat test_file.txt | grep Links
```

## Interview Questions

**Q: What is the difference between an inode and a filename?**
**A:** An inode is the internal representation of a file's metadata and disk location, while a filename is just a human-readable alias stored in a directory entry. Multiple filenames (hard links) can point to the same inode.

**Q: Why would `df` show free space, but a "No space left on device" error occurs when creating a new file?**
**A:** This usually indicates that the filesystem has run out of available inodes. Every file requires an inode; if the inode table is full, no new files can be created regardless of available block space.

**Q: Does moving a file within the same filesystem change its inode number?**
**A:** No. Moving (renaming) a file within the same filesystem only updates the directory entry (mapping the name to the inode). The inode itself and its number remain the same.

**Q: What are the three main timestamps in an inode, and how do they differ?**
**A:** `atime` (Access time) is updated when the file is read. `mtime` (Modification time) is updated when the file content is changed. `ctime` (Change time) is updated when the inode metadata (like permissions or ownership) or content is changed.
