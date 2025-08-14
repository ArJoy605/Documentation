# RH124 Revision Notebook: Chapter 6 - Manage Local Users and Groups

This enhanced notebook provides a comprehensive and well-organized summary of Chapter 6 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It covers managing local users and groups, incorporating all requested topics, including user and group definitions, UID ranges, superuser concepts, `sudo` configuration, password policies, and more. The content is structured with clear explanations, command tables, practical examples, common pitfalls, best practices, and revision exercises to support learning and real-world application.

## Chapter 6: Manage Local Users and Groups

### Key Concepts

- **What is a User?**
  - A user is an entity (human or process) that interacts with the system, identified by a unique **User ID (UID)**.
  - Types:
    - **Regular users**: For individuals (e.g., employees, UID ≥ 1000).
    - **System users**: For services/processes (e.g., `daemon`, UID 1–999).
    - **Root user**: Administrative account with UID 0, granting full system access.
  - Use `id username` to display a user’s UID, GID, and group memberships.
    - Example: `id jdoe` outputs `uid=1001(jdoe) gid=1001(jdoe) groups=1001(jdoe),1002(developers)`.

- **What is a Group?**
  - A group is a collection of users sharing permissions, identified by a **Group ID (GID)**.
  - Stored in `/etc/group` with four colon-separated fields:
    - Format: `group_name:password:GID:members`
      - `group_name`: Name (e.g., `developers`).
      - `password`: `x` (password in `/etc/gshadow`, rarely used).
      - `GID`: Group ID (e.g., `1002`).
      - `members`: Comma-separated secondary group members (e.g., `jdoe,alice`).
    - Example: `developers:x:1002:jdoe,alice`.

- **Primary vs. Supplementary Groups**:
  - **Primary Group**: The default group for a user’s new files, specified by GID in `/etc/passwd`.
  - **Supplementary Groups**: Additional groups for shared access, listed in `/etc/group`.
  - Example: A user `jdoe` with primary group `jdoe` (GID 1001) and supplementary group `developers` (GID 1002).

- **Key Configuration Files**:
  - **`/etc/passwd`**: Stores user details in seven colon-separated fields:
    - Format: `username:password:UID:GID:GECOS:home_dir:shell`
      - `username`: Login name (e.g., `jdoe`).
      - `password`: `x` (password stored in `/etc/shadow`).
      - `UID`: User ID (e.g., `1001`).
      - `GID`: Primary group ID (e.g., `1001`).
      - `GECOS`: Comment field (e.g., “John Doe, IT Dept”).
      - `home_dir`: Home directory (e.g., `/home/jdoe`).
      - `shell`: Default shell (e.g., `/bin/bash`).
    - Example: `jdoe:x:1001:1001:John Doe:/home/jdoe:/bin/bash`.
  - **`/etc/shadow`**: Stores encrypted passwords and password aging policies.
    - Format: `username:encrypted_password:last_change:min_days:max_days:warn:inactive:expire`.
    - Example: `jdoe:$6$...hash...:19365:7:90:7::`.
  - **`/etc/group`**: Lists groups and their members.
  - **`/etc/gshadow`**: Stores group passwords and admins (rarely used).

- **UID Ranges**:
  - `0`: Root user.
  - `1–999`: System users (reserved for services).
  - `1000+`: Regular users (default starting UID in RHEL, configurable in `/etc/login.defs`).

- **Shadow Passwords and Password Policy**:
  - Passwords are encrypted (e.g., SHA-512) and stored in `/etc/shadow`.
  - Policies include:
    - `min_days`: Minimum days before password change.
    - `max_days`: Maximum password validity period.
    - `warn`: Days to warn before expiration.
    - `expire`: Account expiration date.
  - Configurable via `/etc/login.defs` and `/etc/security/pwquality.conf`.

- **Superuser and Switching Users**:
  - The **superuser** (root, UID 0) has unrestricted access to all commands and files.
  - Switching users:
    - `su user1`: Switches to `user1` without loading their environment.
    - `su - user1`: Switches to `user1` with full environment (loads `~/.bashrc`).
    - `sudo -u user1 command`: Runs a command as `user1`.
    - `sudo -i`: Switches to root with full environment (similar to `su -`).
    - **Difference**:
      - `sudo su`: Runs `su` as root, keeping current environment.
      - `sudo su -` or `sudo -i`: Loads root’s environment for a clean session.
      - `sudo -i` is preferred for consistent root environment and audit logging.

- **Configuring `sudo`**:
  - Managed via `/etc/sudoers`, edited with `visudo` for syntax validation.
  - Format: `user host=(run_as) commands`
    - Example: `jdoe ALL=(ALL) ALL` allows `jdoe` to run any command as any user on all hosts.
  - Group-based: `%wheel ALL=(ALL) ALL` grants `wheel` group members full `sudo` access.
  - Restrict commands: `jdoe ALL=(ALL) /usr/bin/systemctl` limits `jdoe` to `systemctl`.

- **Restricting Access**:
  - Lock accounts: Add `!` to password field in `/etc/shadow` (`usermod -L`).
  - Disable login: Set shell to `/sbin/nologin` (`usermod -s /sbin/nologin`).
  - Example: `sudo usermod -L -s /sbin/nologin jdoe` prevents `jdoe` from logging in.

- **RHEL 9/10 Notes**:
  - Enhanced `useradd` defaults for security (e.g., stronger hashing).
  - RHEL 10 integrates Lightspeed for user management queries (e.g., `rhel lightspeed "create a new user"`).
  - Centralized identity management (e.g., SSSD) is introduced but not covered in RH124.

### Command Reference

| Category            | Command                                    | Description                                      |
|---------------------|--------------------------------------------|--------------------------------------------------|
| **User Management** | `useradd user1`                            | Create user with default settings                |
|                     | `useradd -m -c "John Doe" -s /bin/bash user2` | Create user with home dir, comment, and shell |
|                     | `usermod -aG group1 user1`                 | Add user to secondary group                      |
|                     | `usermod -c "Jane Doe" user1`              | Change user comment                              |
|                     | `usermod -L user1`                         | Lock user account                                |
|                     | `usermod -U user1`                         | Unlock user account                              |
|                     | `userdel user1`                            | Delete user (without home dir)                   |
|                     | `userdel -r user1`                         | Delete user and home directory                   |
|                     | `id user1`                                 | Display user’s UID, GID, and groups              |
| **Group Management**| `groupadd group1`                          | Create group                                     |
|                     | `groupadd -g 2000 group2`                  | Create group with specific GID                   |
|                     | `groupmod -n newgroup group1`              | Rename group                                     |
|                     | `groupdel group1`                          | Delete group                                     |
|                     | `getent group group1`                      | Display group details                            |
| **Password Management** | `passwd user1`                         | Set or change user’s password                    |
|                     | `chage -l user1`                           | List password aging info                         |
|                     | `chage -M 90 user1`                        | Set password expiration to 90 days               |
|                     | `chage -m 7 user1`                         | Set minimum days between password changes        |
| **Viewing Info**    | `cat /etc/passwd`                          | View all users                                   |
|                     | `cat /etc/shadow`                          | View password info (requires root)               |
|                     | `cat /etc/group`                           | View all groups                                  |
|                     | `getent passwd user1`                      | View specific user info                          |
| **Switching Users** | `su - user1`                               | Switch to user1 with full environment            |
|                     | `sudo -u user1 command`                    | Run command as user1                             |
|                     | `sudo -i`                                  | Switch to root with full environment             |
| **Sudo Configuration** | `visudo`                                | Edit `/etc/sudoers` safely                       |

### `useradd` and `usermod` Options

| Option | Description                                      | Example Usage                     |
|--------|--------------------------------------------------|-----------------------------------|
| `-c`   | Set comment/GECOS field (e.g., full name)        | `useradd -c "John Doe" jdoe`     |
| `-g`   | Set primary group (GID or name)                  | `useradd -g developers jdoe`     |
| `-G`   | Add to secondary groups (comma-separated)        | `useradd -G admins,testers jdoe` |
| `-a`   | Append to groups (used with `-G` in `usermod`)   | `usermod -aG testers jdoe`       |
| `-d`   | Specify home directory path                      | `useradd -d /home/custom jdoe`   |
| `-m`   | Create home directory if it doesn’t exist        | `useradd -m jdoe`                |
| `-s`   | Set default shell                                | `useradd -s /bin/bash jdoe`      |
| `-L`   | Lock user account (adds `!` to `/etc/shadow`)    | `usermod -L jdoe`                |
| `-U`   | Unlock user account (removes `!`)                | `usermod -U jdoe`                |

### Practical Examples

1. **Create a New User**:
   - Create `jdoe` with home directory, custom shell, and comment:

     ```bash
     sudo useradd -m -c "John Doe, IT Dept" -s /bin/bash jdoe
     sudo passwd jdoe
     id jdoe  # Output: uid=1001(jdoe) gid=1001(jdoe) groups=1001(jdoe)
     ```

2. **Add User to a Group**:
   - Create `developers` group and add `jdoe`:

     ```bash
     sudo groupadd developers
     sudo usermod -aG developers jdoe
     id jdoe  # Output: groups=1001(jdoe),1002(developers)
     ```

3. **Configure Password Aging**:
   - Set `jdoe` password to expire every 60 days with a 7-day minimum:

     ```bash
     sudo chage -M 60 -m 7 jdoe
     chage -l jdoe  # Verify: Max days: 60, Min days: 7
     ```

4. **Delete a User Safely**:
   - Delete `jdoe` and their home directory:

     ```bash
     sudo userdel -r jdoe
     getent passwd jdoe  # Confirm deletion (no output)
     ```

5. **Configure `sudo` Access**:
   - Grant `jdoe` full `sudo` privileges via `wheel`:

     ```bash
     sudo usermod -aG wheel jdoe
     ```

   - Restrict `jdoe` to specific commands in `/etc/sudoers`:

     ```bash
     sudo visudo
     # Add: jdoe ALL=(ALL) /usr/bin/systemctl,/usr/bin/yum
     ```

6. **Restrict Access**:
   - Lock `jdoe`’s account and disable login:

     ```bash
     sudo usermod -L -s /sbin/nologin jdoe
     sudo grep jdoe /etc/shadow  # Password field starts with `!`
     ```

7. **Switch Users**:
   - Switch to `jdoe` with environment:

     ```bash
     su - jdoe
     ```

   - Run a command as `jdoe`:

     ```bash
     sudo -u jdoe whoami  # Output: jdoe
     ```

   - Switch to root with full environment:

     ```bash
     sudo -i
     ```

### Common Pitfalls

- **No Home Directory**: Forgetting `-m` with `useradd` prevents home directory creation, causing login issues.
- **Group Overwrite**: Using `usermod -G` without `-a` replaces all secondary groups. Always use `-aG`.
- **Permission Denied**: Commands like `useradd`, `usermod`, and `passwd` require `sudo`.
- **Active User Processes**: `userdel` fails if the user is logged in. Use `pkill -u user1` to terminate processes.
- **Password Policy Errors**: Unrealistic `chage` settings (e.g., `-M 0`) can lock users out.
- **Case Sensitivity**: Usernames and group names are case-sensitive (e.g., `Jdoe` ≠ `jdoe`).
- **Direct `/etc/sudoers` Edits**: Editing without `visudo` risks syntax errors, locking out `sudo` access.

### Best Practices and Tips

- **Descriptive GECOS**: Use `-c` for clear comments (e.g., `useradd -c "John Doe, IT Dept" jdoe`).
- **Strong Passwords**: Configure `/etc/security/pwquality.conf` (e.g., min length 8, require special characters).
- **Backup Before Deletion**: Archive home directories before `userdel -r`:

  ```bash
  sudo tar czf /backup/jdoe.tar.gz /home/jdoe
  ```

- **Verify Changes**: Use `id`, `getent passwd`, or `getent group` after modifications.
- **Automate User Creation**: Script bulk tasks:

  ```bash
  for u in user1 user2; do sudo useradd -m -s /bin/bash $u; sudo passwd $u; done
  ```

- **Sudo Best Practices**: Use group-based permissions (e.g., `%wheel`) and restrict commands for security.
- **Password Aging**: Set balanced policies (e.g., `chage -M 90 -m 7`) for security and usability.
- **RHEL 10 Tip**: Use Lightspeed for suggestions (e.g., `rhel lightspeed "add user with home directory"`).
- **Monitor Files**: Check `/etc/passwd`, `/etc/shadow`, and `/etc/group` with `cat` or `less` after changes.

### Revision Quiz/Exercises

- **Questions**:
  1. What command adds `user1` to `admins` without removing other groups? (`usermod -aG admins user1`)
  2. What are the seven fields in `/etc/passwd`? (username, password, UID, GID, GECOS, home_dir, shell)
  3. How do you lock a user account? (`usermod -L user1`)
  4. What’s the difference between `sudo su -` and `sudo -i`? (Both load root’s environment, but `sudo -i` uses `sudo` authentication and logging.)
  5. How do you set a 90-day password expiration? (`chage -M 90 user1`)
  6. What does `/etc/group` store? (Group name, password, GID, members)

- **Quick Exercise**:
  - Create user `testuser`, add to `testers` group, set password, and restrict to `systemctl` via `sudo`:

    ```bash
    sudo groupadd testers
    sudo useradd -m -c "Test User" -G testers testuser
    sudo passwd testuser
    sudo visudo
    # Add: testuser ALL=(ALL) /usr/bin/systemctl
    id testuser
    ```
