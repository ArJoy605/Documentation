# RH124 Revision Notebook: Chapter 6 - Manage Local Users and Groups

This notebook provides a detailed summary of Chapter 6 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on managing local users and groups on a RHEL system, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 6: Manage Local Users and Groups

### Key Concepts

- **Users in Linux**:
  - Every process and file is associated with a user, identified by a User ID (UID).
  - Types: Regular users (e.g., employees), system users (e.g., `daemon` for services), and the root user (UID 0, full administrative privileges).
  - User information is stored in `/etc/passwd` (user details) and `/etc/shadow` (encrypted passwords).
- **Groups in Linux**:
  - Groups simplify permission management by assigning permissions to multiple users at once.
  - Identified by a Group ID (GID), stored in `/etc/group` and `/etc/gshadow`.
  - Types: Primary group (default for new files) and secondary groups (additional memberships).
- **Key Files**:
  - `/etc/passwd`: Stores user info (username, UID, GID, home directory, shell).
    - Format: `username:x:UID:GID:GECOS:home_dir:shell` (e.g., `user1:x:1001:1001:User One:/home/user1:/bin/bash`).
  - `/etc/shadow`: Stores encrypted passwords and account policies (e.g., expiration).
    - Format: `username:encrypted_password:last_change:min_days:max_days:warn:inactive:expire`.
  - `/etc/group`: Lists groups and their members.
    - Format: `group_name:x:GID:member1,member2`.
  - `/etc/gshadow`: Stores group passwords and admins (rarely used).
- **User and Group Management**:
  - Create, modify, and delete users and groups using command-line tools.
  - Assign users to groups for access control (e.g., shared directories).
- **Password Management**:
  - Passwords are encrypted using algorithms like SHA-512 (default in RHEL).
  - Policies include minimum password age, expiration, and complexity.
- **RHEL 9/10 Notes**:
  - Improved `useradd` defaults for secure configurations (e.g., stronger password hashing).
  - RHEL 10 integrates Lightspeed for user management queries (e.g., `rhel lightspeed "create a new user"`).
  - Support for centralized identity management (e.g., SSSD) is introduced but not covered in RH124.

### Important Commands

- **User Management**:

  ``` bash
  useradd user1  # Create user with default settings
  useradd -m -c "John Doe" -s /bin/bash user2  # Create user with home dir, comment, and shell
  usermod -aG group1 user1  # Add user to secondary group
  usermod -c "Jane Doe" user1  # Change user comment
  usermod -L user1  # Lock user account
  usermod -U user1  # Unlock user account
  userdel user1  # Delete user (without home dir)
  userdel -r user1  # Delete user and home directory
  id user1  # Display user’s UID, GID, and group memberships
  ```

- **Group Management**:

  ```bash
  groupadd group1  # Create group
  groupadd -g 2000 group2  # Create group with specific GID
  groupmod -n newgroup group1  # Rename group
  groupdel group1  # Delete group
  getent group group1  # Display group details
  ```

- **Password Management**:

  ```bash
  passwd user1  # Set or change user’s password
  chage -l user1  # List password aging info
  chage -M 90 user1  # Set password expiration to 90 days
  chage -m 7 user1  # Set minimum days between password changes
  ```

- **Viewing User/Group Info**:

  ```bash
  cat /etc/passwd  # View all users
  cat /etc/shadow  # View password info (requires root)
  cat /etc/group  # View all groups
  getent passwd user1  # View specific user info
  ```

- **Switching Users**:

  ```bash
  su - user1  # Switch to user1 with full environment
  sudo -u user1 command  # Run command as user1
  sudo -i  # Switch to root with full environment
  ```

### Practical Examples

- **Scenario: Create a New User**:

  - Create user `jdoe` with home directory and comment:

    ```bash
    sudo useradd -m -c "John Doe" jdoe
    sudo passwd jdoe  # Set password
    id jdoe  # Verify UID, GID, and groups
    ```

  - Output: `uid=1001(jdoe) gid=1001(jdoe) groups=1001(jdoe)`.
- **Scenario: Add User to a Group**:
  - Create group `developers` and add `jdoe`:

    ```bash
    sudo groupadd developers
    sudo usermod -aG developers jdoe
    id jdoe  # Verify: groups=1001(jdoe),1002(developers)
    ```

- **Scenario: Set Password Policy**:
  - Require `jdoe` to change password every 60 days:
  
    ```bash
    sudo chage -M 60 jdoe
    chage -l jdoe  # Verify policy
    ```

- **Scenario: Delete a User Safely**:
  - Delete `jdoe` and their home directory:

    ```bash
    sudo userdel -r jdoe
    getent passwd jdoe  # Confirm deletion (no output)
    ```

- **Daily Use: Check Active Users**:
  - List logged-in users: `who` or `w`.
  - Verify user details: `getent passwd jdoe`.
- **Scenario: Troubleshoot Access**:
  - User can’t log in? Check `/etc/shadow` (with `sudo`):

    ```bash
    sudo grep jdoe /etc/shadow  # Locked if password field is !!
    sudo usermod -U jdoe  # Unlock if needed
    ```

### Common Pitfalls

- **Forgetting `-m` with `useradd`**: Without `-m`, no home directory is created, causing login issues.
- **Overwriting Groups**: Using `usermod -G` without `-a` replaces all secondary groups. Always use `-aG` to append.
- **Root Privileges**: Most user/group commands require `sudo`. Running without it results in "Permission denied".
- **Deleting Users with Active Processes**: `userdel` fails if user is logged in. Use `pkill -u user1` to terminate processes first.
- **Password Policy Errors**: Setting unrealistic policies (e.g., `chage -M 0`) can lock users out. Verify with `chage -l`.
- **Case Sensitivity**: Usernames and group names are case-sensitive (e.g., `User1` ≠ `user1`).

### Best Practices and Tips

- **Use Descriptive Comments**: Add meaningful GECOS fields (e.g., `useradd -c "John Doe, IT Dept" jdoe`) for clarity.
- **Secure Passwords**: Enforce strong passwords via `/etc/security/pwquality.conf` (e.g., min length 8, require special characters).
- **Backup Before Deletion**: Archive home directories before `userdel -r` (e.g., `tar czf /backup/jdoe.tar.gz /home/jdoe`).
- **Use `id` for Verification**: Always check user/group assignments after changes (e.g., `id jdoe`).
- **Automate with Scripts**: Create users in bulk with a script:

  ```bash
  for u in user1 user2; do sudo useradd -m $u; sudo passwd $u; done
  ```

- **RHEL 10 Tip**: Use Lightspeed for user tasks (e.g., `rhel lightspeed "add user with home directory"` suggests `useradd -m`).
- **Monitor `/etc/passwd` and `/etc/group`**: Use `cat` or `less` to review changes after modifications.

### Revision Quiz/Notes

- **Questions**:
  - What command adds `user1` to group `admins` without removing other groups? (`usermod -aG admins user1`)
  - What’s stored in `/etc/shadow`? (Encrypted passwords and aging policies.)
  - How do you lock a user account? (`usermod -L user1`)
- **Quick Exercise**:
  - Create user `testuser` with home directory, add to group `testers`, set password, and verify:

    ```bash
    sudo groupadd testers
    sudo useradd -m -c "Test User" -G testers testuser
    sudo passwd testuser
    id testuser
    ```

- **Self-Test**:
  - Explain the fields in `/etc/passwd` (username, x, UID, GID, comment, home, shell).
  - Why use `userdel -r`? (Removes home directory and mail spool for complete cleanup.)

---
