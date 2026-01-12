---
tags: ['linux', 'roadmap']
---

# Logical Volume Manager (LVM)

## Summary
Logical Volume Manager (LVM) is a device mapper framework that provides logical volume management for the Linux kernel. It adds a layer of abstraction between physical storage devices and the file system, allowing for flexible disk management, including dynamic resizing of volumes, spanning volumes across multiple disks, and creating point-in-time snapshots.

## Detailed Explanation
LVM operates through a hierarchical structure of three main components:

### 1. Physical Volume (PV)
A PV is a physical storage device, such as a hard drive, a solid-state drive, or a partition (e.g., \`/dev/sdb1\`), that has been initialized for use by LVM.
*   **Command**: \`pvcreate\`
\`\`\`bash
# Initialize a disk as a Physical Volume
sudo pvcreate /dev/sdb
# View physical volumes
sudo pvs
\`\`\`

### 2. Volume Group (VG)
A VG is a central pool of storage created by combining one or more Physical Volumes. It acts as a single administrative unit.
*   **Command**: \`vgcreate\`
\`\`\`bash
# Create a Volume Group named 'data_vg' using two disks
sudo vgcreate data_vg /dev/sdb /dev/sdc
# View volume groups
sudo vgs
\`\`\`

### 3. Logical Volume (LV)
An LV is a "virtual partition" carved out of a Volume Group. This is the device that is formatted with a filesystem (like ext4 or XFS) and mounted for use.
*   **Command**: \`lvcreate\`
\`\`\`bash
# Create a 20GB Logical Volume named 'app_lv' in 'data_vg'
sudo lvcreate -L 20G -n app_lv data_vg
# Format and mount
sudo mkfs.ext4 /dev/data_vg/app_lv
sudo mount /dev/data_vg/app_lv /mnt/app
\`\`\`

### Flexibility and Resizing
LVM's greatest advantage is the ability to resize volumes without repartitioning the entire disk.
*   **lvextend**: Increases the size of an LV. Use the \`-r\` flag to automatically resize the underlying filesystem as well.
\`\`\`bash
# Extend LV by 10GB and resize filesystem automatically
sudo lvextend -r -L +10G /dev/data_vg/app_lv
\`\`\`

### Snapshots
LVM snapshots are point-in-time copies of a logical volume. They do not initially consume much space; they only store the changes (Copy-on-Write) made to the original volume after the snapshot was taken.
\`\`\`bash
# Create a 5GB snapshot of 'app_lv'
sudo lvcreate -L 5G -s -n app_lv_snap /dev/data_vg/app_lv
\`\`\`

## Interview Questions

**Q: What is the difference between a Physical Volume (PV) and a Logical Volume (LV)?**
**A:** A Physical Volume is the actual storage hardware (or partition) initialized for LVM. A Logical Volume is a virtual partition created from a Volume Group (which pools PVs) and is where the filesystem resides.

**Q: How do you increase the size of an LVM volume that is already mounted?**
**A:** You use the \`lvextend\` command. For example, \`lvextend -r -L +5G /dev/vg_name/lv_name\`. The \`-r\` (or \`--resizefs\`) flag ensures that the underlying filesystem (if supported, like ext4 or XFS) is grown to match the new size of the logical volume.

**Q: What happens if an LVM snapshot runs out of space?**
**A:** If the snapshot volume's capacity is exceeded by the amount of changed data on the origin volume, the snapshot becomes invalid and unusable. It must be removed and recreated with more space if needed.

**Q: Can a Logical Volume span across multiple physical disks?**
**A:** Yes. Since a Volume Group can consist of multiple Physical Volumes, a Logical Volume created within that group can utilize space from any or all of the PVs in that pool.
