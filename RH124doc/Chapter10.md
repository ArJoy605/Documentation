# RH124 Revision Notebook: Chapter 10 - Configure and Secure SSH

This notebook summarizes Chapter 10 of the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on configuring and securing the Secure Shell (SSH) service for remote access, with detailed explanations, commands, extensive practical examples, corner cases, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 10: Configure and Secure SSH

### Key Concepts

- **What is OpenSSH?**:
  - OpenSSH is an open-source implementation of the SSH protocol for secure remote access and file transfer.
  - Provides encrypted communication over untrusted networks using ciphers (e.g., RSA, ECDSA, Ed25519 in RHEL 10).
  - Components: `sshd` (server daemon), `ssh` (client), `scp` (file copy), `sftp` (file transfer), `ssh-keygen` (key management).
- **SSH Overview**:
  - SSH enables secure remote command-line access, file transfers, and tunneling.
  - Default port: 22; configurable in `/etc/ssh/sshd_config` (server) and `/etc/ssh/ssh_config` (client).
- **Authentication Methods**:
  - **Password-based**: Username and password (vulnerable to brute-force attacks).
  - **Key-based**: Public/private key pairs (more secure, recommended for production).
  - Others: GSSAPI, Kerberos (not covered in RH124).
- **Security Features**:
  - Restrict root login, change ports, disable password authentication, or limit users.
  - Integrates with SELinux and `firewalld` for access control.
- **RHEL 9/10 Notes**:
  - RHEL 10 prefers stronger ciphers (e.g., `ed25519` keys).
  - RHEL Lightspeed suggests SSH commands (e.g., `rhel lightspeed "secure SSH"` suggests disabling root login).

### Accessing Remote Command Line with SSH

- **Purpose**: Connect to a remote system securely to execute commands.
- **Command**: `ssh [user@]host [-p port]`.
- **Examples**:
  1. Connect to a server:

     ```bash
     ssh user1@192.168.1.100
     ```

     Output: Prompts for password or uses key-based authentication.
  2. Connect on a non-standard port (e.g., 2222):

     ```bash
     ssh -p 2222 user1@server.example.com
     ```

  3. Run a single command remotely:

     ```bash
     ssh user1@server.example.com "uptime"
     ```

     Output: Shows remote system’s uptime.
  4. Corner Case: Connection timeout (e.g., wrong IP or firewall):

     ```bash
     ssh user1@192.168.1.999
     # Fix: Verify IP, check firewall: sudo firewall-cmd --list-all
     ```

### Secure Shell Examples

- **Use Cases**: Remote management, file transfers, and secure tunneling.
- **Examples**:
  1. Copy a file to a remote server:

     ```bash
     scp report.txt user1@server:/home/user1/
     ```

     Output: Transfers file securely.
  2. Interactive file transfer with `sftp`:

     ```bash
     sftp user1@server
     sftp> put data.tar.gz
     sftp> get remote_file.txt
     ```

  3. Tunnel a local port to a remote service:

     ```bash
     ssh -L 8080:localhost:80 user1@server
     ```

     Access `http://localhost:8080` locally to reach remote web server.
  4. Corner Case: `scp` fails due to permissions:

     ```bash
     scp file.txt user1@server:/root/
     # Error: Permission denied
     # Fix: Use writable path or sudo: scp file.txt user1@server:/home/user1/
     ```

### Identifying Remote Users

- **Purpose**: Determine who is logged in via SSH on a server.
- **Commands**:
  - `w`: Show logged-in users and their activities.
  - `who`: Display user sessions, including remote hosts.
  - `last`: Show recent login history.
- **Examples**:
  1. List current SSH users:

     ```bash
     w
     ```

     Output: Shows `user1` from `192.168.1.50` on `pts/0`.
  2. Check detailed login info:

     ```bash
     who
     ```

     Output: `user1 pts/0 2025-08-14 10:00 (192.168.1.50)`.
  3. View recent SSH logins:

     ```bash
     last | grep sshd
     ```

     Output: Login history with timestamps and IPs.
  4. Corner Case: User not visible in `w` (disconnected session):

     ```bash
     ps aux | grep sshd  # Check for orphaned sshd processes
     sudo pkill -u user1  # Terminate user’s sessions
     ```

### SSH Host Keys

- **Purpose**: Host keys verify the server’s identity to prevent man-in-the-middle attacks.
- **Location**: `/etc/ssh/ssh_host_*` (e.g., `ssh_host_rsa_key`, `ssh_host_ed25519_key`).
- **Types**: RSA, ECDSA, Ed25519 (RHEL 10 prefers Ed25519 for performance).
- **Client Verification**: Stored in `~/.ssh/known_hosts`.
- **Examples**:
  1. Check server host keys:

     ```bash
     ls /etc/ssh/ssh_host_*
     ```

     Output: Lists `ssh_host_ed25519_key`, `ssh_host_rsa_key`, etc.
  2. Regenerate host keys (e.g., after server cloning):

     ```bash
     sudo rm /etc/ssh/ssh_host_*
     sudo ssh-keygen -A
     sudo systemctl restart sshd
     ```

  3. Corner Case: Host key mismatch:

     ```bash
     ssh user1@server
     # Error: Host key verification failed
     # Fix: Remove old key from ~/.ssh/known_hosts
     ssh-keyscan server >> ~/.ssh/known_hosts
     ```

### SSH Known Hosts Key Management

- **Purpose**: Manage client-side `known_hosts` to trust server keys.
- **File**: `~/.ssh/known_hosts` (per user) or `/etc/ssh/ssh_known_hosts` (global).
- **Commands**:
  - `ssh-keyscan`: Fetch host keys.
  - `ssh-keygen -R`: Remove host key.
- **Examples**:
  1. Add server key to `known_hosts`:

     ```bash
     ssh-keyscan 192.168.1.100 >> ~/.ssh/known_hosts
     ```

  2. Remove outdated key:

     ```bash
     ssh-keygen -R 192.168.1.100
     ```

  3. Verify known hosts:

     ```bash
     cat ~/.ssh/known_hosts | grep 192.168.1.100
     ```

  4. Corner Case: Corrupted `known_hosts`:

     ```bash
     mv ~/.ssh/known_hosts ~/.ssh/known_hosts.bak
     ssh-keyscan server >> ~/.ssh/known_hosts
     # Test: ssh user1@server
     ```

### Configuring SSH Key-Based Authentication

- **Purpose**: Enable passwordless, secure login using public/private key pairs.
- **Process**:
  - Generate key pair on client.
  - Copy public key to server’s `~/.ssh/authorized_keys`.
  - Configure `sshd` to allow key-based authentication.

### Generating SSH Keys

- **Command**: `ssh-keygen [-t type] [-b bits]`.
- **Types**: `rsa` (4096 bits for security), `ed25519` (faster, RHEL 10 default).
- **Examples**:
  1. Generate RSA key:

     ```bash
     ssh-keygen -t rsa -b 4096
     # Accept defaults: ~/.ssh/id_rsa, id_rsa.pub
     ```

  2. Generate Ed25519 key:

     ```bash
     ssh-keygen -t ed25519
     ```

  3. Generate key with passphrase:

     ```bash
     ssh-keygen -t rsa -b 4096 -C "user1@client"
     # Enter passphrase for extra security
     ```

  4. Corner Case: Overwriting existing key:

     ```bash
     ssh-keygen -t rsa
     # Prompt: Overwrite ~/.ssh/id_rsa? (Backup first: cp ~/.ssh/id_rsa ~/.ssh/id_rsa.bak)
     ```

### Copying SSH Keys to Remote Hosts

- **Commands**:
  - `ssh-copy-id`: Copy public key to server.
  - Manual copy: Append to `~/.ssh/authorized_keys`.
- **Examples**:
  1. Copy key to server:

     ```bash
     ssh-copy-id user1@192.168.1.100
     ```

     Output: Prompts for password, then copies `id_rsa.pub`.
  2. Manual copy:

     ```bash
     cat ~/.ssh/id_rsa.pub | ssh user1@server 'cat >> ~/.ssh/authorized_keys'
     ```

  3. Verify key on server:

     ```bash
     ssh user1@server cat ~/.ssh/authorized_keys
     ```

  4. Corner Case: Permissions issue:

     ```bash
     ssh-copy-id user1@server
     # Error: Permission denied
     # Fix on server:
     ssh user1@server
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
     ```

### Using ssh-agent for Non-Interactive Authentication

- **Purpose**: Manage private keys for passwordless SSH without entering passphrases repeatedly.
- **Commands**:
  - `ssh-agent`: Start agent.
  - `ssh-add`: Add key to agent.
- **Examples**:
  1. Start `ssh-agent`:

     ```bash
     eval "$(ssh-agent)"
     ```

     Output: `Agent pid 1234`.
  2. Add private key:

     ```bash
     ssh-add ~/.ssh/id_rsa
     ```

     Output: Prompts for passphrase if set.
  3. Test passwordless login:

     ```bash
     ssh user1@192.168.1.100
     ```

  4. Corner Case: Agent not running:

     ```bash
     ssh-add
     # Error: Could not open connection to agent
     # Fix: eval "$(ssh-agent)"
     ```

  5. List loaded keys:

     ```bash
     ssh-add -l
     ```

### Customizing OpenSSH Service Configuration

- **File**: `/etc/ssh/sshd_config` (server configuration).
- **Key Directives**:
  - `Port`: Change default port (e.g., `2222`).
  - `PermitRootLogin`: Enable/disable root login.
  - `PasswordAuthentication`: Enable/disable passwords.
  - `AllowUsers`: Restrict SSH to specific users.
- **Process**: Edit, test, reload `sshd`.
- **Examples**:
  1. Change SSH port to 2222:

     ```bash
     sudo vim /etc/ssh/sshd_config
     # Set: Port 2222
     sudo sshd -t  # Test config
     sudo systemctl reload sshd
     sudo firewall-cmd --add-port=2222/tcp --permanent
     sudo firewall-cmd --reload
     ```

  2. Restrict to specific users:

     ```bash
     sudo vim /etc/ssh/sshd_config
     # Add: AllowUsers user1 user2
     sudo systemctl reload sshd
     ```

  3. Corner Case: Syntax error in `sshd_config`:

     ```bash
     sudo sshd -t
     # Error: Bad configuration
     # Fix: Correct syntax, retest, reload
     ```

  4. Backup and edit:

     ```bash
     sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
     sudo vim /etc/ssh/sshd_config
     ```

### Prohibiting the Superuser from Logging In Using SSH

- **Purpose**: Prevent root login to reduce attack surface.
- **Directive**: `PermitRootLogin no` in `/etc/ssh/sshd_config`.
- **Examples**:
  1. Disable root login:

     ```bash
     sudo vim /etc/ssh/sshd_config
     # Set: PermitRootLogin no
     sudo systemctl reload sshd
     ```

  2. Test:

     ```bash
     ssh root@server
     # Output: Permission denied
     ```

  3. Corner Case: Root still needed:

     ```bash
     # Use sudo user: ssh user1@server; sudo -i
     # Or set: PermitRootLogin prohibit-password (allows key-based root login)
     ```

  4. Verify configuration:

     ```bash
     sudo sshd -T | grep permitrootlogin
     ```

### Prohibiting Password Authentication

- **Purpose**: Enforce key-based authentication for security.
- **Directive**: `PasswordAuthentication no` in `/etc/ssh/sshd_config`.
- **Examples**:
  1. Disable password authentication:

     ```bash
     sudo vim /etc/ssh/sshd_config
     # Set: PasswordAuthentication no
     sudo systemctl reload sshd
     ```

  2. Test with password:

     ```bash
     ssh user1@server
     # Output: Permission denied (passwords disabled)
     ```

  3. Ensure key-based works:

     ```bash
     ssh-copy-id user1@server
     ssh user1@server  # Should connect
     ```

  4. Corner Case: No keys configured:

     ```bash
     # Users without keys can’t log in
     # Fix: Set up keys first with ssh-copy-id
     ```

### Practical Examples

1. **Set Up Secure SSH Access**:

   ```bash
   ssh-keygen -t ed25519
   ssh-copy-id user1@192.168.1.100
   sudo vim /etc/ssh/sshd_config
   # Set: PermitRootLogin no
   # Set: PasswordAuthentication no
   sudo systemctl reload sshd
   ssh user1@192.168.1.100  # Test passwordless login
   ```

2. **Change SSH Port and Update Firewall**:

   ```bash
   sudo vim /etc/ssh/sshd_config
   # Set: Port 2222
   sudo sshd -t
   sudo systemctl reload sshd
   sudo firewall-cmd --add-port=2222/tcp --permanent
   sudo firewall-cmd --remove-port=22/tcp --permanent
   sudo firewall-cmd --reload
   ssh -p 2222 user1@server
   ```

3. **Troubleshoot SSH Connection Failure**:

   ```bash
   ssh user1@server
   # Error: Connection refused
   systemctl status sshd  # Check if running
   journalctl -u sshd -b  # View logs
   sudo firewall-cmd --list-all  # Check firewall
   sudo systemctl start sshd  # Start if stopped
   ```

4. **Manage User Sessions**:

   ```bash
   w  # List SSH users
   sudo pkill -u user1  # Terminate user1’s sessions
   last | grep sshd  # Check login history
   ```

5. **Secure File Transfer**:

   ```bash
   scp /data/backup.tar.gz user1@server:/backup/
   sftp user1@server
   sftp> put logs.tar.gz
   sftp> get remote_data.txt
   ```

### Common Pitfalls

- **No `sudo`**: Editing `/etc/ssh/sshd_config` or restarting `sshd` requires root.
- **Config Errors**: Syntax errors break SSH:

  ```bash
  sudo sshd -t  # Test before reloading
  ```

- **Firewall Blocks**: Missing firewall rules block connections:

  ```bash
  sudo firewall-cmd --add-port=22/tcp --permanent
  sudo firewall-cmd --reload
  ```

- **Key Permissions**: Incorrect permissions on `~/.ssh` (700) or `authorized_keys` (600) cause failures:

  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```

- **Restart vs. Reload**: `systemctl restart sshd` disconnects sessions; use `reload`.
- **SELinux Issues**: SELinux may block SSH if contexts are wrong:

  ```bash
  sudo restorecon -R ~/.ssh
  ```

### Best Practices

- **Use Key-Based Authentication**: Set `PasswordAuthentication no`.
- **Disable Root Login**: Set `PermitRootLogin no`.
- **Non-Standard Port**: Use ports like 2222 to reduce attacks.
- **Limit Users**: Use `AllowUsers user1 user2` in `sshd_config`.
- **Backup Config**: Save `/etc/ssh/sshd_config.bak` before changes.
- **Monitor Logs**: Check `journalctl -u sshd -f` for security events.
- **Test Safely**: Keep a console session open when modifying SSH.
- **RHEL 10**: Use Lightspeed (e.g., `rhel lightspeed "secure SSH server"`).

### Revision Quiz/Exercises

- **Questions**:
  1. How do you copy an SSH key to a server? (`ssh-copy-id user1@server`)
  2. What command disables password authentication? (`PasswordAuthentication no` in `sshd_config`)
  3. How do you check active SSH users? (`w` or `who`)
  4. Why use `ssh-agent`? (Non-interactive key authentication.)
- **Exercises**:
  1. Set up key-based authentication:

     ```bash
     ssh-keygen -t ed25519
     ssh-copy-id user1@192.168.1.100
     ssh user1@192.168.1.100
     ```

  2. Secure SSH server:

     ```bash
     sudo vim /etc/ssh/sshd_config
     # Set: PermitRootLogin no
     # Set: PasswordAuthentication no
     sudo systemctl reload sshd
     ```

  3. Troubleshoot connection:

     ```bash
     journalctl -u sshd -n 20
     sudo firewall-cmd --list-all
     sudo systemctl status sshd
     ```
