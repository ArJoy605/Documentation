# RH124 Revision Notebook: Chapter 15 - Access Linux File Systems

This notebook summarizes Chapter 15 of the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on identifying, mounting, unmounting, and managing Linux file systems, as well as searching for files using `find` and `locate`, with detailed explanations, commands, extensive practical examples, corner cases, common pitfalls, best practices, and revision exercises.

## Chapter 15: Accessing Linux File Systems

### Key Concepts

- **File Systems**:
  - Organize and store data on storage devices (e.g., disks, SSDs).
  - Common types in RHEL: **XFS** (default, scalable), **ext4** (legacy), **vfat** (USB drives), **nfs** (network), **tmpfs** (memory-based).
- **Mounting**:
  - Attaches a file system to a directory (mount point, e.g., `/mnt`, `/media`) for access.
  - Unmounting detaches it, making it inaccessible.
- **Storage Management**:
  - Devices: Represented as block devices (e.g., `/dev/sda1`).
  - Partitions: Subdivisions of disks (e.g., `/dev/sda1`, `/dev/sda2`).
  - Logical Volumes: Managed by LVM for flexible storage.
- **Key Files**:
  - `/etc/fstab`: Defines permanent mounts (format: `device mount_point filesystem_type options dump pass`).
  - `/proc/mounts`: Lists active mounts.
  - `/dev/`: Contains device files.
- **File Searching**:
  - `find`: Searches file system in real-time based on criteria (e.g., name, size, permissions).
  - `locate`: Uses a prebuilt database for faster searches.
- **RHEL 9/10 Notes**:
  - XFS is optimized for large files and high performance.
  - RHEL 10 enhances `systemd` automounting and Lightspeed support (e.g., `rhel lightspeed "mount USB drive"` suggests `mount /dev/sdb1 /mnt`).

### Identifying File Systems and Devices

- **Purpose**: Determine storage devices, file system types, and their properties.
- **Commands**:
  - `lsblk`: List block devices and mount points.
  - `blkid`: Show UUIDs and file system types.
  - `fdisk -l` (requires `sudo`): List disk partitions.
- **Examples**:
  1. List devices:

     ```bash
     lsblk
     ```

     Output: Shows `/dev/sda1` (root, XFS), `/dev/sdb1` (vfat), etc.
  2. Get UUIDs:

     ```bash
     blkid
     ```

     Output: `UUID="1234-ABCD" TYPE="ext4"`.
  3. Corner Case: Device not listed:

     ```bash
     lsblk
     # No USB device
     # Fix: Rescan: sudo echo "- - -" > /sys/class/scsi_host/hostX/scan
     ```

### Storage Management Concepts

- **Block Devices**: Represent storage (e.g., `/dev/sda`, `/dev/nvme0n1`).
- **Disk Partitions**: Subdivide disks (e.g., `/dev/sda1` for boot, `/dev/sda2` for root).
- **Logical Volumes**: Managed by Logical Volume Manager (LVM) for dynamic resizing.
- **File Systems**: Structure data (e.g., XFS, ext4).
- **Example**:
  - Check LVM setup:

    ```bash
    lvs  # List logical volumes
    vgs  # List volume groups
    pvs  # List physical volumes
    ```

### File Systems and Mount Points

- **Mount Points**: Directories where file systems are attached (e.g., `/mnt`, `/media/username`).
- **Common File Systems**:
  - XFS: Default, high-performance.
  - ext4: Reliable, widely used.
  - vfat: Compatible with Windows (USB drives).
  - nfs: Network file system.
- **Example**:
  - List mounts:

    ```bash
    mount | grep /dev
    ```

### File Systems, Storage, Block Devices

- **Block Devices**: Physical or virtual storage (e.g., `/dev/sda`, `/dev/nvme0n1`).
- **Commands**:
  - `lsblk -f`: Show file systems and UUIDs.
  - `parted -l`: List partition details (requires `sudo`).
- **Examples**:
  1. List block devices with file systems:

     ```bash
     lsblk -f
     ```

  2. Check partition table:

     ```bash
     sudo parted -l
     ```

  3. Corner Case: Unformatted device:

     ```bash
     blkid /dev/sdb1
     # No output
     # Fix: Format: sudo mkfs.xfs /dev/sdb1
     ```

### Disk Partitions

- **Purpose**: Divide disks for separate file systems (e.g., `/boot`, `/home`).
- **Tools**: `fdisk`, `parted`, `gparted` (GUI, not in RH124).
- **Examples**:
  1. List partitions:

     ```bash
     sudo fdisk -l /dev/sda
     ```

  2. Create partition (interactive):

     ```bash
     sudo fdisk /dev/sdb
     # Commands: n (new), p (primary), w (write)
     ```

  3. Corner Case: Partition table locked:

     ```bash
     sudo fdisk /dev/sdb
     # Error: Device busy
     # Fix: Unmount: sudo umount /dev/sdb1
     ```

### Logical Volumes

- **Purpose**: LVM allows dynamic resizing and pooling of storage.
- **Components**:
  - Physical Volumes (PV): Disks/partitions (e.g., `/dev/sda1`).
  - Volume Groups (VG): Pool of PVs.
  - Logical Volumes (LV): Virtual partitions in a VG.
- **Examples**:
  1. Create logical volume:

     ```bash
     sudo pvcreate /dev/sdb1
     sudo vgcreate myvg /dev/sdb1
     sudo lvcreate -L 10G -n mylv myvg
     sudo mkfs.xfs /dev/myvg/mylv
     ```

  2. Mount LV:

     ```bash
     sudo mkdir /mnt/mylv
     sudo mount /dev/myvg/mylv /mnt/mylv
     ```

  3. Corner Case: No space in VG:

     ```bash
     lvcreate -L 20G myvg
     # Error: Insufficient space
     # Fix: Add PV: pvcreate /dev/sdb2; vgextend myvg /dev/sdb2
     ```

### Examining File Systems

- **Commands**:
  - `fsck`: Check and repair file systems.
  - `df -h`: Disk usage.
  - `du -sh`: Directory size.
- **Examples**:
  1. Check file system:

     ```bash
     sudo fsck /dev/sdb1
     ```

  2. Disk usage:

     ```bash
     df -h /mnt
     ```

  3. Corner Case: File system errors:

     ```bash
     fsck /dev/sdb1
     # Errors found
     # Fix: sudo fsck -y /dev/sdb1
     ```

### Mounting and Unmounting File Systems

- **Mounting**: Attaches file system to a directory.
- **Unmounting**: Detaches file system, ensuring no open files.
- **Examples**:
  1. Mount USB:

     ```bash
     sudo mkdir /mnt/usb
     sudo mount /dev/sdb1 /mnt/usb
     ```

  2. Unmount:

     ```bash
     sudo umount /mnt/usb
     ```

  3. Corner Case: Device busy:

     ```bash
     umount /mnt/usb
     # Error: Device busy
     # Fix: lsof /mnt/usb; sudo fuser -k /mnt/usb
     ```

### Mounting File Systems Manually

- **Command**: `mount [device] [mount_point]`.
- **Examples**:
  1. Mount ext4 partition:

     ```bash
     sudo mount -t ext4 /dev/sdc1 /mnt/data
     ```

  2. Mount read-only:

     ```bash
     sudo mount -o ro /dev/sdb1 /mnt/readonly
     ```

  3. Corner Case: Wrong file system type:

     ```bash
     mount -t ext4 /dev/sdb1 /mnt
     # Error: Wrong fs type
     # Fix: blkid /dev/sdb1; mount -t vfat
     ```

### Identifying Block Devices

- **Commands**: `lsblk`, `blkid`, `fdisk -l`.
- **Examples**:
  1. List block devices:

     ```bash
     lsblk
     ```

  2. Get UUID:

     ```bash
     blkid /dev/sdb1
     ```

  3. Corner Case: Device not detected:

     ```bash
     lsblk
     # Missing device
     # Fix: Rescan: sudo echo "- - -" > /sys/class/scsi_host/hostX/scan
     ```

### Mounting by Block Device Name

- **Syntax**: `mount /dev/<device> /mount_point`.
- **Examples**:
  1. Mount by device:

     ```bash
     sudo mount /dev/sdb1 /mnt/data
     ```

  2. Verify:

     ```bash
     mount | grep /mnt/data
     ```

  3. Corner Case: Device name changed:

     ```bash
     mount /dev/sdb1 /mnt
     # Error: No such device
     # Fix: Use UUID in fstab
     ```

### Mounting by File System UUID

- **Purpose**: UUIDs ensure consistent mounting despite device name changes.
- **Examples**:
  1. Get UUID:

     ```bash
     blkid /dev/sdb1
     ```

     Output: `UUID="1234-ABCD"`.
  2. Add to `fstab`:

     ```bash
     sudo vim /etc/fstab
     # Add: UUID=1234-ABCD /mnt/data xfs defaults 0 0
     ```

  3. Mount:

     ```bash
     sudo mount -a
     ```

  4. Corner Case: Invalid UUID:

     ```bash
     mount -a
     # Error: UUID not found
     # Fix: Verify: blkid
     ```

### Automatic Mounting of Removable Storage Devices

- **Tool**: `systemd` and `udisks2` handle automounting (e.g., USB drives in `/media/username`).
- **Examples**:
  1. Check automount:

     ```bash
     mount | grep /media
     ```

  2. Disable automount (not typical in RH124):

     ```bash
     sudo systemctl stop udisks2
     ```

  3. Corner Case: Automount fails:

     ```bash
     journalctl -u udisks2
     # Fix: Ensure udisks2: systemctl start udisks2
     ```

### Unmounting File Systems

- **Command**: `umount [device|mount_point]`.
- **Examples**:
  1. Unmount by mount point:

     ```bash
     sudo umount /mnt/data
     ```

  2. Unmount by device:

     ```bash
     sudo umount /dev/sdb1
     ```

  3. Corner Case: Files in use:

     ```bash
     umount /mnt/data
     # Error: Target busy
     # Fix: lsof /mnt/data; sudo fuser -k /mnt/data
     ```

### Locating Files on the System

- **Tools**:
  - `find`: Real-time search based on criteria.
  - `locate`: Fast, database-driven search (requires `mlocate`).
- **Setup for `locate`**:

  ```bash
  sudo dnf install mlocate
  sudo updatedb  # Update locate database
  ```

### Searching for Files with `find`

- **Syntax**: `find [path] [criteria]`.
- **Examples**:
  1. Find file by name:

     ```bash
     find /home -name "config.txt"
     ```

  2. Find directories:

     ```bash
     find /var -type d -name "log"
     ```

  3. Corner Case: Permission denied:

     ```bash
     find /root
     # Error: Permission denied
     # Fix: sudo find /root -name file
     ```

### Searching for Files with `locate`

- **Command**: `locate [pattern]`.
- **Examples**:
  1. Find file:

     ```bash
     locate config.txt
     ```

  2. Update database:

     ```bash
     sudo updatedb
     ```

  3. Corner Case: Outdated database:

     ```bash
     locate newfile.txt
     # No results
     # Fix: sudo updatedb
     ```

### Searching for Files in Real Time

- **Tool**: `find` (real-time, no database needed).
- **Examples**:
  1. Find recent files:

     ```bash
     find /home -mmin -60  # Modified in last hour
     ```

  2. Find empty files:

     ```bash
     find /tmp -empty
     ```

  3. Corner Case: Slow search:

     ```bash
     find /
     # Slow on large systems
     # Fix: Narrow path: find /home
     ```

### Searching Files Based on Ownership or Permission

- **Examples**:
  1. Find files owned by `user1`:

     ```bash
     find /home -user user1
     ```

  2. Find files with 777 permissions:

     ```bash
     find / -perm 777
     ```

  3. Corner Case: Permission errors:

     ```bash
     find / -perm 600
     # Errors: Permission denied
     # Fix: sudo find / -perm 600
     ```

### Searching Files Based on Size

- **Examples**:
  1. Find files larger than 100MB:

     ```bash
     find / -size +100M
     ```

  2. Find files smaller than 1MB:

     ```bash
     find /home -size -1M
     ```

  3. Corner Case: No results:

     ```bash
     find /tmp -size +1G
     # Fix: Verify size: find /tmp -size +100k
     ```

### Searching Files Based on File Type

- **Types**: `f` (file), `d` (directory), `l` (symlink).
- **Examples**:
  1. Find regular files:

     ```bash
     find /home -type f -name "*.txt"
     ```

  2. Find directories:

     ```bash
     find /var -type d
     ```

  3. Corner Case: Wrong type:

     ```bash
     find / -type f -name log
     # Fix: find / -type d -name log
     ```

### Searching Files Based on Modification Time and Birth Time

- **Options**:
  - `-mtime <days>`: Modified days ago.
  - `-mmin <minutes>`: Modified minutes ago.
  - `-ctime`: Creation time (not always available).
- **Examples**:
  1. Files modified in last 24 hours:

     ```bash
     find /home -mtime -1
     ```

  2. Files modified in last 30 minutes:

     ```bash
     find /tmp -mmin -30
     ```

  3. Corner Case: No creation time:

     ```bash
     find / -ctime -1
     # Fix: Use -mtime or check fs support
     ```

### Practical Examples

1. **Mount and Access USB Drive**:

   ```bash
   lsblk
   sudo mkdir /mnt/usb
   sudo mount /dev/sdb1 /mnt/usb
   ls /mnt/usb
   sudo umount /mnt/usb
   ```

2. **Add Permanent Mount**:

   ```bash
   blkid /dev/sdc1
   sudo vim /etc/fstab
   # Add: UUID=1234-ABCD /mnt/data xfs defaults 0 0
   sudo mount -a
   ```

3. **Find Large Log Files**:

   ```bash
   find /var/log -type f -size +10M
   ```

4. **Search for Config Files**:

   ```bash
   sudo updatedb
   locate httpd.conf
   sudo find /etc -name "*.conf"
   ```

5. **Set Up LVM and Mount**:

   ```bash
   sudo pvcreate /dev/sdb1
   sudo vgcreate datavg /dev/sdb1
   sudo lvcreate -L 5G -n datalv datavg
   sudo mkfs.xfs /dev/datavg/datalv
   sudo mount /dev/datavg/datalv /mnt/data
   ```

### Common Pitfalls

- **Invalid `fstab`**: Syntax errors prevent booting:

  ```bash
  sudo mount -a  # Test before reboot
  ```

- **Device Busy**: Cannot unmount if files are open:

  ```bash
  lsof /mnt/data
  sudo fuser -k /mnt/data
  ```

- **Missing File System**: Mount fails if device isn’t formatted:

  ```bash
  sudo mkfs.xfs /dev/sdb1
  ```

- **Permission Denied in `find`**: Use `sudo` for restricted paths:

  ```bash
  sudo find /root
  ```

- **Outdated `locate` Database**: Run `sudo updatedb` for recent files.
- **No Space Left**: Check with `df -h` before mounting.

### Best Practices

- **Use UUIDs in `fstab`**: Ensures consistent mounts:

  ```bash
  blkid /dev/sdb1
  ```

- **Test Mounts**: Mount manually before adding to `fstab`.

- **Backup `fstab`**:

  ```bash
  sudo cp /etc/fstab /etc/fstab.bak
  ```

- **Monitor Disk Usage**: Automate checks:

  ```bash
  df -h > /var/log/disk_$(date +%F).log
  ```

- **Update `locate` Database**: Schedule `updatedb`:

  ```bash
  sudo crond -e
  # Add: 0 0 * * * updatedb
  ```

- **RHEL 10**: Use Lightspeed (e.g., `rhel lightspeed "find large files"` suggests `find / -size +100M`).

### Revision Quiz/Exercises

- **Questions**:
  1. How do you mount by UUID? (`mount UUID=1234-ABCD /mnt`)
  2. What finds files larger than 50MB? (`find / -size +50M`)
  3. What does `/etc/fstab` do? (Defines permanent mounts.)
- **Exercises**:
  1. Mount and verify a partition:

     ```bash
     sudo mkdir /mnt/test
     sudo mount /dev/sdb1 /mnt/test
     ls /mnt/test
     ```

  2. Find recent files:

     ```bash
     find /home -mmin -60
     ```

  3. Set up permanent NFS mount:

     ```bash
     sudo vim /etc/fstab
     # Add: 192.168.1.100:/share /mnt/nfs nfs defaults 0 0
     sudo mount -a
     ```
