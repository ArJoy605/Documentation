# RH124 Revision Notebook: Chapter 11 - Analyze and Store Logs

This notebook provides a detailed summary of Chapter 11 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on analyzing and storing system logs to troubleshoot issues and monitor system activity, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 11: Analyze and Store Logs

### Key Concepts

- **System Logging**:
  - Logs record system events, errors, and activities (e.g., user logins, service failures, security events).
  - Essential for troubleshooting, auditing, and monitoring system health.
- **Systemd Journal**:
  - Managed by `journald`, the default logging system in RHEL.
  - Stores logs in a binary format, accessible via `journalctl`.
  - Logs are stored in memory or `/var/log/journal` (if persistent storage is enabled).
- **Key Log Files**:
  - `/var/log/messages`: General system messages (not used if `journald` is exclusive).
  - `/var/log/secure` or `/var/log/auth.log`: Authentication and security events.
  - `/var/log/boot.log`: Boot-related messages.
  - `/var/log/cron`: Cron job activities.
  - Service-specific logs: E.g., `/var/log/httpd/` for Apache.
- **Log Management**:
  - View, filter, and analyze logs with `journalctl`.
  - Configure log rotation to manage disk space (via `logrotate`).
- **Log Rotation**:
  - Managed by `logrotate` to compress, archive, or delete old logs.
  - Configuration: `/etc/logrotate.conf` (global) and `/etc/logrotate.d/` (service-specific).
- **RHEL 9/10 Notes**:
  - RHEL 10 enhances `journald` with faster querying and better integration with RHEL Lightspeed.
  - Lightspeed can assist with log analysis (e.g., `rhel lightspeed "find SSH login failures"` suggests `journalctl -u sshd | grep "Failed"`).

### Important Commands

- **Viewing Logs with `journalctl`**:

  ```bash
  journalctl  # View all logs since boot
  journalctl -b  # View logs from current boot
  journalctl -u sshd  # View logs for sshd service
  journalctl -f  # Follow logs in real-time (like tail -f)
  journalctl -p 3  # Show logs with priority 3 (error) or higher
  journalctl --since "2025-08-10"  # Logs since a date
  journalctl --since "2025-08-10 14:00" --until "2025-08-10 15:00"  # Logs in time range
  journalctl | grep "error"  # Filter for "error"
  ```

- **Log Navigation**:

  ```bash
  journalctl -r  # Show logs in reverse (newest first)
  journalctl -n 20  # Show last 20 lines
  journalctl -o verbose  # Detailed output with metadata
  ```

- **Managing Journal Storage**:

  ```bash
  journalctl --disk-usage  # Check journal size
  journalctl --vacuum-time=7d  # Delete logs older than 7 days
  journalctl --vacuum-size=500M  # Reduce journal to 500MB
  ```

- **Traditional Log Files**:

  ```bash
  cat /var/log/messages  # View general system logs
  less /var/log/secure  # View authentication logs
  tail -f /var/log/cron  # Monitor cron logs in real-time
  ```

- **Log Rotation**:

  ```bash
  logrotate -v /etc/logrotate.conf  # Test logrotate configuration
  logrotate -f /etc/logrotate.conf  # Force rotation
  ls /etc/logrotate.d/  # View service-specific rotation configs
  ```

### Practical Examples

- **Scenario: Troubleshoot SSH Login Failures**:
  - Check recent SSH logs:

    ```bash
    journalctl -u sshd -b | grep "Failed"
    ```

    Output: Shows failed login attempts (e.g., `Failed password for user1`).
  - Fix: Verify user account (`id user1`) or unlock (`sudo usermod -U user1`).
- **Scenario: Monitor Real-Time Service Activity**:

  - Watch `httpd` logs live:

    ```bash
    journalctl -u httpd -f
    ```

  - Stop with Ctrl+C.
- **Scenario: Analyze Disk Space Issues**:
  - Check journal size:

    ```bash
    journalctl --disk-usage
    ```

    Output: E.g., `Journals take up 1.2G on disk`.
  - Clean old logs:

    ```bash
    sudo journalctl --vacuum-time=30d
    ```

- **Scenario: Configure Log Rotation**:
  - Create custom rotation for `/var/log/myapp.log`:

    ```bash
    sudo vim /etc/logrotate.d/myapp
    # Add:
    /var/log/myapp.log {
        daily
        rotate 7
        compress
        missingok
    }
    ```

  - Test: `sudo logrotate -v /etc/logrotate.d/myapp`.
- **Daily Use: Save Critical Logs**:
  - Export SSH logs for audit:

    ```bash
    journalctl -u sshd --since "2025-08-01" > sshd_audit.log
    ```

- **Scenario: Check Boot Issues**:
  - View boot logs:

    ```bash
    journalctl -b -1  # Previous boot
    journalctl -b | grep "Failed"  # Boot failures
    ```

### Common Pitfalls

- **Non-Persistent Journals**: By default, journals are in memory and lost on reboot unless `/var/log/journal` is enabled. Enable persistence:

  ```bash
  sudo mkdir -p /var/log/journal
  sudo systemd-tmpfiles --create --prefix /var/log/journal
  ```

- **Missing Logs**: If `journalctl` shows no data, ensure `systemd-journald` is running (`systemctl status systemd-journald`).
- **Overlooking `logrotate`**: Large logs can fill disks. Check `/etc/logrotate.conf` for rotation settings.
- **Incorrect Filters**: `journalctl` filters (e.g., `--since`) use strict formats (e.g., `2025-08-10 14:00:00`). Verify syntax.
- **Permission Issues**: Viewing some logs requires `sudo` (e.g., `sudo cat /var/log/secure`).
- **Ignoring Service Logs**: General logs (`/var/log/messages`) may miss service-specific details. Use `journalctl -u service`.

### Best Practices and Tips

- **Enable Persistent Logging**: Ensure `/var/log/journal` exists for log retention across reboots.
- **Use `journalctl` Filters**: Narrow logs by service (`-u`), priority (`-p`), or time (`--since`) for efficiency.
- **Automate Log Analysis**: Script common checks:

  ```bash
  journalctl -p 3 -b > errors.log  # Save errors from current boot
  ```

- **Configure Log Rotation**: Set reasonable rotation policies (e.g., `weekly`, `rotate 4`) in `/etc/logrotate.d/`.
- **Monitor Disk Usage**: Regularly check `journalctl --disk-usage` and clean with `--vacuum-size`.
- **RHEL 10 Tip**: Use Lightspeed for log queries (e.g., `rhel lightspeed "show recent errors"` suggests `journalctl -p 3 -b`).
- **Backup Logs**: Archive critical logs before rotation:

  ```bash
  sudo tar czf /backup/logs_$(date +%F).tar.gz /var/log
  ```

### Revision Quiz/Notes

- **Questions**:
  - What command shows logs for `crond` since yesterday? (`journalctl -u crond --since "yesterday"`)
  - How do you clean journals older than 14 days? (`journalctl --vacuum-time=14d`)
  - What’s the purpose of `/etc/logrotate.d/`? (Service-specific log rotation configs.)
- **Quick Exercise**:
  - View `sshd` logs for the last hour and save errors to a file:

    ```bash
    journalctl -u sshd --since "1 hour ago" | grep -i "error" > sshd_errors.log
    cat sshd_errors.log
    ```

- **Self-Test**:
  - Explain the difference between `journalctl -b` and `journalctl -b -1` (current vs. previous boot).
  - Why enable persistent journals? (Retains logs across reboots for troubleshooting.)

---
