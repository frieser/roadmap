---
tags: ['linux', 'roadmap']
---

# Available Memory and Disk (free -h, df -h, du, vmstat)

## Summary
Monitoring system resources is a fundamental task for Linux administrators. Understanding how much memory and disk space is available ensures system stability and helps in troubleshooting performance bottlenecks. Key tools like \`free\` and \`vmstat\` provide insights into RAM and virtual memory usage, while \`df\` and \`du\` are essential for managing storage capacity.

## Detailed Explanation

### 1. Memory Monitoring

#### \`free -h\`
The \`free\` command displays the total amount of free and used physical and swap memory in the system, as well as the buffers and caches used by the kernel. The \`-h\` flag stands for "human-readable."

\`\`\`bash
free -h
\`\`\`

**Key Columns:**
- **total**: Total installed memory (MemTotal in \`/proc/meminfo\`).
- **used**: Memory currently in use by applications and the OS.
- **free**: Memory not being used for *anything*.
- **buff/cache**: Memory used by the kernel for disk buffers and page cache. This is *reclaimable* memory.
- **available**: An estimate of how much memory is available for starting new applications without swapping. This is the most important metric to watch.

> **Note:** Linux follows the philosophy that "Free RAM is wasted RAM." It uses idle memory for caching files to speed up disk I/O. If an application needs it, the kernel immediately drops caches to provide memory.

#### \`vmstat\`
\`vmstat\` (Virtual Memory Statistics) reports information about processes, memory, paging, block IO, traps, and cpu activity.

\`\`\`bash
# Report every 2 seconds for 5 times
vmstat 2 5
\`\`\`

**Critical Fields:**
- **si / so**: Swap-In and Swap-Out. If these are consistently above zero, the system is under memory pressure and is swapping to disk, which significantly degrades performance.
- **r / b**: Processes in running queue (\`r\`) or blocked/waiting for IO (\`b\`).
- **wa**: Time spent waiting for IO. High \`wa\` often points to disk bottlenecks.

---

### 2. Disk Monitoring

#### \`df -h\`
\`df\` (disk free) displays the amount of disk space available on file systems.

\`\`\`bash
df -h
\`\`\`

**Common usage:**
- \`df -h .\`: Show usage for the current directory's partition.
- \`df -i\`: Show inode usage (sometimes you run out of inodes before disk space).

#### \`du\`
\`du\` (disk usage) estimates file space usage for directories.

\`\`\`bash
# Summary of current directory in human-readable format
du -sh

# Show the size of top-level subdirectories/files in the current folder
du -h --max-depth=1 | sort -hr
\`\`\`

**Common flags:**
- \`-s\`: Summary.
- \`-h\`: Human readable.
- \`-c\`: Grand total.

---

## Interview Questions

**Q: What is the difference between 'free' and 'available' memory in the \`free -h\` output?**
**A:** 'Free' memory is RAM that is absolutely not in use (not even for caching). 'Available' memory is an estimate of how much memory can be allocated to new processes without causing the system to swap. It includes 'free' memory plus a portion of the memory currently used for buffers and cache that can be reclaimed.

**Q: If \`df -h\` shows 50% free space but you get a "No space left on device" error, what could be the cause?**
**A:** You might have run out of **Inodes**. Each file and directory requires an inode. If you have many tiny files, you can exhaust the inode table even if plenty of physical disk space remains. Check with \`df -i\`.

**Q: What do high 'si' and 'so' values in \`vmstat\` indicate?**
**A:** They indicate active swapping (Swap-In / Swap-Out). This means the system is reading and writing memory pages to/from the disk because physical RAM is full. This is a clear sign of memory exhaustion and leads to high latency and reduced performance.

**Q: How can you find the top 5 largest directories under \`/var\`?**
**A:** You can use: \`sudo du -h /var --max-depth=1 | sort -hr | head -n 5\`. This summarizes the usage of each immediate subdirectory under \`/var\` and sorts them by size.

**Q: What does the 'buff/cache' column represent? Is it "lost" memory?**
**A:** No, it is not lost. It represents memory used by the Linux kernel to cache disk data (Page Cache) and filesystem metadata (Slab). This memory is effectively "free" in the sense that the kernel will automatically and instantly reclaim it if an application requires more RAM.
