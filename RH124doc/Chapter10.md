# RH124 Revision Notebook: Chapter 10 - Configure and Secure SSH

This notebook provides a detailed summary of Chapter 10 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on configuring and securing the Secure Shell (SSH) service for remote access, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 10: Configure and Secure SSH

### Key Concepts

- **Secure Shell (SSH)**:
  - SSH is a protocol for secure remote access and file transfer over untrusted networks.
  - Uses encryption (e.g., RSA, ECDSA) for authentication and data protection.
  - Primary service in RHEL: `sshd` (OpenSSH daemon).
- **SSH Components**:
  - **Client**: `ssh` for connecting to remote systems, `scp` for file transfers, `sftp` for secure file access.
  - **Server**: `sshd` listens for connections (default port: 22).
  - Configuration file: `/etc/ssh/sshd_config` (server) and `/etc/ssh/ssh_config` (client).
- **Authentication Methods**:
  - **Password-based**: User provides username and password.
  - **Key-based**: Uses public/private key pairs for stronger security.
  - **Other**: GSSAPI, Kerberos (not covered in RH124).
- **Key-Based Authentication**:
  - Private key: Stored on client (e.g., `~/.ssh/id_rsa`).
  - Public key: Stored on server (e.g., `~/.ssh/authorized_keys`).
  - More secure than passwords, eliminates brute-force risks.
- **Security Considerations**:
  - Restrict root login, change default port, or disable password authentication for enhanced security.
  - Use SELinux and firewalls (`firewalld`) to limit SSH access.
- **RHEL 9/10 Notes**:
  - OpenSSH in RHEL 10 uses stronger default ciphers (e.g., `ed25519` keys).
  - RHEL Lightspeed (RHEL 10) can suggest SSH configurations (e.g., `rhel lightspeed "disable root SSH login"`).

### Important Commands

- **Managing SSH Service**:

  ```bash
  systemctl status sshd  # Check if sshd is running
  systemctl start sshd  # Start sshd
  systemctl enable sshd  # Enable sshd at boot
  systemctl restart sshd  # Restart after config changes
  ```

- **Connecting via SSH**:

  ```bash
  ssh user1@192.168.1.100  # Connect to remote host
  ssh -p 2222 user1@server  # Connect on non-standard port
  ```

- **File Transfer**:

  ```bash
  scp file.txt user1@server:/home/user1  # Copy file to remote
  scp user1@server:/remote/file.txt .  # Copy file from remote
  sftp user1@server  # Interactive file transfer
  ```

- **Key-Based Authentication**:

  ```bash
  ssh-keygen -t rsa  # Generate RSA key pair (~/.ssh/id_rsa, id_rsa.pub)
  ssh-copy-id user1@server  # Copy public key to remote server
  ssh user1@server  # Connect using key-based authentication
  ```

- **Editing SSH Configuration**:

  ```bash
  sudo vim /etc/ssh/sshd_config  # Edit server config
  sudo systemctl reload sshd  # Apply changes without disconnecting
  ```

- **Checking SSH Logs**:

  ```bash
  journalctl -u sshd  # View sshd logs
  journalctl -u sshd -f  # Monitor logs in real-time
  ```

### Practical Examples

- **Scenario: Enable SSH Access**:
  - Ensure `sshd` is running and enabled:

    ```bash
    sudo systemctl status sshd
    sudo systemctl enable --now sshd
    ```

  - Verify: `ss -tuln | grep :22` (shows SSH listening on port 22).
- **Scenario: Set Up Key-Based Authentication**:
  - Generate key pair on client:

    ```bash
    ssh-keygen -t rsa -b 4096
    # Press Enter for default location (~/.ssh/id_rsa)
    ```

  - Copy public key to server:

    ```bash
    ssh-copy-id user1@192.168.1.100
    ```

  - Test: `ssh user1@192.168.1.100` (no password prompt).
- **Scenario: Secure SSH by Disabling Root Login**:
  - Edit `/etc/ssh/sshd_config`:

    ```bash
    sudo vim /etc/ssh/sshd_config
    # Change: PermitRootLogin no
    ```

  - Reload: `sudo systemctl reload sshd`.
  - Test: `ssh root@server` (should fail).
- **Scenario: Change SSH Port**:
  - Edit `/etc/ssh/sshd_config`:

    ```bash
    sudo vim /etc/ssh/sshd_config
    # Change: Port 2222
    ```

  - Update firewall: `sudo firewall-cmd --add-port=2222/tcp --permanent; sudo firewall-cmd --reload`.
  - Reload: `sudo systemctl reload sshd`.
  - Test: `ssh -p 2222 user1@server`.
- **Daily Use: Secure File Transfer**:
  - Copy a file to remote server:

    ```bash
    scp backup.tar.gz user1@server:/backup/
    ```

  - Verify: `ssh user1@server ls /backup`.
- **Scenario: Troubleshoot SSH Failure**:
  - Check logs for failed login:

    ```bash
    sudo journalctl -u sshd | grep "Failed"
    ```

  - Fix (e.g., unlock user: `sudo usermod -U user1` or correct config).

### Common Pitfalls

- **Forgetting `sudo`**: Editing `/etc/ssh/sshd_config` or restarting `sshd` requires root privileges.
- **Incorrect Config Syntax**: Errors in `sshd_config` can break SSH. Test config before reloading:

  ```bash
  sudo sshd -t
  ```

- **Firewall Blocking**: If SSH fails, check `firewalld`: `sudo firewall-cmd --list-all` (ensure port 22 or custom port is open).
- **Key Permission Issues**: `~/.ssh` and `authorized_keys` need strict permissions (600 for keys, 700 for directory). Fix:

  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```

- **Root Login Enabled**: Default `PermitRootLogin yes` is insecure. Set to `no` in production.
- **Disconnected Sessions**: Restarting `sshd` with `systemctl restart` may drop active sessions; use `reload` instead.

### Best Practices and Tips

- **Use Key-Based Authentication**: Disable password authentication in `/etc/ssh/sshd_config`:

  ```bash
  PasswordAuthentication no
  ```

- **Change Default Port**: Use a non-standard port (e.g., 2222) to reduce automated attacks.
- **Limit Users**: Restrict SSH access in `sshd_config`:

  ```bash
  AllowUsers user1 user2
  ```

- **Backup Config**: Before editing `sshd_config`, back up:

  ```bash
  sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
  ```

- **Monitor Logs**: Regularly check `journalctl -u sshd` for unauthorized access attempts.
- **RHEL 10 Tip**: Use Lightspeed for SSH tasks (e.g., `rhel lightspeed "secure SSH server"` suggests disabling root login).
- **Test Changes Safely**: Keep a local console or alternate SSH session open when modifying `sshd_config`.

### Revision Quiz/Notes

- **Questions**:
  - What command copies a public key to a server? (`ssh-copy-id user1@server`)
  - How do you disable root SSH login? (Edit `sshd_config`: `PermitRootLogin no`)
  - What’s the purpose of `journalctl -u sshd`? (View SSH service logs.)
- **Quick Exercise**:
  - Generate an SSH key, copy it to a server, and test passwordless login:

    ```bash
    ssh-keygen -t rsa
    ssh-copy-id user1@192.168.1.100
    ssh user1@192.168.1.100
    ```

- **Self-Test**:
  - Explain the difference between `scp` and `sftp` (`scp` for single file transfers, `sftp` for interactive sessions).
  - Why reload instead of restart `sshd`? (Avoids disconnecting active sessions.)

---
