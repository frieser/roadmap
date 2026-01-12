# Linux
---
tags: ['linux', 'roadmap']
---

# Filesystems (ext4, xfs, btrfs, journaling)

## Summary
Linux filesystems are the methods and data structures that an operating system uses to keep track of files on a disk or partition. This note covers the most common journaling filesystems used in modern Linux distributions: **ext4**, **XFS**, and **Btrfs**. It explores their architecture, the role of journaling in data integrity, and practical management via CLI tools.

## Detailed Explanation

### 1. The Role of Journaling
A **journaling filesystem** maintains a "journal" (a dedicated area on the disk) where it records changes before they are actually committed to the main filesystem.
- **Data Integrity**: Prevents filesystem corruption in case of a power failure or system crash.
- **Fast Recovery**: Upon reboot, the system replays the journal to complete or roll back pending transactions, significantly speeding up the consistency check compared to a full `fsck` scan.

### 2. Common Linux Filesystems

#### ext4 (Fourth Extended Filesystem)
The most common default for many distributions (like Ubuntu and Debian).
- **Type**: Block-based journaling filesystem.
- **Key Features**: Extents (reducing fragmentation), delayed allocation, and backward compatibility with ext2/ext3.
- **Limits**: Supports volumes up to 1 EB and files up to 16 TB.

#### XFS
The default filesystem for RHEL, CentOS, and Fedora.
- **Type**: 64-bit high-performance journaling filesystem.
- **Key Features**: Optimized for parallel I/O and large files. It uses B+ trees for space management.
- **Constraints**: XFS partitions can be grown online but cannot be shrunk.

#### Btrfs (B-tree FS)
A modern "Copy-on-Write" (CoW) filesystem focused on fault tolerance and advanced features.
- **Key Features**: Built-in RAID support, writable snapshots, data scrubbing, and checksums for data and metadata.
- **CoW Model**: Instead of overwriting data, it writes new data to a new block and updates the pointers, ensuring high reliability.

### 3. Comparison Table

| Feature | ext4 | XFS | Btrfs |
| :--- | :--- | :--- | :--- |
| **Data Model** | Block-based | B+ tree | Copy-on-Write (CoW) |
| **Snapshots** | No (native) | No (native) | Yes |
| **Shrinkable** | Yes | No | Yes |
| **Deduplication** | No | No | Yes |
| **Built-in RAID** | No | No | Yes |

### 4. Bash Examples

**Creating Filesystems (mkfs)**
```bash
# Create an ext4 filesystem on partition /dev/sdb1
sudo mkfs.ext4 /dev/sdb1

# Create an XFS filesystem
sudo mkfs.xfs /dev/sdb2

# Create a Btrfs filesystem
sudo mkfs.btrfs /dev/sdb3
```

**Checking and Repairing**
```bash
# Check an ext4 filesystem (partition must be unmounted)
sudo fsck.ext4 -f /dev/sdb1

# Repair an XFS filesystem (offline)
sudo xfs_repair /dev/sdb2

# Verify a Btrfs filesystem integrity
sudo btrfs check /dev/sdb3
```

**Management and Tuning**
```bash
# Mount a filesystem with specific options
sudo mount -t xfs -o noatime /dev/sdb2 /mnt/data

# Change the label of an ext4 partition
sudo tune2fs -L "DATA_DRIVE" /dev/sdb1

# Get detailed info about an XFS partition
xfs_info /mnt/data
```

## Interview Questions

**Q: What is the main advantage of a journaling filesystem?**
**A:** It ensures filesystem consistency and allows for rapid recovery after an improper shutdown. By recording changes in a journal before applying them, the system avoids the need for a full disk scan (`fsck`), which can take hours on large drives.

**Q: How does Copy-on-Write (CoW) in Btrfs improve data safety?**
**A:** In a CoW system, data is never overwritten in place. When a file is modified, the new data is written to a new block. If the system crashes during the write, the old data remains intact because the pointers haven't been updated yet, preventing "split-write" corruption.

**Q: Why would you choose XFS over ext4 for a database server?**
**A:** XFS is designed for high-concurrency environments and handles large files and parallel I/O much more efficiently than ext4. It excels in performance when multiple processes are writing to the same filesystem simultaneously.

**Q: Can you shrink an XFS partition?**
**A:** No. While XFS partitions can be expanded online, the current implementation does not support shrinking. If you need a smaller partition, you must back up the data, recreate the filesystem with a smaller size, and restore the data.
