# Chapter 9: Control Services and Daemons

## Key Concepts

- **Services and Daemons**:
  - A **service** is a managed program, typically running in the background (e.g., `sshd` for SSH, `httpd` for web server).
  - A **daemon** is a background process, often ending in `d` (e.g., `crond` for scheduling, `cupsd` for printing).
  - Services provide critical system functions like networking, logging, or web hosting.
- **Introduction to Systemd**:
  - `systemd` is the default init system in RHEL, managing system startup, services, and resources.
  - Organizes tasks into **units** (e.g., `.service` for services, `.timer` for scheduled tasks, `.mount` for mounts).
  - Configuration files: `/usr/lib/systemd/system` (default), `/etc/systemd/system` (custom overrides).
  - Benefits: Parallel startup, dependency management, detailed logging via `journalctl`.
- **RHEL 9/10 Notes**:
  - Enhanced `systemd` performance and logging in RHEL 10.
  - RHEL Lightspeed (RHEL 10) suggests commands (e.g., `rhel lightspeed "restart web server"` suggests `systemctl restart httpd`).

### Identifying Automatically Started System Processes

- **Purpose**: Determine which services start at boot to ensure critical functionality or optimize resources.
- **Commands**:
  - `systemctl list-unit-files --type=service --state=enabled`: List services set to start at boot.
  - `systemctl list-units --type=service --state=running`: List currently running services.
- **Examples**:
  1. List enabled services:

     ```bash
     systemctl list-unit-files --type=service --state=enabled | grep enabled
     ```

     Output: Shows services like `sshd.service`, `crond.service`.
  2. Check running services:

     ```bash
     systemctl list-units --type=service --state=running
     ```

     Output: Lists active services (e.g., `httpd`, `NetworkManager`).
  3. Verify specific service (e.g., `sshd`):

     ```bash
     systemctl is-enabled sshd
     ```

     Output: `enabled` (starts at boot) or `disabled`.

### Describing Service Units

- **Definition**: A service unit (`.service` file) defines how `systemd` manages a service, including start/stop commands, dependencies, and settings.
- **Location**:
  - Default: `/usr/lib/systemd/system` (system-provided, read-only).
  - Custom: `/etc/systemd/system` (user overrides or custom services).
- **Structure**:
  - `[Unit]`: Metadata (e.g., `Description`, `After` for dependencies).
  - `[Service]`: Execution details (e.g., `ExecStart`, `Type`).
  - `[Install]`: Boot behavior (e.g., `WantedBy=multi-user.target`).
- **Example**:
  - View `sshd.service`:

     ```bash
     cat /usr/lib/systemd/system/sshd.service
     ```

     Output: Shows `ExecStart=/usr/sbin/sshd -D`, `WantedBy=multi-user.target`.

- **Examples**:
  1. Check `httpd` service unit:

     ```bash
     systemctl cat httpd
     ```

     Output: Displays unit file contents.
  2. Create a custom service (e.g., `myapp.service`):

     ```bash
     sudo nano /etc/systemd/system/myapp.service
     # Content:
     [Unit]
     Description=My Custom App
     After=network.target
     [Service]
     ExecStart=/usr/bin/myapp --option
     Restart=always
     [Install]
     WantedBy=multi-user.target
     ```

     Reload `systemd`: `sudo systemctl daemon-reload`.
  3. Verify custom service:

     ```bash
     systemctl status myapp
     ```

### Viewing Service States

- **States**:
  - **Active (running)**: Service is running (e.g., `sshd` handling SSH connections).
  - **Active (exited)**: Service ran once and exited (e.g., one-shot scripts).
  - **Inactive**: Stopped but can be started.
  - **Failed**: Failed to start or crashed.
  - **Enabled**: Starts at boot.
  - **Disabled**: Does not start at boot.
  - **Static**: Cannot be enabled/disabled; started by dependencies (e.g., `systemd-journald`).
- **Commands**:
  - `systemctl status <service>`: Shows state, PID, logs.
  - `systemctl is-active <service>`: Returns `active` or `inactive`.
  - `systemctl is-enabled <service>`: Returns `enabled`, `disabled`, or `static`.

- **Examples**:
  1. Check `sshd` state:

     ```bash
     systemctl status sshd
     ```

     Output: `active (running)`, PID, recent logs.
  2. Verify if `httpd` is active:

     ```bash
     systemctl is-active httpd
     ```

     Output: `active` or `inactive`.
  3. Check if `crond` starts at boot:

     ```bash
     systemctl is-enabled crond
     ```

     Output: `enabled`.
  4. Identify static services:

     ```bash
     systemctl list-unit-files --type=service --state=static
     ```

### Listing Service Units

- **Purpose**: View all available, loaded, or running services to manage or troubleshoot.
- **Commands**:
  - `systemctl list-units --type=service`: List loaded, active services.
  - `systemctl list-units --type=service --all`: Include inactive services.
  - `systemctl list-unit-files --type=service`: List all service files (enabled, disabled, static).

- **Examples**:
  1. List running services:

     ```bash
     systemctl list-units --type=service --state=running
     ```

     Output: Shows `sshd`, `httpd`, etc.
  2. List all service units:

     ```bash
     systemctl list-units --type=service --all | grep service
     ```

     Output: Includes `inactive` services like `bluetooth`.
  3. List enabled and disabled services:

     ```bash
     systemctl list-unit-files --type=service | grep -E 'enabled|disabled'
     ```

  4. Save service list for audit:

     ```bash
     systemctl list-unit-files --type=service > /tmp/services_list.txt
     ```

### Verifying Service Status

- **Purpose**: Confirm if a service is running, enabled, or failed.
- **Commands**:
  - `systemctl status <service>`: Detailed status (state, PID, logs).
  - `systemctl is-active <service>`: Check running state.
  - `systemctl is-failed <service>`: Check if failed.
  - `journalctl -u <service>`: View service logs.

- **Examples**:
  1. Detailed status for `NetworkManager`:

     ```bash
     systemctl status NetworkManager
     ```

     Output: Shows `active (running)`, uptime, recent logs.
  2. Check if `httpd` is running:

     ```bash
     systemctl is-active httpd
     ```

     Output: `active` or `inactive`.
  3. Check for failed `httpd`:

     ```bash
     systemctl is-failed httpd
     ```

     Output: `failed` if crashed, else `active` or `inactive`.
  4. View `sshd` logs:

     ```bash
     journalctl -u sshd -n 20
     ```

### Controlling System Services

- **Purpose**: Start, stop, restart, or reload services to manage system functionality.
- **Commands**:
  - `systemctl start <service>`: Start service.
  - `systemctl stop <service>`: Stop service.
  - `systemctl restart <service>`: Stop and start.
  - `systemctl reload <service>`: Reload config without stopping (if supported).

- **Examples**:
  1. Start `httpd`:

     ```bash
     sudo systemctl start httpd
     systemctl status httpd
     ```

  2. Stop `bluetooth`:

     ```bash
     sudo systemctl stop bluetooth
     systemctl is-active bluetooth  # Output: inactive
     ```

  3. Restart `sshd`:

     ```bash
     sudo systemctl restart sshd
     ```

  4. Reload `httpd` after config change:

     ```bash
     sudo systemctl reload httpd
     journalctl -u httpd -f  # Monitor for errors
     ```

### Starting and Stopping Services

- **Details**:
  - Starting: Activates a service (e.g., for immediate use).
  - Stopping: Halts a service (e.g., to free resources or troubleshoot).
  - Use `sudo` for system services.
- **Examples**:
  1. Start `crond` for scheduling:

     ```bash
     sudo systemctl start crond
     systemctl status crond
     ```

  2. Stop `cups` printing service:

     ```bash
     sudo systemctl stop cups
     systemctl is-active cups  # Output: inactive
     ```

  3. Start `firewalld`:

     ```bash
     sudo systemctl start firewalld
     ```

  4. Stop `postfix` mail service:

     ```bash
     sudo systemctl stop postfix
     journalctl -u postfix -n 10  # Check for stop confirmation
     ```

### Restarting and Reloading Services

- **Restart**: Stops and starts a service (may interrupt users).
- **Reload**: Updates configuration without stopping (if supported, e.g., `httpd`, `sshd`).
- **Commands**:
  - `systemctl restart <service>`: Full restart.
  - `systemctl reload <service>`: Reload config.
- **Examples**:
  1. Restart `httpd` after updating `/etc/httpd/conf/httpd.conf`:

     ```bash
     sudo systemctl restart httpd
     systemctl status httpd
     ```

  2. Reload `sshd` after changing `/etc/ssh/sshd_config`:

     ```bash
     sudo systemctl reload sshd
     ```

  3. Restart `NetworkManager`:

     ```bash
     sudo systemctl restart NetworkManager
     journalctl -u NetworkManager -f
     ```

  4. Try reload, fallback to restart for `nginx`:

     ```bash
     sudo systemctl reload nginx || sudo systemctl restart nginx
     ```

### Listing Unit Dependencies

- **Purpose**: View services a unit depends on or is required by to troubleshoot startup issues.
- **Commands**:
  - `systemctl list-dependencies <service>`: Show dependency tree.
  - `systemctl show <service> -p Requires,Wants,After`: Show specific dependencies.
- **Examples**:
  1. View `httpd` dependencies:

     ```bash
     systemctl list-dependencies httpd
     ```

     Output: Shows dependencies like `network.target`.
  2. Check what `sshd` requires:

     ```bash
     systemctl show sshd -p Requires
     ```

  3. List services depending on `NetworkManager`:

     ```bash
     systemctl list-dependencies --reverse NetworkManager
     ```

  4. Debug startup failure:

     ```bash
     systemctl list-dependencies firewalld
     journalctl -u firewalld  # Check if dependencies failed
     ```

### Masking and Unmasking Services

- **Purpose**:
  - **Mask**: Prevents a service from starting (manually or automatically).
  - **Unmask**: Re-enables a masked service.
- **Commands**:
  - `systemctl mask <service>`: Disable start.
  - `systemctl unmask <service>`: Re-enable.
- **Examples**:
  1. Mask `bluetooth` to prevent startup:

     ```bash
     sudo systemctl mask bluetooth
     systemctl status bluetooth  # Output: masked
     ```

  2. Unmask `bluetooth`:

     ```bash
     sudo systemctl unmask bluetooth
     sudo systemctl start bluetooth
     ```

  3. Mask unused `avahi-daemon`:

     ```bash
     sudo systemctl mask avahi-daemon
     ```

  4. Verify masking:

     ```bash
     systemctl list-unit-files | grep masked
     ```

### Enabling Services to Start or Stop at Boot

- **Purpose**: Configure services to start (or not) during system boot.
- **Commands**:
  - `systemctl enable <service>`: Start at boot.
  - `systemctl disable <service>`: Prevent start at boot.
  - `systemctl enable --now <service>`: Enable and start immediately.
- **Examples**:
  1. Enable `httpd` at boot:

     ```bash
     sudo systemctl enable httpd
     systemctl is-enabled httpd  # Output: enabled
     ```

  2. Disable `cups` at boot:

     ```bash
     sudo systemctl disable cups
     systemctl is-enabled cups  # Output: disabled
     ```

  3. Enable and start `firewalld`:

     ```bash
     sudo systemctl enable --now firewalld
     systemctl status firewalld
     ```

  4. Disable `postfix`:

     ```bash
     sudo systemctl disable postfix
     systemctl list-unit-files | grep postfix
     ```

### Summary of systemctl Commands

- **Status and Verification**:
  - `systemctl status <service>`: Detailed state and logs.
  - `systemctl is-active <service>`: Check running status.
  - `systemctl is-enabled <service>`: Check boot status.
  - `systemctl is-failed <service>`: Check failure status.
- **Control**:
  - `start`, `stop`, `restart`, `reload`: Manage service state.
  - `enable`, `disable`, `enable --now`: Manage boot behavior.
  - `mask`, `unmask`: Prevent/allow starting.
- **Listing**:
  - `list-units --type=service`: Loaded services.
  - `list-units --type=service --all`: All services.
  - `list-unit-files --type=service`: All service files.
- **Dependencies and Logs**:
  - `list-dependencies <service>`: Dependency tree.
  - `journalctl -u <service>`: Service logs.

### Practical Examples

1. **Manage SSH Access**:

   ```bash
   systemctl status sshd          # Check state
   sudo systemctl restart sshd    # Restart after config change
   systemctl is-enabled sshd      # Confirm enabled
   journalctl -u sshd -n 20       # View recent logs
   ```

2. **Set Up Web Server**:

   ```bash
   sudo systemctl enable --now httpd  # Enable and start
   systemctl status httpd            # Verify
   sudo systemctl reload httpd       # Reload after config change
   systemctl list-dependencies httpd  # Check dependencies
   ```

3. **Troubleshoot Failed Service**:

   ```bash
   systemctl is-failed crond         # Check failure
   journalctl -u crond -b            # View logs
   sudo systemctl restart crond      # Restart after fixing
   ```

4. **Prevent Unnecessary Service**:

   ```bash
   sudo systemctl mask avahi-daemon  # Mask service
   systemctl status avahi-daemon     # Confirm masked
   sudo systemctl unmask avahi-daemon  # Re-enable if needed
   ```

5. **Monitor and Log Services**:

   ```bash
   journalctl -u NetworkManager -f    # Real-time logs
   systemctl list-units --type=service --state=running > /tmp/active_services.txt
   ```

6. **Custom Service Creation**:

   ```bash
   sudo nano /etc/systemd/system/backup.service
   # Content:
   [Unit]
   Description=Daily Backup Service
   After=network.target
   [Service]
   ExecStart=/bin/bash /scripts-UPDATEDscripts/backup.sh
   [Install]
   WantedBy=multi-user.target
   sudo systemctl daemon-reload
   sudo systemctl enable --now backup
   ```

### Common Pitfalls

- **No `sudo`**: Commands like `start`, `stop`, `enable` require root (e.g., `sudo systemctl start sshd`).
- **Restart vs. Reload**: `restart` disrupts users; use `reload` when possible (e.g., `systemctl reload httpd`).
- **Masked Services**: Starting a masked service fails; check with `systemctl status` and unmask.
- **Incorrect Names**: Misspelling (e.g., `ssh` vs. `sshd`) causes “unit not found” errors. Use tab completion.
- **Disabling Critical Services**: Disabling `sshd` may block remote access; verify impact.
- **Missing Dependencies**: A service may fail if dependencies (e.g., `network.target`) aren’t met.

### Best Practices

- **Always Check Status**: Use `systemctl status <service>` before/after changes.
- **Prefer `reload`**: Minimizes downtime (e.g., `systemctl reload sshd`).
- **Monitor Logs**: Use `journalctl -u <service> -f` for debugging.
- **Backup Configs**: Save copies before editing (e.g., `sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak`).
- **Automate Checks**:

  ```bash
  for s in sshd httpd crond; do systemctl status $s >> /tmp/service_status.log; done
  ```

- **RHEL 10**: Use Lightspeed (e.g., `rhel lightspeed "manage services"`).

### Revision Quiz/Exercises

- **Questions**:
  1. How do you enable and start `httpd` in one command? (`systemctl enable --now httpd`)
  2. What’s the difference between `restart` and `reload`? (`restart` stops/starts; `reload` updates config.)
  3. How do you check `sshd` logs? (`journalctl -u sshd`)
  4. Why mask a service? (Prevents starting for security/resource management.)
- **Exercises**:
  1. Manage `crond`:

     ```bash
     systemctl status crond
     sudo systemctl enable --now crond
     journalctl -u crond -n 10
     ```

  2. Troubleshoot `httpd`:

     ```bash
     systemctl is-failed httpd
     journalctl -u httpd -b
     sudo systemctl restart httpd
     ```

  3. Mask/unmask `bluetooth`:

     ```bash
     sudo systemctl mask bluetooth
     systemctl status bluetooth
     sudo systemctl unmask bluetooth
     ```
