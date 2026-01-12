#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Swap space is a designated area on a storage drive that the Linux kernel uses as virtual memory when physical RAM is full. It allows the system to run more applications than would otherwise fit in RAM by moving inactive memory pages to disk. Swap can be implemented as either a dedicated **swap partition** or a **swap file**.

## Detailed Explanation

### Purpose of Swap
Swap acts as a "safety net" for memory-intensive workloads. While disk access is much slower than RAM, having swap prevents the system from crashing or killing processes (OOM Killer) when physical memory is exhausted. It also allows the kernel to swap out rarely used memory pages to disk, freeing up high-speed RAM for active processes and disk caching.

### Swap Partition vs. Swap File
- **Swap Partition**: A dedicated block device. Historically faster because it avoids filesystem overhead, but harder to resize without repartitioning.
- **Swap File**: A regular file on an existing filesystem. Modern Linux systems (like Ubuntu) use swap files by default because they are easy to create, delete, and resize without touching partitions.

### Managing Swap Space (Bash Examples)

#### 1. Creating and Activating a Swap File
To create a 2GB swap file:
```bash
# 1. Create a file of the desired size (e.g., 2GB)
# 'fallocate' is faster than 'dd' for creating large files
sudo fallocate -l 2G /swapfile

# 2. Set strict permissions (only root should read/write)
# This is CRITICAL for security
sudo chmod 600 /swapfile

# 3. Mark the file as swap space
sudo mkswap /swapfile

# 4. Enable the swap file
sudo swapon /swapfile

# 5. Verify the swap is active
swapon --show
# OR
free -h
```

#### 2. Persistent Swap (/etc/fstab)
To ensure swap is enabled after a reboot, add an entry to `/etc/fstab`:
```bash
# Edit /etc/fstab and add this line:
# /swapfile none swap sw 0 0

# You can use 'tee' to append it safely:
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### 3. Deactivating and Removing Swap
```bash
# 1. Deactivate the swap
sudo swapoff /swapfile

# 2. Remove the entry from /etc/fstab (manually edit /etc/fstab)

# 3. Delete the swap file
sudo rm /swapfile
```

### Swappiness
The `swappiness` parameter (0-100) controls how aggressively the kernel uses swap.
- **Low value (e.g., 10)**: Favors RAM, avoids swapping until absolutely necessary.
- **High value (e.g., 60-100)**: Swaps more aggressively.

```bash
# Check current swappiness
cat /proc/sys/vm/swappiness

# Set swappiness temporarily (until reboot)
sudo sysctl vm.swappiness=10

# Set swappiness permanently
# Add 'vm.swappiness=10' to /etc/sysctl.conf
```

## Interview Questions

**Q: What is the difference between a swap partition and a swap file?**
**A:** A swap partition is a dedicated section of the hard drive with no filesystem, whereas a swap file is a large file within an existing filesystem. Swap files are more flexible because they can be easily resized or moved without repartitioning the disk, while partitions were historically preferred for slightly better performance by avoiding filesystem overhead.

**Q: Why is it important to run `chmod 600` on a swap file?**
**A:** Swap space contains the contents of RAM, which may include sensitive information like passwords, encryption keys, or private data from any running process. Setting permissions to `600` ensures that only the root user can read or write to the swap file, preventing local users from accessing sensitive data.

**Q: How do you make a swap file permanent across reboots?**
**A:** You must add an entry to the `/etc/fstab` file. The entry follows this format: `/path/to/swapfile none swap sw 0 0`. This tells the Linux initialization process to mount and enable the swap space during boot.

**Q: What does the `swappiness` parameter do, and how can you change it?**
**A:** `swappiness` is a kernel parameter (0-100) that determines the balance between swapping out runtime memory and clearing caches. A lower value makes the kernel prefer keeping data in RAM, while a higher value encourages swapping. It can be changed temporarily using `sysctl vm.swappiness=X` or permanently by adding the setting to `/etc/sysctl.conf`.
