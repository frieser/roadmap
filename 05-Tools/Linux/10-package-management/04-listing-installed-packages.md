#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Listing installed packages is a critical skill for system auditing, troubleshooting, and replicating system environments. By querying the local package database, administrators can verify which software versions are present, identify unnecessary packages for removal, and generate manifests for automated deployments. Different Linux distributions use specialized tools like **dpkg**, **apt**, **rpm**, and **dnf** to interact with their respective package databases.

## Detailed Explanation

Linux package managers maintain a local database of all installed software. Accessing this data varies depending on whether the system uses the Debian (`.deb`) or Red Hat (`.rpm`) package format.

### 1. Debian/Ubuntu Systems
Debian-based systems primarily use `dpkg` as the low-level tool and `apt` as the high-level interface.

#### Key Commands:
- **`dpkg -l`**: Lists all packages known to the system. The `ii` prefix indicates a package is "Installed and OK".
- **`apt list --installed`**: The modern high-level command. It provides a more readable output, including architecture and update status.
- **`dpkg-query -W`**: Useful for scripting; it allows custom output formatting using the `-f` flag.

#### Bash Example:
```bash
# List all packages using dpkg and filter for 'python'
dpkg -l | grep python

# List only installed packages using the modern apt interface
apt list --installed

# Get a clean list of package names and versions for a manifest
dpkg-query -W -f='${Package} ${Version}\n' > installed_packages.txt
```

### 2. RHEL/CentOS/Fedora Systems
Red Hat-based systems use `rpm` for low-level queries and `dnf` (or the older `yum`) for high-level management.

#### Key Commands:
- **`rpm -qa`**: Queries all (`-a`) installed packages. It is fast and produces a simple list of package names.
- **`dnf list installed`**: Provides a categorized list of installed packages, often separating those installed from repositories vs. those installed manually.
- **`rpm -qi <package>`**: Displays detailed information (version, install date, description) for a specific installed package.

#### Bash Example:
```bash
# List all installed packages and count them
rpm -qa | wc -l

# Search for the 'kernel' package version
rpm -qa | grep kernel

# List installed packages using DNF
dnf list installed
```

### 3. Arch Linux (Pacman)
Arch Linux uses a single unified tool, `pacman`, for both local queries and remote installations.

#### Key Commands:
- **`pacman -Q`**: Lists all locally installed packages.
- **`pacman -Qe`**: Lists explicitly installed packages (excluding dependencies).
- **`pacman -Qi <package>`**: Shows detailed local package information.

### Comparison Table

| Distribution | Low-Level Command | High-Level Command | Search Installed |
| :--- | :--- | :--- | :--- |
| **Debian/Ubuntu** | `dpkg -l` | `apt list --installed` | `dpkg -l \| grep <name>` |
| **RHEL/CentOS** | `rpm -qa` | `dnf list installed` | `rpm -qa \| grep <name>` |
| **Arch Linux** | `pacman -Q` | `pacman -Q` | `pacman -Qs <name>` |

## Interview Questions

**Q: How do you list all installed packages on a Debian system and save only the package names to a file?**
**A:** You can use `dpkg --get-selections | awk '{print $1}' > packages.txt` or `dpkg-query -f='${Package}\n' -W > packages.txt`.

**Q: What is the significance of the `ii` status in the output of `dpkg -l`?**
**A:** The first `i` indicates the desired state is "Install", and the second `i` indicates the current status is "Installed". Together, `ii` confirms the package is correctly installed on the system.

**Q: How can you find out when a specific package was installed on an RPM-based system?**
**A:** You can use the command `rpm -qi <package_name>` and look for the "Install Date" field in the output.

**Q: How do you list only the packages that were explicitly installed by the user on Arch Linux, ignoring dependencies?**
**A:** Use the command `pacman -Qe`. This is particularly useful for identifying the core software you chose to install without the noise of their dependencies.

**Q: Why might you prefer `apt list --installed` over `dpkg -l` in a modern terminal?**
**A:** `apt list --installed` provides color-coded output, highlights whether a package was automatically installed as a dependency, and indicates if an upgrade is available, making it more user-friendly for interactive use.
