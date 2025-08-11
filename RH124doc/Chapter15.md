# RH124 Revision Notebook: Chapter 15 - Access Linux File Systems

This notebook provides a detailed summary of Chapter 15 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on accessing and managing Linux file systems, including mounting, unmounting, and understanding file system types, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 15: Access Linux File Systems

### Key Concepts

- **File Systems in Linux**:
  - A file system organizes and stores data on storage devices (e.g., disks, SSDs).
  - Common types in RHEL: **XFS** (default), **ext4** (legacy), **vfat** (USB drives), **nfs** (network), **tmpfs** (memory-based).
  - Each file system is mounted to a directory (e.g., `/mnt`) to make it accessible.
- **Mounting**:
  - **Mount**: Attaches a file system to a directory in the file system hierarchy.
  - **Unmount**: Detaches a file system, making it inaccessible.
  - Mount points are typically `/mnt`, `/media`, or custom directories.
- **Key Files and Directories**:
  - `/etc/fstab`: Defines permanent mount points and file system options.
    - Format: `device mount_point filesystem_type options dump pass`.
  - `/proc/mounts` or `/etc/mtab`: Lists currently mounted file systems.
  - `/dev/`: Contains device files (e.g., `/dev/sda1` for a disk partition).
- **Device Identification**:
  - Devices are identified by name (e.g., `/dev/sda1`), UUID, or label.
  - UUIDs are unique identifiers for file systems, preferred for consistency.
- **Tools for File System Management**:
  - `mount`, `umount`: Manual mounting and unmounting.
  - `lsblk`, `blkid`: List block devices and their details.
  - `df`, `du`: Monitor disk usage.
- **RHEL 9/10 Notes**:
  - XFS is optimized for large files and scalability in RHEL 9/10.
  - RHEL 10 enhances `systemd` integration for automounting and Lightspeed support for file system tasks (e.g., `rhel lightspeed "mount a USB drive"` suggests `mount` commands).

### Important Commands

- **Listing File Systems and Devices**:

  ```bash
  lsblk  # List block devices and mount points
  blkid  # Show device UUIDs and file system types
  df -h  # Show disk usage in human-readable format
  du -sh /path  # Show directory size
  cat /proc/mounts  # List active mounts
  ```

- **Mounting File Systems**:

  ```bash
  mount /dev/sdb1 /mnt  # Mount device to /mnt
  mount -t ext4 /dev/sdb1 /mnt  # Specify file system type
  mount -o ro /dev/sdb1 /mnt  # Mount read-only
  mount  # List all mounted file systems
  ```

- **Unmounting File Systems**:

  ```bash
  umount /mnt  # Unmount file system
  umount /dev/sdb1  # Unmount by device
  ```

- **Managing `/etc/fstab`**:

  ```bash
  cat /etc/fstab  # View mount configurations
  sudo vim /etc/fstab  # Edit fstab (e.g., add UUID-based mount)
  ```

- **Finding Device Information**:

  ```bash
  blkid /dev/sdb1  # Get UUID and file system type
  lsblk -f  # Show file system details with UUIDs
  ```

- **Checking Disk Usage**:

  ```bash
  df -h /  # Check free space on root
  du -sh /home/user1  # Check size of user1’s home
  ```

### Practical Examples

- **Scenario: Mount a USB Drive**:
  - Identify USB device:

    ```bash
    lsblk
    ```

    Output: Shows `/dev/sdb1` (e.g., vfat file system).
  - Mount:

    ```bash
    sudo mkdir /mnt/usb
    sudo mount /dev/sdb1 /mnt/usb
    ls /mnt/usb  # Verify contents
    ```

  - Unmount: `sudo umount /mnt/usb`.
- **Scenario: Add Permanent Mount in `/etc/fstab`**:
  - Get UUID:

    ```bash
    blkid /dev/sdb1
    ```

    Output: `UUID="1234-ABCD" TYPE="ext4"`.
  - Edit `/etc/fstab`:

    ```bash
    sudo vim /etc/fstab
    # Add: UUID=1234-ABCD /mnt/data ext4 defaults 0 0
    ```

  - Test: `sudo mount -a` (checks `fstab` for errors).
- **Scenario: Check Disk Usage**:
  - Monitor root file system:

    ```bash
    df -h /
    ```

    Output: Shows used/free space (e.g., `20G used, 50G free`).
  - Find large directories:

    ```bash
    du -sh /var/* | sort -hr | head
    ```

- **Daily Use: Automount a Network File System**:
  - Mount an NFS share:

    ```bash
    sudo mkdir /mnt/nfs
    sudo mount -t nfs 192.168.1.100:/share /mnt/nfs
    ```

  - Add to `/etc/fstab`:

    ```bash
    192.168.1.100:/share /mnt/nfs nfs defaults 0 0
    ```

- **Scenario: Troubleshoot Mount Failure**:
  - Mount fails? Check device:

    ```bash
    lsblk
    blkid /dev/sdb1
    ```

  - Check logs: `journalctl -xe | grep mount`.
  - Verify `fstab`: `sudo mount -a`.

### Common Pitfalls

- **Mounting Without Permissions**: Mounting system directories requires `sudo`.
- **Incorrect `fstab` Syntax**: Errors in `/etc/fstab` can prevent booting. Always test with `mount -a` before rebooting.
- **Device Not Found**: Ensure device exists (`lsblk`) and file system is supported (`modprobe vfat` for FAT32).
- **Busy File System**: `umount` fails if files are in use. Check with `lsof /mnt` and close processes (`fuser -k /mnt`).
- **UUID Mismatches**: If disks change, UUIDs in `fstab` may be invalid. Update with `blkid`.
- **Full File System**: Running out of space causes errors. Monitor with `df -h`.

### Best Practices and Tips

- **Use UUIDs in `fstab`**: Prevents issues with changing device names (e.g., `/dev/sdb` to `/dev/sdc`).
- **Test `fstab` Changes**: Run `sudo mount -a` to validate before rebooting.
- **Mount Temporarily First**: Test mounts manually (`mount /dev/sdb1 /mnt`) before adding to `fstab`.
- **Monitor Disk Usage**: Schedule checks:

  ```bash
  df -h > /var/log/disk_usage_$(date +%F).log
  ```

- **RHEL 10 Tip**: Use Lightspeed for file system tasks (e.g., `rhel lightspeed "mount ext4 drive"` suggests `mount -t ext4`).
- **Backup `fstab`**: Before editing:

  ```bash
  sudo cp /etc/fstab /etc/fstab.bak
  ```

- **Clean Up Mount Points**: Remove unused mount directories (`rmdir /mnt/usb`) after unmounting.

### Revision Quiz/Notes

- **Questions**:
  - What command lists all mounted file systems? (`mount` or `cat /proc/mounts`)
  - How do you find a device’s UUID? (`blkid /dev/sdb1`)
  - What does `df -h` show? (Disk usage in human-readable format.)
- **Quick Exercise**:
  - Mount `/dev/sdc1` to `/mnt/data`, check contents, and unmount:

    ```bash
    sudo mkdir /mnt/data
    sudo mount /dev/sdc1 /mnt/data
    ls /mnt/data
    sudo umount /mnt/data
    ```

- **Self-Test**:
  - Explain the `/etc/fstab` fields (device, mount point, file system, options, dump, pass).
  - Why use UUIDs over device names in `fstab`? (Device names can change; UUIDs are unique.)

---
