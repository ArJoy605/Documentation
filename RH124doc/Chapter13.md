# RH124 Revision Notebook: Chapter 13 - Archive and Transfer Files

This notebook provides a detailed summary of Chapter 13 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on archiving files and transferring them securely between systems, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 13: Archive and Transfer Files

### Key Concepts

- **Archiving**:
  - Archiving combines multiple files or directories into a single file (e.g., `.tar`) for backup or transfer.
  - Common tool: `tar` (Tape Archive), which supports creating, extracting, and listing archives.
- **Compression**:
  - Reduces file size for efficient storage or transfer.
  - Common compression tools: `gzip` (`.gz`), `bzip2` (`.bz2`), `xz` (`.xz`).
  - Combined with `tar` for compressed archives (e.g., `.tar.gz`, `.tar.xz`).
- **File Transfer**:
  - Secure transfer over networks using tools like `scp` (Secure Copy) and `rsync`.
  - `scp` uses SSH for encryption, suitable for one-time transfers.
  - `rsync` optimizes transfers by copying only differences, ideal for backups or synchronization.
- **Key Files and Directories**:
  - Archives are often stored in `/backup` or user-defined locations.
  - Temporary files for transfers may use `/tmp` (cleared on reboot).
- **RHEL 9/10 Notes**:
  - RHEL 10 improves `rsync` performance for large datasets.
  - RHEL Lightspeed (RHEL 10) can suggest archive/transfer commands (e.g., `rhel lightspeed "backup home directory"` suggests `tar` commands).

### Important Commands

- **Creating Archives with `tar`**:

  ```bash
  tar -cvf archive.tar file1 file2  # Create archive (c: create, v: verbose, f: file)
  tar -czvf archive.tar.gz /home/user1  # Create compressed archive with gzip
  tar -cjvf archive.tar.bz2 /home/user1  # Use bzip2 compression
  tar -cJvf archive.tar.xz /home/user1  # Use xz compression
  ```

- **Listing Archive Contents**:

  ```bash
  tar -tvf archive.tar  # List contents (t: list)
  tar -tzvf archive.tar.gz  # List contents of gzip archive
  ```

- **Extracting Archives**:

  ```bash
  tar -xvf archive.tar  # Extract to current directory (x: extract)
  tar -xzvf archive.tar.gz -C /path/to/dir  # Extract to specific directory
  tar -xjvf archive.tar.bz2  # Extract bzip2 archive
  tar -xJvf archive.tar.xz  # Extract xz archive
  ```

- **Compression without `tar`**:

  ```bash
  gzip file.txt  # Compress to file.txt.gz
  gunzip file.txt.gz  # Decompress
  bzip2 file.txt  # Compress to file.txt.bz2
  bunzip2 file.txt.bz2  # Decompress
  xz file.txt  # Compress to file.txt.xz
  unxz file.txt.xz  # Decompress
  ```

- **Secure File Transfer with `scp`**:

  ```bash
  scp file.txt user1@remote:/home/user1  # Copy to remote
  scp user1@remote:/remote/file.txt .  # Copy from remote to current directory
  scp -r /local/dir user1@remote:/remote/dir  # Copy directory recursively
  ```

- **Synchronizing Files with `rsync`**:

  ```bash
  rsync -av /local/dir user1@remote:/remote/dir  # Sync directory (a: archive, v: verbose)
  rsync -av --delete /local/dir user1@remote:/remote/dir  # Sync and delete unmatched files
  rsync -avz /local/dir user1@remote:/remote/dir  # Sync with compression
  ```

### Practical Examples

- **Scenario: Backup Home Directory**:
  - Create a compressed archive of `/home/user1`:

    ```bash
    tar -czvf /backup/user1_$(date +%F).tar.gz /home/user1
    ```

  - Verify contents: `tar -tzvf /backup/user1_2025-08-11.tar.gz`.
- **Scenario: Restore Backup**:
  - Extract archive to `/restore`:

    ```bash
    mkdir /restore
    tar -xzvf /backup/user1_2025-08-11.tar.gz -C /restore
    ls /restore/home/user1  # Verify
    ```

- **Scenario: Transfer Files Securely**:
  - Copy `report.pdf` to remote server:

    ```bash
    scp report.pdf user1@192.168.1.100:/home/user1
    ```

  - Verify on remote: `ssh user1@192.168.1.100 ls /home/user1`.
- **Scenario: Synchronize Project Directory**:
  - Sync `/project` to remote server, preserving permissions:

    ``` bash
    rsync -av /project user1@192.168.1.100:/project
    ```

  - Update only changed files: `rsync -av --update /project user1@192.168.1.100:/project`.
- **Daily Use: Compress Log Files**:
  - Compress old logs:

    ```bash
    gzip /var/log/old.log
    ls /var/log/old.log.gz  # Verify
    ```

- **Scenario: Incremental Backup with `rsync`**:
  - Back up `/data` to remote, excluding temporary files:

    ```bash
    rsync -av --exclude '*.tmp' /data user1@remote:/backup/data
    ```

### Common Pitfalls

- **Forgetting Compression Flags**: Using `tar -cvf` without `z`, `j`, or `J` creates uncompressed archives, wasting space.
- **Overwriting Files**: `tar -xvf` or `scp` can overwrite files without warning. Check destination first (`ls` or `tar -tvf`).
- **Permission Issues**: Transfers to system directories (e.g., `/etc`) or archives require `sudo`. Fix: `sudo scp` or `sudo tar`.
- **SSH Configuration**: `scp` and `rsync` require SSH access. Ensure `sshd` is running and firewall allows port 22 (`sudo firewall-cmd --list-all`).
- **Large Transfers with `scp`**: `scp` copies entire files, which is slow for large datasets. Use `rsync` for incremental transfers.
- **Incorrect `rsync` Options**: Omitting `--delete` in `rsync` can leave outdated files on the destination. Verify with `--dry-run`.

### Best Practices and Tips

- **Use Compression for Archives**: Prefer `xz` for best compression ratio (`tar -cJvf`) or `gzip` for speed (`tar -czvf`).
- **Verify Archives**: Always check contents with `tar -tvf` before extracting or transferring.
- **Use `rsync` for Backups**: Its incremental nature saves time and bandwidth (e.g., `rsync -avz`).
- **Secure Transfers**: Always use `scp` or `rsync` over SSH for encryption; avoid insecure tools like `ftp`.
- **Automate Backups**: Script daily backups:

  ```bash
  # backup.sh
  tar -czvf /backup/home_$(date +%F).tar.gz /home
  rsync -avz /backup/home_$(date +%F).tar.gz user1@remote:/backup
  ```

  Run: `chmod +x backup.sh; ./backup.sh`.
- **RHEL 10 Tip**: Use Lightspeed for archive tasks (e.g., `rhel lightspeed "create tar archive"` suggests `tar -czvf`).
- **Check Disk Space**: Before archiving or transferring, verify space with `df -h`.

### Revision Quiz/Notes

- **Questions**:
  - What command creates a gzip-compressed archive? (`tar -czvf archive.tar.gz files`)
  - How do you copy a directory to a remote server? (`scp -r /dir user1@remote:/dest` or `rsync -av /dir user1@remote:/dest`)
  - What does `rsync --delete` do? (Removes files in destination not present in source.)
- **Quick Exercise**:
  - Create a compressed archive of `/etc`, transfer it to a remote server, and verify:

    ```bash
    sudo tar -czvf /backup/etc_$(date +%F).tar.gz /etc
    scp /backup/etc_$(date +%F).tar.gz user1@192.168.1.100:/backup
    ssh user1@192.168.1.100 ls /backup
    ```

- **Self-Test**:
  - Explain the difference between `scp` and `rsync` (`scp` for one-time transfers, `rsync` for incremental sync).
  - Why use `tar -C`? (Extracts to a specified directory, avoids clutter.)

---
