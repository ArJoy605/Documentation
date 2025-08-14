# Chapter 11 - Analyze and Store Logs

## Key Concepts

- **System Logging**:
  - Logs capture system events (e.g., errors, user logins, service activities) for troubleshooting, auditing, and monitoring.
  - Managed by `rsyslog` (traditional logs) and `systemd-journald` (journal logs) in RHEL.
- **System Log Architecture**:
  - **rsyslog**: Writes logs to text files (e.g., `/var/log/messages`, `/var/log/secure`).
  - **systemd-journald**: Stores logs in binary format, accessible via `journalctl`.
  - **logrotate**: Manages log file size and rotation to prevent disk space issues.
  - Logs can be stored in memory, on disk, or forwarded to remote servers.
- **RHEL 9/10 Notes**:
  - RHEL 10 improves `journald` querying and integrates with RHEL Lightspeed (e.g., `rhel lightspeed "find SSH errors"` suggests `journalctl -u sshd | grep Failed`).
  - Enhanced `chronyd` for precise time synchronization, critical for log accuracy.

### Describing the System Log Architecture

- **Components**:
  - **rsyslog**: Traditional logging system, writes to `/var/log` files (e.g., `/var/log/messages` for general events, `/var/log/secure` for authentication).
  - **systemd-journald**: Binary logging, stores in memory or `/var/log/journal` (if persistent).
  - **logrotate**: Rotates, compresses, and deletes old logs.
  - **chronyd**: Ensures accurate timestamps for logs.
- **Interaction**:
  - `rsyslog` and `journald` can coexist; `journald` forwards logs to `rsyslog` by default.
  - Logs can be local or sent to a remote syslog server.
- **Example**:
  - Check logging services:

    ```bash
    systemctl status rsyslog systemd-journald
    ```

    Output: Shows `active (running)` for both.

### System Logging

- **Purpose**: Record system and application events for diagnostics and security.
- **Files**:
  - `/var/log/messages`: General system messages (not used if `journald` is exclusive).
  - `/var/log/secure` or `/var/log/auth.log`: Authentication events.
  - `/var/log/cron`: Cron job logs.
  - `/var/log/boot.log`: Boot process messages.
  - Service-specific: E.g., `/var/log/httpd/` for Apache.
- **Examples**:
  1. View recent system messages:

     ```bash
     sudo cat /var/log/messages | tail -n 10
     ```

  2. Check authentication logs:

     ```bash
     sudo less /var/log/secure
     ```

  3. Corner Case: Missing `/var/log/messages`:

     ```bash
     # If empty, check journald: journalctl -b
     # Or verify rsyslog: systemctl status rsyslog
     ```

### Uses of `less` and `tail`

- **Purpose**: Efficiently view and monitor log files.
- **Commands**:
  - `less`: Interactive browsing (navigate with arrows, `/` to search, `q` to quit).
  - `tail`: Show last lines or follow logs in real-time.
- **Examples**:
  1. Browse `/var/log/secure`:

     ```bash
     sudo less /var/log/secure
     # Search: /Failed
     ```

  2. Monitor cron logs live:

     ```bash
     sudo tail -f /var/log/cron
     ```

  3. Show last 20 lines of boot log:

     ```bash
     sudo tail -n 20 /var/log/boot.log
     ```

  4. Corner Case: Large log file crashes `less`:

     ```bash
     sudo tail -n 100 /var/log/messages | less
     ```

### Selected System Log Files

- **Key Files**:
  - `/var/log/messages`: System-wide events (e.g., service starts, errors).
  - `/var/log/secure`: Authentication (e.g., SSH logins, sudo).
  - `/var/log/cron`: Cron job executions.
  - `/var/log/boot.log`: Boot messages.
  - `/var/log/dmesg`: Kernel messages (use `dmesg` command).
- **Examples**:
  1. Check SSH login attempts:

     ```bash
     sudo cat /var/log/secure | grep sshd
     ```

  2. View cron job failures:

     ```bash
     sudo grep "ERROR" /var/log/cron
     ```

  3. Inspect boot issues:

     ```bash
     sudo less /var/log/boot.log
     ```

  4. Corner Case: Log file not found:

     ```bash
     # If /var/log/messages missing, use: journalctl -b > messages.log
     ```

### Reviewing Syslog Files

- **Purpose**: Analyze traditional log files for system events.
- **Commands**: `cat`, `less`, `grep`, `tail`.
- **Examples**:
  1. Find SSH errors:

     ```bash
     sudo grep "Failed" /var/log/secure
     ```

  2. Monitor recent system messages:

     ```bash
     sudo tail -n 50 /var/log/messages
     ```

  3. Search for kernel issues:

     ```bash
     sudo dmesg | grep -i error
     ```

  4. Corner Case: Log file rotated:

     ```bash
     sudo zcat /var/log/messages-20250801.gz | grep error
     ```

### Logging Events to the System (Rsyslog)

- **Rsyslog**: Processes and routes log messages based on `/etc/rsyslog.conf` and `/etc/rsyslog.d/`.
- **Facilities**: `auth`, `cron`, `kern`, `user`, etc.
- **Priorities**: `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug`.
- **Configuration**:
  - Example rule in `/etc/rsyslog.conf`:

    ```bash
    auth.*    /var/log/secure
    cron.*    /var/log/cron
    ```

- **Examples**:
  1. Check `rsyslog` status:

     ```bash
     systemctl status rsyslog
     ```

  2. Log a custom message:

     ```bash
     logger -p user.info "Test message from user1"
     sudo tail /var/log/messages
     ```

  3. Configure `rsyslog` to log SSH to a custom file:

     ```bash
     sudo vim /etc/rsyslog.d/ssh.conf
     # Add: auth.*    /var/log/ssh.log
     sudo systemctl restart rsyslog
     ```

  4. Corner Case: `rsyslog` not running:

     ```bash
     systemctl start rsyslog
     ```

### Sending Syslog Messages Manually

- **Command**: `logger` sends custom messages to `rsyslog`.
- **Examples**:
  1. Log a warning:

     ```bash
     logger -p user.warning "Disk usage high"
     sudo tail /var/log/messages
     ```

  2. Log with tag:

     ```bash
     logger -t MYAPP "Application started"
     ```

  3. Log to specific facility:

     ```bash
     logger -p cron.err "Cron job failed"
     sudo tail /var/log/cron
     ```

  4. Corner Case: `logger` ignored:

     ```bash
     # Check rsyslog rules: sudo grep user /etc/rsyslog.conf
     ```

### Logrotate

- **Purpose**: Manages log file size by rotating, compressing, and deleting old logs.
- **Configuration**: `/etc/logrotate.conf` (global), `/etc/logrotate.d/` (service-specific).
- **Key Options**: `daily`, `weekly`, `rotate <count>`, `compress`, `missingok`.
- **Examples**:
  1. Check logrotate config:

     ```bash
     cat /etc/logrotate.d/httpd
     ```

  2. Create custom rotation:

     ```bash
     sudo vim /etc/logrotate.d/myapp
     # Add:
     /var/log/myapp.log {
         weekly
         rotate 4
         compress
         missingok
     }
     ```

  3. Test rotation:

     ```bash
     sudo logrotate -v /etc/logrotate.d/myapp
     ```

  4. Force rotation:

     ```bash
     sudo logrotate -f /etc/logrotate.conf
     ```

  5. Corner Case: Rotation fails due to permissions:

     ```bash
     sudo chown root:root /var/log/myapp.log
     sudo logrotate -f /etc/logrotate.d/myapp
     ```

### Reviewing System Journal Entries

- **Purpose**: Analyze binary logs managed by `systemd-journald`.
- **Commands**:
  - `journalctl`: View journal entries.
  - Filters: `-u` (service), `-p` (priority), `--since`, `--until`, `-b` (boot).
- **Examples**:
  1. View all logs since boot:

     ```bash
     journalctl -b
     ```

  2. Filter SSH logs:

     ```bash
     journalctl -u sshd | grep Failed
     ```

  3. Show errors only:

     ```bash
     journalctl -p 3
     ```

  4. View logs for a time range:

     ```bash
     journalctl --since "2025-08-14 09:00" --until "2025-08-14 10:00"
     ```

  5. Corner Case: No logs shown:

     ```bash
     # Check journald: systemctl status systemd-journald
     # Enable persistence: sudo mkdir -p /var/log/journal
     ```

### Monitoring Logs

- **Purpose**: Watch logs in real-time for debugging or monitoring.
- **Commands**: `journalctl -f`, `tail -f`.
- **Examples**:
  1. Monitor SSH logs live:

     ```bash
     journalctl -u sshd -f
     ```

  2. Watch cron logs:

     ```bash
     sudo tail -f /var/log/cron
     ```

  3. Monitor kernel messages:

     ```bash
     dmesg --follow
     ```

  4. Corner Case: Logs truncated:

     ```bash
     journalctl -u httpd -f -o verbose  # Include metadata
     ```

### Preserving the System Journal

- **Purpose**: Retain journal logs across reboots (default is in-memory).
- **Configuration**: Enable `/var/log/journal`.
- **Examples**:
  1. Enable persistent storage:

     ```bash
     sudo mkdir -p /var/log/journal
     sudo systemd-tmpfiles --create --prefix /var/log/journal
     ```

  2. Verify persistence:

     ```bash
     journalctl --disk-usage
     ```

  3. Check logs from previous boot:

     ```bash
     journalctl -b -1
     ```

  4. Corner Case: No disk space:

     ```bash
     df -h /var/log
     sudo journalctl --vacuum-size=500M
     ```

### Storing the System Journal Permanently

- **Purpose**: Ensure logs are saved to disk for auditing.
- **Configuration**: Edit `/etc/systemd/journald.conf`.
- **Example**:
  1. Enable persistent storage:

     ```bash
     sudo vim /etc/systemd/journald.conf
     # Set: Storage=persistent
     sudo systemctl restart systemd-journald
     ```

  2. Verify:

     ```bash
     ls /var/log/journal
     ```

  3. Corner Case: Journal corruption:

     ```bash
     sudo journalctl --verify
     # Fix: sudo journalctl --vacuum-time=1s
     ```

### Configuring Persistent System Journal

- **File**: `/etc/systemd/journald.conf`.
- **Key Options**:
  - `Storage=persistent`: Store logs on disk.
  - `SystemMaxUse`: Limit journal size (e.g., `500M`).
  - `MaxRetentionSec`: Limit retention time (e.g., `604800` for 7 days).
- **Examples**:
  1. Limit journal to 1GB:

     ```bash
     sudo vim /etc/systemd/journald.conf
     # Set: SystemMaxUse=1G
     sudo systemctl restart systemd-journald
     ```

  2. Retain logs for 14 days:

     ```bash
     sudo vim /etc/systemd/journald.conf
     # Set: MaxRetentionSec=1209600
     sudo systemctl restart systemd-journald
     ```

  3. Check size:

     ```bash
     journalctl --disk-usage
     ```

  4. Corner Case: Config syntax error:

     ```bash
     sudo journalctl -b | grep journald.conf
     # Fix: Correct syntax, restart service
     ```

### Maintaining Accurate Time

- **Purpose**: Ensure log timestamps are accurate for correlation and auditing.
- **Tools**: `chronyd` (default in RHEL), `timedatectl`.
- **Examples**:
  1. Check system time:

     ```bash
     timedatectl
     ```

  2. Set time zone:

     ```bash
     sudo timedatectl set-timezone America/New_York
     ```

  3. Corner Case: Incorrect time:

     ```bash
     sudo chronyc makestep  # Force sync with NTP
     ```

### Setting Local Clocks and Time Zones

- **Commands**:
  - `timedatectl`: View/set time and time zone.
  - `timedatectl list-timezones`: List available time zones.
- **Examples**:
  1. View time status:

     ```bash
     timedatectl
     ```

     Output: Shows time, time zone, NTP status.
  2. Set time zone:

     ```bash
     sudo timedatectl set-timezone Europe/London
     ```

  3. Set manual time:

     ```bash
     sudo timedatectl set-time "2025-08-14 14:00:00"
     ```

  4. Corner Case: Time zone not found:

     ```bash
     timedatectl list-timezones | grep London
     ```

### Configuring and Monitoring Chronyd

- **Purpose**: Synchronize system time with NTP servers using `chronyd`.
- **Configuration**: `/etc/chrony.conf` (NTP servers, options).
- **Commands**:
  - `chronyc`: Manage `chronyd`.
  - `systemctl status chronyd`: Check service.
- **Examples**:
  1. Check `chronyd` status:

     ```bash
     systemctl status chronyd
     ```

  2. Verify NTP sync:

     ```bash
     chronyc sources
     ```

  3. Add custom NTP server:

     ```bash
     sudo vim /etc/chrony.conf
     # Add: server ntp.example.com iburst
     sudo systemctl restart chronyd
     ```

  4. Force time sync:

     ```bash
     sudo chronyc makestep
     ```

  5. Corner Case: `chronyd` not syncing:

     ```bash
     sudo systemctl restart chronyd
     chronyc tracking
     ```

### Practical Examples

1. **Troubleshoot SSH Failures**:

   ```bash
   journalctl -u sshd -b | grep Failed > ssh_errors.log
   sudo less /var/log/secure
   ```

2. **Monitor Cron Jobs**:

   ```bash
   sudo tail -f /var/log/cron
   logger -p cron.info "Test cron event"
   ```

3. **Configure Log Rotation**:

   ```bash
   sudo vim /etc/logrotate.d/custom
   # Add:
   /var/log/custom.log {
       daily
       rotate 5
       compress
       missingok
   }
   sudo logrotate -v /etc/logrotate.d/custom
   ```

4. **Set Up Persistent Journal**:

   ```bash
   sudo mkdir -p /var/log/journal
   sudo vim /etc/systemd/journald.conf
   # Set: Storage=persistent
   sudo systemctl restart systemd-journald
   journalctl --disk-usage
   ```

5. **Sync Time for Logs**:

   ```bash
   timedatectl set-timezone America/Chicago
   sudo systemctl restart chronyd
   chronyc sources
   ```

### Common Pitfalls

- **Non-Persistent Journals**: Logs lost on reboot without `/var/log/journal`:

  ```bash
  sudo mkdir -p /var/log/journal
  sudo systemd-tmpfiles --create --prefix /var/log/journal
  ```

- **Rsyslog Disabled**: No `/var/log/messages` if `rsyslog` is stopped:

  ```bash
  sudo systemctl start rsyslog
  ```

- **Logrotate Fails**: Incorrect permissions or syntax:

  ```bash
  sudo logrotate -d /etc/logrotate.conf  # Debug
  ```

- **Time Sync Issues**: Logs have wrong timestamps if `chronyd` fails:

  ```bash
  chronyc tracking
  ```

- **Journal Size**: Large journals fill disk:

  ```bash
  sudo journalctl --vacuum-size=500M
  ```

### Best Practices

- **Enable Persistent Journals**: Set `Storage=persistent` in `/etc/systemd/journald.conf`.
- **Use Specific Filters**: `journalctl -u`, `-p`, `--since` for targeted analysis.
- **Automate Log Checks**:

  ```bash
  journalctl -p 3 -b > /tmp/errors.log
  ```

- **Configure Logrotate**: Use `weekly`, `rotate 4`, `compress` for efficiency.
- **Sync Time**: Ensure `chronyd` is active (`systemctl enable chronyd`).
- **RHEL 10**: Use Lightspeed (e.g., `rhel lightspeed "analyze logs"`).

### Revision Quiz/Exercises

- **Questions**:
  1. How do you view `httpd` logs since yesterday? (`journalctl -u httpd --since "yesterday"`)
  2. What command rotates logs manually? (`logrotate -f /etc/logrotate.conf`)
  3. How do you set the time zone to Tokyo? (`timedatectl set-timezone Asia/Tokyo`)
- **Exercises**:
  1. Monitor SSH logs:

     ```bash
     journalctl -u sshd -f
     ```

  2. Configure custom log rotation:

     ```bash
     sudo vim /etc/logrotate.d/test
     # Add: /var/log/test.log { daily, rotate 3 }
     sudo logrotate -v /etc/logrotate.d/test
     ```

  3. Sync time:

     ```bash
     sudo timedatectl set-timezone UTC
     sudo chronyc makestep
     ```
