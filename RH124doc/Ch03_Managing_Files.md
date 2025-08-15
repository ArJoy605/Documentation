# Chapter 3: Manage Files from the Command Line

## The File System Hierarchy Standard

The Filesystem Hierarchy Standard (FHS) defines the directory structure and contents in Linux distributions like RHEL, ensuring consistency and predictability. The root directory (`/`) is the top-level, with subdirectories organized for specific purposes, separating shareable vs. non-shareable and static vs. variable data. Below is a simplified tree diagram of key RHEL directories:

```plaintext
/ (root)
├── bin/        # Essential user binaries (ls, cp)
├── boot/       # Boot loader and kernel files
├── dev/        # Device files (sda, null)
├── etc/        # Configuration files (passwd, fstab)
├── home/       # User home directories
│   └── user1/  # Example user directory
│   └── user2/  # Another user directory
│   └── user3/  # Yet another user directory
├── lib/        # Shared libraries
├── lib64/      # 64-bit libraries
├── media/      # Mount points for removable media
├── mnt/        # Temporary mount points
├── opt/        # Optional third-party software
├── proc/       # Virtual process/kernel info
├── root/       # Root user’s home
├── run/        # Runtime data (PIDs, sockets)
├── sbin/       # System binaries (fdisk, reboot)
├── sys/        # Virtual hardware/kernel settings
├── tmp/        # Temporary files
├── usr/        # User programs and data
│   ├── bin/    # Non-essential binaries
│   └── share/  # Documentation, man pages
│   └── sbin/   # Non-essential system binaries
│   └── tmp/    # Temporary files for users
└── var/        # Variable data
    ├── log/    # Log files (messages)
    └── tmp/    # Persistent temporary files  
```

## Important Red Hat Directories Table

| Directory | Description |
|-----------|-------------|
| `/` | Root directory, the foundational entry point of the file system hierarchy, encompassing all directories and files. Essential for system operation; direct modifications can destabilize the system and should be avoided. |
| `/bin` | Contains essential user command binaries (e.g., `ls`, `cp`, `cat`) required for basic operations, accessible in single-user mode or during boot. Available to all users; symlinked to `/usr/bin` in modern RHEL for streamlined maintenance. |
| `/sbin` | Houses system administration binaries (e.g., `fdisk`, `reboot`, `ifconfig`) critical for system maintenance, typically restricted to root or privileged users. Symlinked to `/usr/sbin` for consistency. |
| `/boot` | Stores boot loader configuration files (e.g., GRUB), kernel images (e.g., `vmlinuz`), and initramfs files necessary for system startup. Often mounted on a separate partition to protect boot data. |
| `/dev` | Contains device files (e.g., `/dev/sda` for disk drives, `/dev/null` for data disposal) representing hardware and virtual devices. Dynamically managed by the udev system; critical for hardware interaction. |
| `/etc` | Holds system-wide configuration files (e.g., `/etc/passwd` for user accounts, `/etc/fstab` for filesystem mounts, `/etc/ssh/sshd_config` for SSH). Requires careful editing; backups are highly recommended to prevent configuration errors. |
| `/home` | Contains user home directories (e.g., `/home/user1`) where personal files, configurations (e.g., `.bashrc`), and data reside. Features default permissions (e.g., 755) to restrict access to individual owners. |
| `/lib`, `/lib64` | Stores shared libraries and kernel modules (e.g., `.so` files) supporting `/bin` and `/sbin` programs. `/lib64` is dedicated to 64-bit libraries; essential for runtime functionality and system stability. |
| `/media`, `/mnt` | `/media` automounts removable media (e.g., USB drives) with user-friendly access; `/mnt` serves as a mount point for temporary filesystem mounts. Both are typically empty by default and configurable. |
| `/opt` | Designated for installing optional third-party software (e.g., `/opt/app`) to keep it separate from system files. Commonly used for proprietary or large applications; managed independently. |
| `/proc` | A virtual filesystem providing real-time system and process information (e.g., `/proc/cpuinfo` for CPU details, `/proc/meminfo` for memory stats, `/proc/1234` for process data). Exists in memory, not on physical storage. |
| `/root` | The home directory for the root user, isolated from `/home` for security and operational independence. Contains root’s personal files and configurations (e.g., `.bashrc`). |
| `/run` | Stores runtime data (e.g., process IDs, sockets, temporary locks) generated after boot. Cleared on reboot; heavily utilized by systemd and other runtime services. |
| `/sys` | A virtual filesystem for accessing and modifying hardware and kernel parameters (e.g., `/sys/class` for device classes). Used for dynamic system tuning and monitoring. |
| `/tmp` | Holds temporary files accessible to all users, often cleared on reboot to free space. Wide permissions (e.g., 1777) allow access but increase security risks; use cautiously. |
| `/usr` | Contains user-related programs, libraries, and documentation (e.g., `/usr/bin` for additional binaries, `/usr/share` for man pages and locale data). Designed as read-only in production to ensure stability. |
| `/var` | Stores variable data that changes during operation, including logs (`/var/log/messages`), spool files (e.g., `/var/spool/mail`), databases, and websites (`/var/www`). Requires regular monitoring to manage disk space. |

## Specifying Files By Names

### Absolute and Relative Paths

- **Absolute Paths**: Start from the root (`/`) for precise location (e.g., `/etc/passwd`). Ideal for scripts or unambiguous references.
- **Relative Paths**: Relative to the current directory (e.g., `passwd` if already in `/etc`). Shorter but context-dependent.
- **Special Paths**:
  - `.` : Current directory.
  - `..` : Parent directory.
  - `~` : User's home directory (e.g., `~/docs` expands to `/home/user/docs`).
  - `~-` : Previous working directory.

### Navigating Paths

- Check current directory: `pwd` (e.g., `/home/user`).
- Change directories with `cd`:

  ```bash
  cd /var/log  # Absolute path
  cd log       # Relative path (if in /var)
  cd ..        # Parent directory
  cd ~         # Home directory
  cd -         # Previous directory
  ```

- Use wildcards or expansions for efficient navigation (e.g., `cd /var/log/*access*` to match a subdirectory).

## Managing Files Using Command Line Tools

### Command Line File Management

Core commands for creating, copying, moving, deleting, and inspecting files/directories. Preview actions with `ls` or `echo` to avoid errors.

- **Listing Files**:

  ```bash
  ls              # List files
  ls -l           # Detailed (permissions, owner, size, date)
  ls -a           # Include hidden files (e.g., .bashrc)
  ls -lh          # Human-readable sizes (e.g., 4K)
  ls -R           # Recursive (subdirectories)
  ls *.txt        # Match patterns
  stat file.txt   # Detailed file info with creation date (size, permissions, timestamps)
  ```

- **Creating Files**:

  ```bash
  touch file.txt          # Create empty file or update timestamp
  echo "Text" > file.txt  # Create/overwrite with content
  echo "More" >> file.txt # Append content
  ```

- **Creating Directories**:

  ```bash
  mkdir dir               # Create directory
  mkdir -p parent/child   # Create nested directories
  ```

- **Copying Files/Directories**:

  ```bash
  cp file.txt copy.txt      # Copy file
  cp -r dir dir_copy        # Copy directory recursively
  cp -i *.txt /backup       # Prompt before overwrite
  cp -v file.txt /backup/   # Verbose output
  ```

- **Moving/Renaming Files/Directories**:

  ```bash
  mv file.txt new.txt       # Rename
  mv file.txt /new/dir      # Move
  mv -i dir /new/location   # Prompt before overwrite
  ```

- **Deleting Files/Directories**:

  ```bash
  rm file.txt            # Delete file
  rm -r dir              # Delete directory recursively
  rm -i *.txt            # Prompt before deleting
  rm -f file.txt         # Force delete (no prompt)
  rm -rf dir             # Force delete directory and contents
  rmdir empty_dir        # Remove empty directory
  ```

- **Viewing File Types**:

  ```bash
  file file.txt          # Identify type (e.g., ASCII text)
  ```

- **Finding Files** (Basic):

  ```bash
  find /home -name "*.txt"  # Find by name
  find / -type d -name "log"  # Find directories
  ```

## Making Links Between Files

### Hard Links and Soft Links

Links create multiple references to files or directories, used for redundancy or convenience.

| Aspect | Hard Link | Soft Link (Symbolic Link) |
|--------|-----------|---------------------------|
| Definition | Points to file's inode (data). | Points to file's path/name. |
| Creation | `ln original hardlink` | `ln -s original softlink` |
| Inode | Same as original. | Separate inode. |
| Cross Filesystems | No, same filesystem only. | Yes, across filesystems. |
| To Directories | Generally no (root/special permissions). | Yes. |
| If Original Deleted | Data remains via link. | Link breaks (dangling). |
| Usage | Data redundancy, same filesystem. | Shortcuts, flexible references. |

### Creating Hard Links

```bash
ln /path/to/file.txt hardlink.txt
```

- Check: `ls -li` shows identical inode numbers.
- Data only be deleted when all hard links are removed.

### Creating Soft Links

```bash
ln -s /path/to/file.txt softlink.txt
```

- Check: `ls -l` shows `softlink.txt -> file.txt`.

### Limitations of Hard and Soft Links

- **Hard Links**: Cannot cross filesystems; restricted to regular files; changes affect all links.
- **Soft Links**: Break if target is moved/deleted; risk of loops; slower due to path resolution.

### Best Practices for Using Links

- Use hard links for backups on same filesystem.
- Use soft links for service configs (e.g., link to active version).
- Verify with `ls -l` or `readlink softlink`.
- Avoid linking critical files without backups.
- Check for dangling links: `find -L /path -type l`.

## Matching File Names with Shell Expansion

### Command Line Expansion

The shell expands patterns before command execution, in order: brace → tilde → variables → command substitution → globbing.

### Pattern Matching

| Pattern | Description | Example |
|---------|-------------|---------|
| `*` | Any characters (0 or more) | `ls *.txt` (all .txt files) |
| `?` | Single admin | `ls file?.txt` (file1.txt, fileA.txt) |
| `[abc]` | One from set | `ls [123].txt` (1.txt, 2.txt, 3.txt) |
| `[!abc]` | Not in set | `ls [!1].txt` (excludes 1.txt) |
| `[a-z]` | Range | `ls [a-c].txt` (a.txt, b.txt, c.txt) |
| `[:class:]` | Character class | `ls [[:digit:]].txt` (0.txt to 9.txt) |

Extended globbing (enable: `shopt -s extglob`): `?(pat)` (0/1), `*(pat)` (0+), `+(pat)` (1+), `@(pat)` (1), `!(pat)` (not).

### Using Wildcards in Commands

```bash
rm *.bak  # Delete .bak files
cp file[1-3].txt /backup  # Copy specific files
```

### Using Tilde Expansion

```bash
cd ~         # Home directory
ls ~user/docs  # Another user's docs
```

### Using Brace Expansion

```bash
touch file{1..3}.txt  # file1.txt file2.txt file3.txt
mkdir {a,b,c}/sub     # a/sub b/sub c/sub
touch file{A,B}{1,2}.txt  # fileA1.txt fileA2.txt fileB1.txt fileB2.txt
```

### Using Command Substitution

```bash
echo "Today is $(date +%Y-%m-%d)"  # Outputs current date
echo "Files: $(ls *.txt)"  # Lists all .txt files
```

## Practical Examples

- **Scenario: Backup Configs**:

  ```bash
  mkdir ~/backup
  cp /etc/*.conf ~/backup
  ln -s ~/backup /etc_backup
  ```

- **Scenario: Batch Create**:

  ```bash
  touch log{2025-01..2025-12}.txt
  ```
  
- **Scenario: Clean Temp**:

  ```bash
  rm -r /tmp/*
  ```

- **Daily Use: Rename Batch**:

  ```bash
  for f in *.txt; do mv "$f" "${f%.txt}.bak"; done
  ```

## Common Pitfalls

- Case sensitivity: `File.txt` ≠ `file.txt`.
- Overwriting: Use `-i` with `cp/mv`.
- Recursive `rm`: Preview with `ls -R`.
- Wildcard mismatches: Test with `echo *`.
- Permissions: Use `sudo` for system files.
