---
---

## Summary
`chgrp` (Change Group) changes the group ownership of a file. Unlike `chown`, a regular user *can* use `chgrp` to change a file's group to any group they are a member of.

## Detailed Explanation

### Syntax
`chgrp [options] group file`

### Use Cases
*   **Collaboration**: Setting a file to the `developers` group so teammates can edit it (assuming `g+w` permission is set).
*   **Services**: ensuring log files are readable by the `syslog` group.

### Recursion
Like `chown`, `chgrp -R dir` applies changes recursively.

## Go-Specific Context/Examples

Go does not have a separate `os.Chgrp` function. You use `os.Chown(path, -1, gid)`. Passing `-1` as the UID tells the system "keep the current owner unchanged".

### Analogy
*   **Bash**: `chgrp 1000 file.txt`
*   **Go**: `os.Chown("file.txt", -1, 1000)`

## Interview Questions

**Q: What is the difference between `chown :group` and `chgrp group`?**
**A:** Functionally, they are identical. `chown` is the more powerful tool (supersedes `chgrp`), but `chgrp` remains for historical POSIX compliance and habit.

**Q: Why would `chgrp` fail for a regular user?**
**A:** If the user does not own the file, OR if the user is not a member of the target group. You cannot change a file to a group you don't belong to (unless you are root).

**Q: Does changing the group affect permissions?**
**A:** Indirectly. It changes *who* the "Group" permissions apply to. If permissions are `770` (User/Group full access), changing the group grants full access to all members of the new group and revokes it from the old group.
