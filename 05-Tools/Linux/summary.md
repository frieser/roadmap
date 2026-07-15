### 01-Navigation-Basics
| cmd | purpose | | cmd | purpose |
|-----|---------|---|-----|---------|
| `pwd` | current dir | | `ls -lah` | list all files |
| `cd ~`, `cd -` | navigate | | `man cmd`, `--help` | docs |
| `which`, `type -a` | locate bin | | `mv src dst` | rename/move |

- `touch`, `mkdir -p`, `rm -rf`, `rmdir` — FHS: `/` root, `/etc` config, `/var/log`, `/proc`, `/sys`, `/home`

### 02-Editing | 03-Shell-Basics | 04-Files
- **Vim**: modal (`i`/`Esc`/`:`), `hjkl`, `:wq`/`:q!`, `/pat`, `dd`, `yy`, `u`. **Nano**: `^O` save, `^X` exit.
- **PATH**: `export PATH=$PATH:/dir`. **Redirect**: `>`(ow) `>>`(app) `2>`(err) `&>`(both). **sudo/su/visudo**.
- **chmod**: `755`/`u+x`, SUID`4xxx`/SGID`2xxx`/sticky`1xxx`. **chown -R**. **tar -czvf/-xzvf**.
- **cp -a** (preserve), **rsync -avz --delete**. Soft link: `ln -s` (path). Hard link: `ln` (inode).

### 05-Text-Processing
| `grep -ivrnE` | `sed 's/old/new/g' -i` | `awk -F: '{print $1,$NF}'` | `sort -nrk2 -t:` |
| `uniq -c` | `cut -d: -f1`, `paste -d,` | `head -n20`, `tail -f/-F` | `find -name -type -exec` |
| `locate` (DB) | `tee`, `xargs` | `sort -u` vs `sort \| uniq` | `awk 'BEGIN/END', NR, NF` |

### 06-Process-Management
- `&` bg, `^Z` suspend, `jobs`, `fg %N`, `bg %N`, `nohup`, `disown`. `ps aux`, `top`/`htop`.
- Signals: `SIGTERM`(15) graceful, `SIGKILL`(9) forced. `kill`, `pkill`, `killall`, `trap`. Nice: -20..19.
- `fork()` → child copy, `exec()` → replace. Zombie: exited, not reaped. Orphan → adopted by init(1).

### 07-Users | 08-Systemd | 09-Server-Review
- `useradd -m -s`, `usermod -aG`, `userdel -r`. ACLs: `getfacl`, `setfacl -m u:`. `/etc/sudoers.d/`.
- Unit: `[Unit]`(After=), `[Service]`(ExecStart=,Type=), `[Install]`(WantedBy=). `systemctl start/enable --now`.
- `systemctl status`, `is-active --quiet`, `journalctl -u svc -f -b -p err --since`. `daemon-reload`.
- `uptime`(load vs nproc), `last`/`lastb`, `ss -tulpn`, `free -h`(available), `df -h`/`-i`, `du -sh --max-depth=1`.

### 10-Packages | 11-Disks | 12-Boot
- Debian: `apt install/search`, `dpkg -l`. RHEL: `dnf install/search`, `rpm -qa`. Snap: SquashFS, --classic.
- **Inode**: metadata(not filename), `ls -i`, `stat`, `df -i`. **ext4**/XFS/Btrfs(CoW). `mount`, `umount -l`.
- **fstab**: UUID. **LVM**: PV→VG→LV, `lvcreate`, `lvextend -r`. Swap: `fallocate`, `mkswap`, `swapon`, `swappiness`.
- **BIOS**(MBR) vs **UEFI**(GPT,ESP). GRUB2: `update-grub`. initramfs. `dmesg -T`, `journalctl -b -1`.

### 13-Networking
- **TCP/IP**: 4-layer. TCP SYN→SYN-ACK→ACK vs UDP. CIDR `/24`, RFC1918: `10/8 172.16/12 192.168/16`.
- **ARP**: IP→MAC, `ip neigh`. **DHCP DORA**: `dhclient`. **Routing**: `ip route`, default gw, `ip route get`.
- **DNS**: `/etc/hosts`→`resolv.conf`→`nsswitch.conf`. A/AAAA/CNAME/MX/NS. `dig +short`, `nslookup`.
- **Netfilter**: iptables(filter/nat/mangle)→nftables(unified), ufw/firewalld. DROP vs REJECT.
- **SSH**: `ssh-keygen -t ed25519`, `known_hosts`(client), `authorized_keys`(server), ProxyJump.
- `scp`, `sftp`, `rsync -avz --delete`(delta). `scp` now uses SFTP backend (OpenSSH 9+).

### 14-Shell-Programming
- Quoting: `''` literal, `""` expand. `${VAR:-def}`, `${VAR##*/}`, `${VAR%/*}`, `${#VAR}`. `$@` vs `$*`, `$?`.
- `for/while/until`, `while read -r line < f`. `[[ ]]` > `[ ]`, `-e/-f/-d`, `-eq/-gt`, `case esac`.
- **Strict mode**: `set -euo pipefail`. `set -x` trace. `shellcheck`.

### 15-Troubleshooting | 16-Containerization
- `ping -c 4`, `traceroute -I/-T`, `mtr -rw`. `tcpdump -i any -nn port 80 -w out.pcap`. BPF filters.
- **ulimits**: soft vs hard, `nofile`/`nproc`. `ulimit -a`. `/etc/security/limits.conf`.
- **cgroups v2**: unified, `/sys/fs/cgroup`, `memory.max`. OCI: runc(low), containerd/CRI-O(high), CRI.
- **Docker**: Dockerfile `CMD`/`ENTRYPOINT`, volumes, Compose. **LXC/Incus**: system containers, `incus launch`.

### 17-SUSE | 18-RHEL-Derivatives
- **Zypper**: `in`/`rm`/`se`/`up`/`dup` (SAT solver). **YaST**: ncurses admin. **Leap** vs **Tumbleweed**. SLES.
- **Fedora**→CentOS Stream→RHEL→Rocky/Alma. **rpm**(low) vs **dnf**(high). SELinux: `getenforce`, `ls -Z`.
