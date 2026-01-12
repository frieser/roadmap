---
tags: ['linux', 'networking', 'roadmap', 'tools']
---

# File Transfer (scp, sftp, rsync)

## Summary
File transfer in Linux systems is primarily handled through secure protocols built on top of SSH. The three most common tools are **scp** (Secure Copy), **sftp** (Secure File Transfer Protocol), and **rsync** (Remote Sync). While they all provide encrypted data transfer, they differ in their efficiency, interactivity, and feature sets. Modern distributions have largely moved `scp` to an SFTP backend to resolve legacy security vulnerabilities.

## Detailed Explanation

### 1. SCP (Secure Copy)
`scp` is used for simple, non-interactive file copies between hosts. It is easy to use but lacks the advanced synchronization features of `rsync`.

*   **Key Command**: `scp [options] source destination`
*   **Security Note**: Historically, `scp` used the RCP protocol, which was prone to security issues (e.g., shell injection). Modern OpenSSH versions (9.0+) use the SFTP protocol under the hood for `scp` by default.

**Examples:**
```bash
# Copy a local file to a remote server
scp file.txt user@remote-host:/path/to/destination/

# Copy a directory recursively from a remote server
scp -r user@remote-host:/path/to/source/ ./local-dir/

# Use a specific SSH port (2222)
scp -P 2222 file.txt user@remote-host:/tmp/
```

### 2. SFTP (Secure File Transfer Protocol)
`sftp` provides an interactive, FTP-like interface but runs entirely over an encrypted SSH tunnel. It is more robust than `scp` and allows for file management tasks (listing, deleting, renaming) within the same session.

*   **Port**: Defaults to 22 (SSH).
*   **Mode**: Interactive or batch mode (`-b`).

**Examples:**
```bash
# Start an interactive session
sftp user@remote-host

# Common interactive commands:
# sftp> ls          # List remote files
# sftp> put local   # Upload file
# sftp> get remote  # Download file
# sftp> mkdir dir   # Create remote directory
# sftp> quit        # Exit session
```

### 3. rsync (Remote Sync)
`rsync` is a highly efficient utility for synchronizing files and directories. It uses a **delta-transfer algorithm**, which minimizes data transfer by only sending the portions of files that have changed.

*   **Key Features**: Compression, preservation of permissions/links, and ability to resume interrupted transfers.
*   **Transport**: Usually uses SSH for remote transfers.

**Examples:**
```bash
# Sync local directory to remote (Archive mode, Verbose, Compressed)
rsync -avz /local/dir/ user@remote-host:/remote/dir/

# Sync and delete files on destination that are no longer on source
rsync -avz --delete /local/dir/ user@remote-host:/remote/dir/

# Resume a partially transferred file
rsync -avz --partial --progress file.zip user@remote-host:/tmp/

# Dry run (test what would be transferred without actually doing it)
rsync -avz --dry-run /local/dir/ user@remote-host:/remote/dir/
```

## Interview Questions

**Q: What makes `rsync` more efficient than `scp` for repeated transfers?**
**A:** `rsync` uses a delta-transfer algorithm. It compares the source and destination files and only transfers the specific chunks of data that have changed, rather than re-sending the entire file. This significantly saves bandwidth and time.

**Q: Why is it recommended to use the SFTP backend for `scp` in modern Linux environments?**
**A:** The legacy SCP protocol was susceptible to shell-command injection attacks because of how it handled wildcards and filenames on the remote side. SFTP provides a cleaner, more secure protocol for the underlying transfer while maintaining the familiar `scp` command-line syntax.

**Q: How do you preserve file permissions and timestamps when using `rsync`?**
**A:** Use the `-a` (archive) flag. This is a shortcut for several options including `-r` (recursive), `-l` (links), `-p` (perms), `-t` (times), `-g` (group), `-o` (owner), and `-D` (devices/special files).

**Q: Which command would you use to sync a local directory to a remote server while removing files at the destination that no longer exist at the source?**
**A:** You would use `rsync` with the `--delete` flag: `rsync -avz --delete /src/ user@host:/dest/`.

**Q: Can `rsync` be used without SSH?**
**A:** Yes, `rsync` can connect to an `rsync daemon` running on port 873. However, this is unencrypted by default. For secure transfers over a network, it is standard practice to use it over SSH (which is the default in most modern versions).
