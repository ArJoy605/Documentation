# RH124 Revision Notebook: Chapter 9 - Control Services and Daemons

This notebook provides a detailed summary of Chapter 9 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on managing services and daemons using `systemd`, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 9: Control Services and Daemons

### Key Concepts
- **Services and Daemons**:
  - A **service** is a program or set of programs managed by the system, typically running in the background (e.g., `sshd` for SSH, `httpd` for web server).
  - A **daemon** is a background process, often ending in `d` (e.g., `crond` for scheduled tasks).
  - Services are critical for system functionality, such as networking, logging, or web hosting.
- **Systemd Overview**:
  - `systemd` is the default init system in RHEL, responsible for starting, stopping, and managing services.
  - Units: Services (`.service`), timers (`.timer`), mounts (`.mount`), etc.
  - Configuration files: Stored in `/etc/systemd/system` (custom) or `/usr/lib/systemd/system` (default).
- **Service States**:
  - **Enabled**: Starts automatically at boot.
  - **Disabled**: Does not start at boot.
  - **Running/Active**: Currently executing.
  - **Stopped/Inactive**: Not running but may be started.
  - **Failed**: Service failed to start or crashed.
- **Service Management**:
  - Use `systemctl` to control services (start, stop, enable, disable, check status).
  - Services can be system-wide or user-specific (e.g., user services in `~/.config/systemd/user`).
- **RHEL 9/10 Notes**:
  - Improved `systemd` performance and logging in RHEL 10.
  - RHEL Lightspeed (RHEL 10) can suggest service management commands (e.g., `rhel lightspeed "restart web server"` suggests `systemctl restart httpd`).

### Important Commands
- **Service Status**:
  ```
  systemctl status sshd  # Show status of sshd service (active, inactive, failed)
  systemctl is-active sshd  # Check if running (returns "active" or "inactive")
  systemctl is-enabled sshd  # Check if enabled at boot
  ```
- **Managing Services**:
  ```
  systemctl start sshd  # Start service
  systemctl stop sshd  # Stop service
  systemctl restart sshd  # Stop and start service
  systemctl reload sshd  # Reload configuration without stopping (if supported)
  systemctl enable sshd  # Enable service to start at boot
  systemctl disable sshd  # Disable auto-start at boot
  systemctl enable --now sshd  # Enable and start immediately
  ```
- **Listing Services**:
  ```
  systemctl list-units --type=service  # List all loaded services
  systemctl list-units --type=service --all  # Include inactive services
  systemctl list-unit-files --type=service  # List service files (enabled/disabled)
  ```
- **Troubleshooting Services**:
  ```
  journalctl -u sshd  # View logs for sshd service
  journalctl -xe  # View recent system logs with errors
  systemctl is-failed sshd  # Check if service failed
  ```
- **Masking Services**:
  ```
  systemctl mask sshd  # Prevent service from starting
  systemctl unmask sshd  # Re-enable service
  ```

### Practical Examples
- **Scenario: Manage SSH Service**:
  - Check if SSH is running:
    ```
    systemctl status sshd
    ```
    Output: Shows `active (running)` or `inactive`.
  - Start SSH if stopped:
    ```
    sudo systemctl start sshd
    ```
  - Enable at boot:
    ```
    sudo systemctl enable sshd
    ```
- **Scenario: Restart Web Server**:
  - Restart Apache (`httpd`) after configuration change:
    ```
    sudo systemctl restart httpd
    ```
  - Reload without interrupting connections (if supported):
    ```
    sudo systemctl reload httpd
    ```
  - Verify: `systemctl status httpd`.
- **Scenario: Troubleshoot Failed Service**:
  - Check if `httpd` failed:
    ```
    systemctl is-failed httpd
    journalctl -u httpd -b  # View logs since last boot
    ```
  - Fix (e.g., correct config in `/etc/httpd/conf/httpd.conf`), then restart:
    ```
    sudo systemctl restart httpd
    ```
- **Scenario: Prevent Unwanted Service**:
  - Mask a service to prevent accidental start:
    ```
    sudo systemctl mask bluetooth
    systemctl status bluetooth  # Shows "masked"
    ```
- **Daily Use: List Running Services**:
  - View active services:
    ```
    systemctl list-units --type=service --state=running
    ```
  - Save to file for audit:
    ```
    systemctl list-units --type=service --state=running > services.txt
    ```
- **Scenario: Monitor Service Logs**:
  - Watch real-time logs for `crond`:
    ```
    journalctl -u crond -f
    ```

### Common Pitfalls
- **Forgetting `sudo`**: Service management requires root privileges (e.g., `sudo systemctl start sshd`).
- **Restart vs. Reload**: `restart` stops and starts, potentially disrupting users; use `reload` when possible (e.g., `systemctl reload httpd`).
- **Masked Services**: Attempting to start a masked service fails. Check with `systemctl status` and unmask if needed.
- **Ignoring Logs**: Service failures often have clues in `journalctl -u service`. Always check logs before retrying.
- **Disabling Critical Services**: Disabling services like `sshd` can lock you out of remote access. Verify impact first.
- **Unit Not Found**: Misspelling service names (e.g., `systemctl start ssh` instead of `sshd`) causes errors. Use tab completion.

### Best Practices and Tips
- **Check Status First**: Always run `systemctl status service` before and after changes to confirm state.
- **Use `reload` When Possible**: Minimizes downtime (e.g., `systemctl reload httpd` for config changes).
- **Monitor Logs**: Use `journalctl -u service -f` for real-time debugging.
- **Enable Essential Services**: Ensure critical services (e.g., `sshd`, `crond`) are enabled (`systemctl enable`).
- **Script Service Checks**: Automate status checks:
  ```
  for s in sshd httpd; do systemctl status $s >> service_status.log; done
  ```
- **RHEL 10 Tip**: Use Lightspeed for service tasks (e.g., `rhel lightspeed "check service status"` suggests `systemctl status`).
- **Backup Configurations**: Before editing service configs (e.g., `/etc/httpd`), back up (e.g., `sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak`).

### Revision Quiz/Notes
- **Questions**:
  - What command starts and enables `httpd` in one step? (`systemctl enable --now httpd`)
  - What’s the difference between `restart` and `reload`? (`restart` stops and starts; `reload` updates config without stopping.)
  - How do you view logs for `sshd`? (`journalctl -u sshd`)
- **Quick Exercise**:
  - Check if `crond` is running, enable it, and view its logs:
    ```
    systemctl status crond
    sudo systemctl enable crond
    journalctl -u crond -n 10
    ```
- **Self-Test**:
  - Explain the output of `systemctl status sshd` (shows service state, PID, recent logs).
  - Why mask a service? (Prevents accidental start, e.g., for security or resource management.)

---

This detailed note on Chapter 9 equips you for managing services and daemons in RHEL. Practice these commands in a RHEL virtual machine (e.g., via KVM or VirtualBox) to build proficiency. If you want to add other chapters (e.g., Chapter 10) to this notebook, combine with previous chapters, or need specific examples (e.g., scripting service management), let me know!