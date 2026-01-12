---
tags: ['linux', 'roadmap']
---

# Mounting (mount, umount, /etc/fstab)

## Summary
In Linux, **mounting** is the process of attaching a filesystem (from a partition, external drive, or network share) to a specific directory in the system's unified directory tree, known as a **mount point**. This makes the contents of the device accessible to the user. The `mount` and `umount` commands handle temporary operations, while the `/etc/fstab` file defines persistent mounts that should be initialized automatically during system boot.

## Detailed Explanation

### 1. The Unified Directory Tree
Unlike Windows, which uses drive letters (C:, D:), Linux uses a single hierarchical tree starting at the root (`/`). Any storage device must be "hooked" into this tree at a mount point (an empty directory) to be used.

### 2. The `mount` Command
The `mount` command is used to manually attach filesystems.

**Basic Syntax:**
```bash
mount [-t fstype] [-o options] <device> <mount_point>
```

**Common Examples:**
```bash
# Mount a partition (e.g., /dev/sdb1) to /mnt/data
sudo mount /dev/sdb1 /mnt/data

# Mount an ISO image (loop device)
sudo mount -o loop ubuntu.iso /mnt/iso

# Mount with specific options (read-only)
sudo mount -o ro /dev/sdc1 /mnt/usb
```

**Common Mount Options:**
| Option | Description |
| :--- | :--- |
| `rw` / `ro` | Mount as Read-Write or Read-Only. |
| `noatime` | Do not update access times on files (improves performance). |
| `nodev` | Do not interpret character or block special devices on the filesystem. |
| `nosuid` | Block the operation of setuid and setgid bits. |
| `noexec` | Do not allow execution of any binaries on the mounted filesystem. |
| `user` | Allow an ordinary user to mount the filesystem. |

### 3. The `umount` Command
Used to detach a filesystem. Note that the command is `umount` (not "unmount").

```bash
# Unmount by mount point
sudo umount /mnt/data

# Unmount by device
sudo umount /dev/sdb1
```

**Troubleshooting "Target is Busy":**
If a process is using a file in the mount point, you cannot unmount it.
```bash
# Find which processes are using the mount point
lsof /mnt/data
fuser -m /mnt/data

# Forced unmount (use with caution)
sudo umount -f /mnt/data  # Force (for unreachable NFS)
sudo umount -l /mnt/data  # Lazy unmount (detaches now, cleans up later)
```

### 4. Persistent Mounts: `/etc/fstab`
To ensure a disk is mounted automatically at boot, it must be added to `/etc/fstab`.

**Structure of an fstab entry:**
```text
<device>    <mount_point>    <type>    <options>    <dump>    <pass>
```

**Example entry:**
```text
UUID=550e8400-e29b-41d4-a716-446655440000  /mnt/data  ext4  defaults,noatime  0  2
```

- **Device**: It is best practice to use the **UUID** instead of `/dev/sdX` because device names can change between reboots. Find it using `blkid`.
- **Mount Point**: The directory where it will be accessible.
- **Type**: Filesystem type (ext4, xfs, vfat, ntfs-3g, nfs).
- **Options**: Comma-separated list (e.g., `defaults` includes `rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, and `async`).
- **Dump**: Used by the `dump` utility (usually 0).
- **Pass**: Used by `fsck` to determine the order of filesystem checks at boot (1 for root, 2 for others, 0 to skip).

**Testing fstab changes:**
After editing `/etc/fstab`, always run:
```bash
sudo mount -a
```
This command mounts all filesystems mentioned in `fstab` that aren't already mounted. If there is an error in the file, this will catch it before you reboot (avoiding a boot failure).

### 5. Systemd Mount Units
In modern distributions, `systemd` handles mounts. When you add an entry to `/etc/fstab`, `systemd` automatically generates a mount unit (e.g., `mnt-data.mount`). You can also create these units manually in `/etc/systemd/system/`.

## Interview Questions

**Q: What is the advantage of using UUIDs in `/etc/fstab` instead of device names like `/dev/sdb1`?**
**A:** Device names are assigned by the kernel in the order they are discovered. Adding or removing hardware can cause `/dev/sdb1` to become `/dev/sdc1`, which would break the mount or mount the wrong partition. UUIDs are unique to the filesystem itself and remain constant regardless of the hardware connection order.

**Q: How can you find out which filesystems are currently mounted on your system?**
**A:** You can use the `mount` command (without arguments), `df -h` for a human-readable summary of space and mount points, or `findmnt` for a tree-like overview.

**Q: What does the `mount -a` command do?**
**A:** It attempts to mount all filesystems listed in `/etc/fstab` that are marked with the `auto` option (which is included in `defaults`) and are not currently mounted.

**Q: If you get a "device is busy" error when trying to `umount`, how do you identify the culprit?**
**A:** Use `lsof <mount_point>` to list open files and the processes that own them, or `fuser -v <mount_point>` to see the PIDs of processes accessing the filesystem.
