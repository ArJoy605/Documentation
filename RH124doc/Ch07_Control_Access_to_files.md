# Chapter 7: Control Access to Files

## Key Concepts

- **Linux File Permissions**:
  - Every file and directory has associated permissions defining who can read, write, or execute it.
  - Permissions are assigned to three categories: **owner** (user), **group**, and **others**.
  - Permissions types:
    - **Read (r)**: View file contents or list directory contents (value: 4).
    - **Write (w)**: Modify file contents or create/delete files in a directory (value: 2).
    - **Execute (x)**: Run a file as a program or enter a directory (value: 1).
- **Permission Representation**:
  - Symbolic: `rwxr-xr-x` (owner: rwx, group: r-x, others: r-x).
  - Numeric (octal): `755` (rwx = 4+2+1=7, r-x = 4+1=5, r-x = 5).
  - Example: `rw-r--r--` = `644` (owner: rw, group: r, others: r).
- **Key Files**:
  - Permissions are stored in the file system’s inode, viewable with `ls -l`.
  - Ownership is tied to User ID (UID) and Group ID (GID) in `/etc/passwd` and `/etc/group`.
- **Special Permissions**:
  - **SetUID (s)**: File executes as the owner (e.g., `passwd` runs as root). Octal: 4000.
  - **SetGID (s)**: File executes as group or inherits group for directory contents. Octal: 2000.
  - **Sticky Bit (t)**: Restricts deletion in directories to owners (e.g., `/tmp`). Octal: 1000.
- **Default Permissions**:
  - Controlled by the user’s **umask**, which subtracts permissions from default (files: 666, directories: 777).
  - Example: `umask 022` → files: 666-022=644, directories: 777-022=755.
  - View umask: `umask` (numeric) or `umask -S` (symbolic).
- **File Access Control Lists (ACLs)**:
  - Extend standard permissions for fine-grained control (e.g., specific users or groups).
  - Managed with `setfacl` and `getfacl`.

## Important Commands

- **Viewing Permissions**:

  ```bash
  ls -l file.txt  # Show permissions, owner, group (e.g., -rw-r--r-- user1 group1)
  ls -ld /mydir  # Show directory permissions
  stat file.txt  # Detailed file info, including permissions and ownership
  ```
  
- **Changing Permissions**:

  ```bash
  chmod u+rwx file.txt  # Add read, write, execute for owner
  chmod +x file.txt  # Add execute for all
  chmod g-w file.txt  # Remove write for group
  chmod o=r file.txt  # Set read-only for others
  chmod 644 file.txt  # Set permissions numerically (rw-r--r--)
  chmod -R 755 /mydir  # Apply recursively to directory
  ```

- **Changing Ownership**:

  ```bash
  chown user1 file.txt  # Change owner to user1
  chown user1:group1 file.txt  # Change owner and group
  chown -R user1:group1 /mydir  # Apply recursively
  chgrp group1 file.txt  # Change group only
  chgrp -R group1 /mydir  # Recursive group change
  ```

- **Special Permissions**:

  ```bash
  chmod u+s program  # Set SetUID (e.g., -rwsr-xr-x)
  chmod g+s /shared  # Set SetGID on directory
  chmod +t /tmp  # Set sticky bit (e.g., drwxrwxrwt)
  chmod 4755 program  # SetUID + rwx for owner, rx for others
  ```

- **Managing umask**:

  ```bash
  umask  # Display current umask (e.g., 0022)
  umask 027  # Set umask (e.g., files: 640, directories: 750)
  umask -S  # Show symbolic umask (e.g., u=rwx,g=rx,o=)
  ```

- **File ACLs**:

  ```bash
  setfacl -m u:user2:rw file.txt  # Grant user2 read/write access
  setfacl -m g:group2:rx /mydir  # Grant group2 read/execute
  getfacl file.txt  # View ACLs
  setfacl -x u:user2 file.txt  # Remove user2’s ACL
  setfacl -b file.txt  # Remove all ACLs
  ```

## Practical Examples

- **Scenario: Secure a Configuration File**:
  - Restrict `/etc/myconfig` to owner-only access:

    ```bash
    sudo chown root:root /etc/myconfig
    sudo chmod 600 /etc/myconfig
    ls -l /etc/myconfig  # Verify: -rw------- root root
    ```

- **Scenario: Share a Directory**:
  - Create a shared directory for `developers` group:

    ```bash
    sudo mkdir /shared
    sudo groupadd developers
    sudo chown :developers /shared
    sudo chmod 770 /shared
    sudo chmod g+s /shared  # New files inherit group
    ls -ld /shared  # Verify: drwxrws--- root developers
    ```

- **Scenario: Set Up a Public Directory**:
  - Allow all users to read files in `/public`:

    ```bash
    sudo mkdir /public
    sudo chmod 755 /public
    ls -ld /public  # Verify: drwxr-xr-x
    ```

- **Scenario: Use ACLs for Specific Access**:
  - Grant `user2` write access to `project.txt` without changing standard permissions:

    ```bash
    setfacl -m u:user2:rw project.txt
    getfacl project.txt  # Verify ACL entry
    ```

- **Daily Use: Check and Fix Permissions**:

  - Find files with incorrect permissions:

    ```bash
    find /home -type f -perm /007  # Files writable by others
    chmod -R o-w /home/user1  # Remove others’ write access
    ```

- **Scenario: SetUID for a Script**:
  - Allow a script to run as owner:

    ```bash
    sudo chmod u+s /usr/local/bin/myscript
    ls -l /usr/local/bin/myscript  # Verify: -rwsr-xr-x
    ```

## Common Pitfalls

- **Incorrect Permissions**: Setting `777` (world-writable) is insecure; use least privilege (e.g., `644` for files, `755` for directories).
- **Forgetting `-R`**: Without `-R`, `chmod` or `chown` only affects the target, not directory contents.
- **Misusing SetUID/SetGID**: Applying to untrusted scripts can lead to security risks (e.g., privilege escalation).
- **Ignoring umask**: Default umask (e.g., `022`) may create overly permissive files. Set stricter umask (e.g., `027`) in `~/.bashrc`.
- **ACL Conflicts**: ACLs override standard permissions but can be confusing if not documented. Use `getfacl` to verify.
- **Permission Denied Errors**: Operations in system directories (e.g., `/etc`) require `sudo`. Check ownership with `ls -l`.

## Best Practices and Tips

- **Follow Least Privilege**: Grant only necessary permissions (e.g., `640` for sensitive files).
- **Use Groups for Collaboration**: Assign shared directories to a group with `g+rw` and SetGID (`g+s`).
- **Backup Before Changes**: Copy critical files (e.g., `sudo cp /etc/myconfig /etc/myconfig.bak`) before modifying permissions.
- **Verify Changes**: Always use `ls -l` or `getfacl` after changing permissions or ownership.
- **Document ACLs**: Maintain a log of ACL changes (e.g., `getfacl /shared > acl_backup.txt`).

---
