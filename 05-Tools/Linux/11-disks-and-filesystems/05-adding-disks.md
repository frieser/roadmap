---
tags: ['linux', 'roadmap']
---

# Adding Disks

## Summary
Adding a new disk to a Linux system is a fundamental task that involves several distinct steps: **Identification**, **Partitioning**, **Formatting**, and **Mounting**. The process ensures the OS recognizes the raw hardware, divides it into logical sections, prepares it for data storage with a filesystem, and integrates it into the directory hierarchy for user access.

## Detailed Explanation

### 1. Identify the New Disk
Before doing anything, you must identify the device name assigned to the new disk (e.g., `/dev/sdb`, `/dev/nvme1n1`).

```bash
# List all block devices
lsblk

# Show detailed partition information for all disks
sudo fdisk -l
```

### 2. Partition the Disk
Partitioning divides the disk into one or more logical volumes.

#### Using `fdisk` (Traditional, best for MBR or disks < 2TB)
`fdisk` is an interactive menu-driven tool.
```bash
sudo fdisk /dev/sdb
# Common commands inside fdisk:
# 'n' - Create new partition
# 'p' - Print partition table
# 'w' - Write changes and exit
# 'q' - Quit without saving
```

#### Using `parted` (Modern, supports GPT and > 2TB)
`parted` can be used interactively or via command line.
```bash
# Create a GPT partition table
sudo parted /dev/sdb mklabel gpt

# Create a primary partition using the whole disk
sudo parted /dev/sdb mkpart primary ext4 0% 100%
```

### 3. Format the Partition (Filesystem)
Once partitioned (e.g., `/dev/sdb1`), you must create a filesystem so the OS can write files.

```bash
# Format as ext4 (standard for many distros)
sudo mkfs.ext4 /dev/sdb1

# Format as XFS (high performance, standard on RHEL/CentOS)
sudo mkfs.xfs /dev/sdb1
```

### 4. Mounting the Filesystem
Mounting attaches the formatted partition to a directory (mount point).

#### Temporary Mount
```bash
# Create mount point
sudo mkdir -p /mnt/data

# Mount the device
sudo mount /dev/sdb1 /mnt/data
```

#### Persistent Mount (via `/etc/fstab`)
To ensure the disk mounts automatically on reboot, add an entry to `/etc/fstab`. It is best practice to use the **UUID**.

```bash
# Get the UUID of the partition
blkid /dev/sdb1

# Example /etc/fstab entry:
# UUID=550e8400-e29b-41d4-a716-446655440000  /mnt/data  ext4  defaults  0  2
```

## Interview Questions

**Q: What is the main difference between MBR and GPT partition tables?**
**A:** MBR (Master Boot Record) is an older standard that supports a maximum of 4 primary partitions and disk sizes up to 2TB. GPT (GUID Partition Table) is the modern standard, supporting up to 128 partitions and virtually unlimited disk sizes (zetabytes).

**Q: Why is it recommended to use UUID in `/etc/fstab` instead of device names like `/dev/sdb1`?**
**A:** Device names can change between reboots (e.g., if disks are added or removed, or if the boot order changes in BIOS). The UUID (Universally Unique Identifier) is unique to the filesystem itself and remains constant regardless of the hardware connection order.

**Q: How do you verify that a disk was mounted correctly?**
**A:** You can use `df -h` to see mounted filesystems and their usage, `mount | grep /dev/sdX` to see mount options, or `lsblk` to see the mapping between devices and mount points.

**Q: What command would you use to change the filesystem of an existing partition to XFS?**
**A:** You would use `mkfs.xfs /dev/sdXn`. Note that this is a destructive operation and will erase all existing data on that partition.
