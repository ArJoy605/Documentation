# Chapter 13: Archive and Transfer Files

## Key Concepts

- **Archiving**:
  - Combines multiple files or directories into a single file (e.g., `.tar`) for backup, storage, or transfer.
  - Managed by `tar` (Tape Archive), a versatile tool for archiving.
- **Compression**:
  - Reduces file size to save space or speed up transfers.
  - Common tools: `gzip` (`.gz`, fast), `bzip2` (`.bz2`, better compression), `xz` (`.xz`, best compression).
  - Often paired with `tar` for compressed archives (e.g., `.tar.gz`, `.tar.xz`).
- **File Transfer**:
  - **scp**: Securely copies files over SSH, ideal for one-time transfers.
  - **sftp**: Interactive file transfer protocol over SSH, suitable for manual uploads/downloads.
  - **rsync**: Synchronizes files, copying only differences, optimized for backups and large datasets.
- **Key Files and Directories**:
  - Archives: Stored in user-defined locations (e.g., `/backup`) or temporary directories (`/tmp`).
  - SSH configuration: Relies on `sshd` for `scp` and `sftp` (port 22, `/etc/ssh/sshd_config`).

### Managing Compressed Tar Archives

- **Purpose**: Combine and compress files/directories for efficient storage or transfer.
- **Tool**: `tar` with compression options (`gzip`, `bzip2`, `xz`).

### The `tar` Command

- **Syntax**: `tar [options] archive_name files/directories`.
- **Key Actions**:
  - Create: `-c` (create archive).
  - List: `-t` (list contents).
  - Extract: `-x` (extract files).
  - File: `-f` (specify archive file).
  - Verbose: `-v` (show progress).
- **Example**:
  - Basic archive creation:

    ```bash
    tar -cvf myarchive.tar /home/user1/documents
    ```

### Selected `tar` Options

- **Common Options**:
  - `-c`: Create archive.
  - `-x`: Extract archive.
  - `-t`: List contents.
  - `-v`: Verbose output.
  - `-f`: Specify archive file.
  - `-C`: Change directory for extraction.
  - `--exclude`: Exclude files/patterns.
- **Examples**:
  1. Create archive excluding `.tmp` files:

     ```bash
     tar -cvf myarchive.tar --exclude='*.tmp' /home/user1
     ```

  2. Extract to specific directory:

     ```bash
     tar -xvf myarchive.tar -C /restore
     ```

  3. Corner Case: Missing `-f`:

     ```bash
     tar -cv /home/user1
     # Error: No file specified
     # Fix: tar -cvf archive.tar /home/user1
     ```

### Compression Options (`gz`, `bz2`, `xz`)

- **Tools**:
  - `gzip`: Fast, moderate compression (`.tar.gz`).
  - `bzip2`: Better compression, slower (`.tar.bz2`).
  - `xz`: Best compression, slowest (`.tar.xz`).
- **Options**:
  - `-z`: Use `gzip`.
  - `-j`: Use `bzip2`.
  - `-J`: Use `xz`.
- **Examples**:
  1. Create `gzip` archive:

     ```bash
     tar -czvf archive.tar.gz /home/user1
     ```

  2. Create `bzip2` archive:

     ```bash
     tar -cjvf archive.tar.bz2 /home/user1
     ```

  3. Create `xz` archive:

     ```bash
     tar -cJvf archive.tar.xz /home/user1
     ```

  4. Corner Case: Wrong compression flag:

     ```bash
     tar -xzvf archive.tar.bz2
     # Error: Not a gzip file
     # Fix: tar -xjvf archive.tar.bz2
     ```

### Listing Options of the `tar` Command

- **Command**: `tar -tvf` to list archive contents.
- **Examples**:
  1. List contents of `.tar`:

     ```bash
     tar -tvf myarchive.tar
     ```

  2. List `gzip` archive:

     ```bash
     tar -tzvf archive.tar.gz
     ```

  3. List specific file:

     ```bash
     tar -tzvf archive.tar.gz | grep document.txt
     ```

  4. Corner Case: Corrupted archive:

     ```bash
     tar -tzvf corrupt.tar.gz
     # Error: Invalid archive
     # Fix: Recreate archive or use backup
     ```

### Archiving Files and Directories

- **Purpose**: Combine multiple files/directories into a single `.tar` file.
- **Examples**:
  1. Archive `/etc`:

     ```bash
     sudo tar -cvf /backup/etc_$(date +%F).tar /etc
     ```

  2. Archive multiple directories:

     ```bash
     tar -cvf project.tar /home/user1/project /var/log
     ```

  3. Exclude large files:

     ```bash
     tar -cvf data.tar --exclude='*.log' /data
     ```

  4. Corner Case: Permission denied:

     ```bash
     tar -cvf /root/backup.tar /root
     # Error: Permission denied
     # Fix: sudo tar -cvf /backup/root.tar /root
     ```

### Listing Contents of an Archive

- **Command**: `tar -tvf` (with `-z`, `-j`, or `-J` for compressed archives).
- **Examples**:
  1. List `.tar.gz` contents:

     ```bash
     tar -tzvf backup.tar.gz
     ```

  2. Filter specific files:

     ```bash
     tar -tzvf backup.tar.gz | grep config
     ```

  3. Check file permissions:

     ```bash
     tar -tvf archive.tar
     ```

  4. Corner Case: Empty archive:

     ```bash
     tar -tvf empty.tar
     # No output
     # Fix: Verify source: ls /source/dir
     ```

### Extracting Files from an Archive

- **Command**: `tar -xvf` (with `-z`, `-j`, or `-J`).
- **Examples**:
  1. Extract to current directory:

     ```bash
     tar -xvf archive.tar
     ```

  2. Extract `gzip` archive to `/restore`:

     ```bash
     mkdir /restore
     tar -xzvf archive.tar.gz -C /restore
     ```

  3. Extract specific file:

     ```bash
     tar -xzvf archive.tar.gz home/user1/document.txt
     ```

  4. Corner Case: Overwriting files:

     ```bash
     tar -xvf archive.tar
     # Warning: Overwrites existing files
     # Fix: Use -C or check: ls /destination
     ```

### Creating a Compressed Archive

- **Purpose**: Combine and compress files for efficient storage/transfer.
- **Examples**:
  1. Create `gzip` archive:

     ```bash
     tar -czvf logs_$(date +%F).tar.gz /var/log
     ```

  2. Create `xz` archive for best compression:

     ```bash
     tar -cJvf data.tar.xz /data
     ```

  3. Archive with progress:

     ```bash
     tar -czvf backup.tar.gz /home | pv  # Requires pv package
     ```

  4. Corner Case: Insufficient disk space:

     ```bash
     tar -czvf /backup/full.tar.gz /home
     # Error: No space
     # Fix: df -h; clear space or use another disk
     ```

### Extracting a Compressed Archive

- **Purpose**: Restore files from a compressed archive.
- **Examples**:
  1. Extract `bzip2` archive:

     ```bash
     tar -xjvf archive.tar.bz2
     ```

  2. Extract to specific directory:

     ```bash
     mkdir /restore
     tar -xJvf data.tar.xz -C /restore
     ```

  3. Extract single directory:

     ```bash
     tar -xzvf backup.tar.gz home/user1/documents
     ```

  4. Corner Case: Wrong compression:

     ```bash
     tar -xzvf archive.tar.xz
     # Error: Not gzip
     # Fix: tar -xJvf archive.tar.xz
     ```

### Transferring Files Between Systems Securely

- **Tools**:
  - `scp`: Secure copy over SSH for one-time transfers.
  - `sftp`: Interactive file transfer over SSH.
  - `rsync`: Efficient synchronization over SSH.
- **Requirement**: SSH access (port 22 open, `sshd` running).

### Transferring Files with `scp`

- **Syntax**: `scp [source] [destination]`.
- **Examples**:
  1. Copy file to remote:

     ```bash
     scp report.pdf user1@192.168.1.100:/home/user1
     ```

  2. Copy directory recursively:

     ```bash
     scp -r /local/project user1@192.168.1.100:/project
     ```

  3. Copy from remote:

     ```bash
     scp user1@192.168.1.100:/remote/data.txt .
     ```

  4. Corner Case: SSH port changed:

     ```bash
     scp -P 2222 file.txt user1@server:/home/user1
     ```

### Transferring Files with `sftp`

- **Purpose**: Interactive file transfers over SSH.
- **Commands**: `put` (upload), `get` (download), `ls`, `cd`, `exit`.
- **Examples**:
  1. Start `sftp` session:

     ```bash
     sftp user1@192.168.1.100
     ```

  2. Upload file:

     ```bash
     sftp> put report.pdf /home/user1
     ```

  3. Download directory:

     ```bash
     sftp> get -r /remote/project
     ```

  4. Corner Case: Connection refused:

     ```bash
     sftp user1@server
     # Error: Connection refused
     # Fix: Check sshd: systemctl status sshd; firewall-cmd --list-all
     ```

### Synchronizing Files with `rsync`

- **Purpose**: Efficiently sync files, copying only differences.
- **Options**:
  - `-a`: Archive mode (preserves permissions, timestamps).
  - `-v`: Verbose.
  - `-z`: Compress during transfer.
  - `--delete`: Remove destination files not in source.
  - `--exclude`: Skip files/patterns.
- **Examples**:
  1. Sync directory to remote:

     ```bash
     rsync -av /local/data user1@192.168.1.100:/remote/data
     ```

  2. Sync with compression:

     ```bash
     rsync -avz /local/backup user1@remote:/backup
     ```

  3. Delete unmatched files:

     ```bash
     rsync -av --delete /local/project user1@remote:/project
     ```

  4. Corner Case: `rsync` hangs:

     ```bash
     rsync -avz /large user1@remote:/dest
     # Fix: Use --progress or check SSH: ssh user1@remote
     ```

### Summary of `tar`, `scp`, `sftp`, and `rsync` Commands

- **tar**:
  - Create: `tar -cvf archive.tar files`.
  - Compress: `tar -czvf/-cjvf/-cJvf` (gzip/bzip2/xz).
  - List: `tar -tvf`.
  - Extract: `tar -xvf` (with `-z/-j/-J`).
- **scp**:
  - Copy to: `scp file user@remote:/dest`.
  - Copy from: `scp user@remote:/file .`.
  - Recursive: `-r`.
- **sftp**:
  - Connect: `sftp user@remote`.
  - Commands: `put`, `get`, `ls`, `cd`.
- **rsync**:
  - Sync: `rsync -av source user@remote:/dest`.
  - Compress: `-z`.
  - Delete: `--delete`.
  - Exclude: `--exclude`.

### Practical Examples

1. **Backup and Transfer Home Directory**:

   ```bash
   tar -czvf /backup/home_$(date +%F).tar.gz /home/user1
   scp /backup/home_$(date +%F).tar.gz user1@192.168.1.100:/backup
   ssh user1@192.168.1.100 ls /backup
   ```

2. **Restore Archive**:

   ```bash
   mkdir /restore
   tar -xzvf /backup/home_2025-08-14.tar.gz -C /restore
   ls /restore
   ```

3. **Sync Project with `rsync`**:

   ```bash
   rsync -avz --exclude '*.log' /project user1@192.168.1.100:/project
   ```

4. **Interactive `sftp` Transfer**:

   ```bash
   sftp user1@192.168.1.100
   sftp> put config.tar.gz
   sftp> get /remote/data.txt
   sftp> exit
   ```

5. **Compress and Archive Logs**:

   ```bash
   tar -cJvf /backup/logs_$(date +%F).tar.xz /var/log
   rsync -avz /backup/logs_$(date +%F).tar.xz user1@remote:/backup
   ```

### Common Pitfalls

- **Missing Compression Flags**: `tar -cvf` creates uncompressed archives, wasting space.
- **Overwriting Files**: `tar -xvf` or `scp` overwrites without warning:

  ```bash
  ls /destination  # Check before extracting
  ```

- **Permission Issues**: System directories require `sudo`:

  ```bash
  sudo tar -czvf /backup/etc.tar.gz /etc
  ```

- **SSH Issues**: `scp`/`sftp`/`rsync` fail if `sshd` is down or firewall blocks port 22:

  ```bash
  systemctl status sshd
  sudo firewall-cmd --list-all
  ```

- **Large `scp` Transfers**: Slow for large files; use `rsync -z` instead.
- **Incorrect `rsync` Options**: Omitting `--delete` leaves outdated files:

  ```bash
  rsync -av --dry-run --delete /source user@remote:/dest  # Test first
  ```

### Revision

- **Questions**:
  1. What creates a `bzip2` archive? (`tar -cjvf archive.tar.bz2 files`)
  2. How do you download a file with `sftp`? (`sftp> get file`)
  3. What does `rsync --exclude` do? (Skips specified files/patterns.)
- **Exercises**:
  1. Archive `/etc` and transfer to remote:

     ```bash
     sudo tar -czvf /backup/etc_$(date +%F).tar.gz /etc
     scp /backup/etc_$(date +%F).tar.gz user1@192.168.1.100:/backup
     ```

  2. Sync `/data` with `rsync`:

     ```bash
     rsync -avz --exclude '*.tmp' /data user1@192.168.1.100:/data
     ```

  3. Extract `xz` archive:
  
     ```bash
     tar -xJvf backup.tar.xz -C /restore
     ```
