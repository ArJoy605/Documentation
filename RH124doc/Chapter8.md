# RH124 Revision Notebook: Chapter 8 - Monitor and Manage Linux Processes

This notebook summarizes Chapter 8 of the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It covers monitoring and managing Linux processes, including process states, job control, signals, administrative user management, and real-time monitoring, with detailed commands, expanded practical examples, common pitfalls, best practices, and revision exercises.

## Chapter 8: Monitor and Manage Linux Processes

### Key Concepts

- **Processes in Linux**:
  - A process is a running instance of a program, identified by a unique Process ID (PID).
  - Types: User processes (e.g., `vim`), system processes (e.g., `systemd`), daemons (e.g., `sshd`).
  - Attributes: PID, parent process ID (PPID), user, priority, memory usage, state.
  - Managed by `systemd` in RHEL 9/10.

- **Why Process States Are Important**:
  - States reveal process behavior (e.g., running, waiting, or hung), aiding in performance tuning and troubleshooting.
  - Example: Identifying zombies (`Z` state) prevents resource leaks; high `D` state processes indicate I/O bottlenecks.

### Process States

- **Running (R)**: Actively using CPU.
- **Sleeping (S)**: Waiting for an event (interruptible, e.g., user input).
- **Uninterruptible Sleep (D)**: Waiting for I/O (e.g., disk, network).
- **Zombie (Z)**: Terminated but not reaped by parent.
- **Stopped (T)**: Paused (e.g., via `Ctrl+Z` or `SIGSTOP`).
- Check states in `ps` (`STAT` column) or `/proc/<pid>/stat`.

- **Examples**:

  1. View process states:

     ```bash
     ps aux | grep httpd  # Check if httpd is in R, S, or Z state
     ```

  2. Check a specific PID’s state:

     ```bash
     cat /proc/1234/stat | awk '{print $3}'  # Shows state (e.g., R, Z)
     ```

### Listing Processes

- **Commands**:
  - `ps`: Current shell processes.
  - `ps aux`: All processes (a: all users, u: user-oriented, x: no terminal).
  - `ps -ef`: All processes with PPID (full format).
  - `ps -u <user>`: User-specific processes.
  - `ps -C <command>`: Processes by command name.
  - `pgrep <name>`: List PIDs by name.
  - `pstree -p <user>`: Process hierarchy for a user.

- **`ps aux` vs `ps -aux`**:
  - `ps aux`: BSD style, shows all processes (USER, PID, %CPU, %MEM, STAT, COMMAND).
  - `ps -aux`: UNIX style, may filter for user `aux` or fail; use `-u <user>` for clarity.
  - Use `aux` for system-wide monitoring; `-aux` for user-specific queries.

- **Examples**:
  1. List all processes for user `alice`:

     ```bash
     ps -u alice  # Shows alice’s processes (e.g., bash, firefox)
     ```

  2. Find `sshd` processes:

     ```bash
     ps -C sshd  # Lists sshd processes
     pgrep -l sshd  # Shows sshd PIDs and names
     ```

  3. Show process hierarchy for `root`:

     ```bash
     pstree -p root  # Displays root’s process tree
     ```

  4. Compare `aux` and `-ef`:

     ```bash
     ps aux | head -5  # Top 5 processes by user format
     ps -ef | head -5  # Top 5 with PPID
     ```

### Controlling Jobs

- **Foreground vs Background**:
  - Foreground: Terminal-bound (e.g., `vim`).
  - Background: Detached (e.g., `command &`).
- **Commands**:
  - `command &`: Run in background.
  - `jobs`: List background jobs.
  - `fg %<job_id>`: Foreground a job.
  - `bg %<job_id>`: Resume in background.
  - `Ctrl+Z`: Suspend foreground process.

- **Examples**:
  1. Run a long script in background:

     ```bash
     python3 long_script.py &  # Start script
     jobs                      # Check job [1]
     ```

  2. Suspend and manage a process:

     ```bash
     vim report.txt     # Start vim
     Ctrl+Z             # Suspend
     jobs               # Shows [1]+ Stopped vim
     bg %1              # Resume in background
     fg %1              # Bring back to foreground
     ```

  3. Run multiple background jobs:

     ```bash
     sleep 100 &        # Job [1]
     tar czf backup.tar.gz /home &  # Job [2]
     jobs               # List all jobs
     ```

### Killing Processes

- **Signals**:
  - `SIGHUP` (1): Reload/restart daemon.
  - `SIGINT` (2): Interrupt (`Ctrl+C`).
  - `SIGQUIT` (3): Quit with core dump.
  - `SIGKILL` (9): Force terminate.
  - `SIGTERM` (15): Graceful terminate (default).
  - `SIGCONT` (18): Resume stopped process.
  - `SIGSTOP` (19): Pause (unblockable).
  - `SIGTSTP` (20): Stop (`Ctrl+Z`).
  - **Actions**: Term (end), Core (end with dump), Stop (pause).
  - List signals: `kill -l`.

- **Commands**:
  - `kill <pid>`: Send `SIGTERM`.
  - `kill -<signal> <pid>`: Specific signal.
  - `pkill <name>`: Kill by name.
  - `killall <name>`: Kill all instances by name.

- **Examples**:
  1. Terminate a hung `firefox`:

     ```bash
     pgrep firefox      # Find PID (e.g., 5678)
     kill 5678          # Try SIGTERM
     kill -9 5678       # Force with SIGKILL if needed
     ```

  2. Reload `sshd`:

     ```bash
     pkill -HUP sshd    # Send SIGHUP to reload
     ```

  3. Stop and resume a process:

     ```bash
     pgrep sleep        # Find PID (e.g., 9012)
     kill -STOP 9012    # Pause
     kill -CONT 9012    # Resume
     ```

  4. Kill all `httpd` instances:

     ```bash
     killall httpd      # Terminate all httpd processes
     ```

### Logging Users Out Administratively

- **Purpose**: End user sessions for security or resource cleanup.
- **Commands**:
  - `pgrep -l -u <user>`: List user processes.
  - `w -h -u <user>`: Show user sessions.
  - `pstree -p <user>`: Show process tree.
  - `pkill -u <user>`: Terminate user processes.

- **Examples**:
  1. Log out user `bob`:

     ```bash
     w -h -u bob        # Check bob’s sessions
     pgrep -l -u bob    # List bob’s processes (e.g., bash, vim)
     pkill -u bob       # Terminate all bob’s processes
     ```

  2. Investigate user `alice`’s processes:

     ```bash
     pstree -p alice    # Show process hierarchy
     pgrep -l -u alice  # List PIDs and names
     ```

  3. Force logout with signal:

     ```bash
     pkill -9 -u bob    # Force terminate bob’s processes
     ```

### Monitoring Process Activity

- **Tools**:
  - `ps`: Static snapshot.
  - `top`: Real-time monitoring.
  - `htop`: Enhanced viewer (install: `sudo dnf install htop`).
- **Real-Time with `top`**:
  - Shows PID, USER, PR, NI, %CPU, %MEM, TIME, COMMAND.
  - **Keystrokes**:
    - `q`: Quit.
    - `k`: Kill (enter PID, signal).
    - `r`: Renice (enter PID, nice value).
    - `P`: Sort by CPU.
    - `M`: Sort by memory.
    - `f`: Manage fields.
    - `h` or `?`: Help.

- **Examples**:
  1. Monitor CPU usage:

     ```bash
     top                # Press P to sort by CPU
     ```

  2. Track memory hogs:

     ```bash
     top                # Press M to sort by memory
     ```

  3. Kill a process in `top`:

     ```bash
     top                # Press k, enter PID (e.g., 1234), choose signal (e.g., 15)
     ```

  4. Watch high CPU processes:

     ```bash
     watch -n 2 'ps aux --sort %cpu | head -5'  # Update every 2 seconds
     ```

  5. Check open files for a process:

     ```bash
     lsof -p 1234       # List files opened by PID 1234
     ```

### Describing Load Average

- **Definition**: Average number of processes in run queue (running or waiting) over 1, 5, and 15 minutes.
- **Commands**:
  - `uptime`: Show load average.
  - `lscpu`: Show CPU count.

- **Interpreting Load Average**:
  - Output: `0.50, 0.75, 1.00` (1-min, 5-min, 15-min).
  - Compare to CPU cores (`lscpu`):
    - Load < cores: Underutilized.
    - Load ≈ cores: Fully utilized.
    - Load > cores: Overloaded (e.g., 4.0 on 2 CPUs means contention).
  - High load may indicate CPU, memory, or I/O bottlenecks.

- **Examples**:
  1. Check load average:

     ```bash
     uptime             # Shows: load average: 0.30, 0.40, 0.50
     ```

  2. Verify CPU count:

     ```bash
     lscpu | grep '^CPU(s)'  # Shows 4 CPUs
     ```

  3. Diagnose high load:

     ```bash
     uptime             # Load: 8.0, 7.5, 7.0 (overloaded on 4 CPUs)
     top                # Identify high CPU processes
     ```

  4. Log load average:

     ```bash
     uptime >> /var/log/load_monitor.log
     ```

### Practical Examples

1. **Identify and Terminate Resource-Hogging Process**:

   ```bash
   top                        # Find PID 5678 using 80% CPU
   kill 5678                  # Try SIGTERM
   kill -9 5678               # Force if needed
   ps aux | grep 5678         # Verify termination
   ```

2. **Manage a Backup Job**:

   ```bash
   tar czf /backup/data.tar.gz /home &  # Start backup
   jobs                                 # Check job [1]
   kill %1                              # Stop if needed
   ```

3. **Handle Zombie Process**:

   ```bash
   ps aux | grep ' Z '        # Find zombie (e.g., PID 7890)
   ps -ef | grep 7890         # Find parent (e.g., PID 1234)
   kill 1234                  # Terminate parent
   ```

4. **Adjust Priority for a Batch Job**:

   ```bash
   nice -n 15 python3 batch_job.py &  # Run with low priority
   pgrep python3                      # Find PID (e.g., 2345)
   renice 10 -p 2345                  # Adjust priority
   top                                # Check NI column
   ```

5. **Monitor and Log High Memory Usage**:

   ```bash
   ps aux --sort -%mem | head -5      # Top 5 memory users
   watch -n 5 'ps aux --sort -%mem'   # Monitor memory usage
   ```

### Common Pitfalls

- **Overusing `SIGKILL`**: `kill -9` risks data loss; try `kill` first.
- **Zombies**: Unreaped zombies waste resources; kill parent process.
- **Load Misinterpretation**: High load on multi-core systems may be normal; check `lscpu`.
- **Forgotten Jobs**: Track with `jobs` to avoid orphaned processes.
- **Syntax Errors**: `ps -aux` may fail; use `ps aux` or `ps -u <user>`.
- **Missing `htop`**: Install with `sudo dnf install htop`.

### Best Practices

- **Use `top` or `htop`**: Sort with `P` (CPU) or `M` (memory).
- **Prefer `SIGTERM`**: Allows cleanup before termination.
- **Monitor Load**: Use `uptime` and compare with `lscpu`.
- **Automate Monitoring**:

  ```bash
  watch -n 5 'ps aux --sort %cpu | head -5' > /tmp/high_cpu.log
  ```

- **Check Zombies**: `ps aux | grep Z`.
- **Use `pstree`**: Visualize process trees for debugging.
- **RHEL 10**: Use `rhel lightspeed "monitor processes"` for command suggestions.

### Revision Quiz/Exercises

- **Questions**:
  1. How do `ps aux` and `ps -ef` differ? (`aux`: BSD, user-oriented; `-ef`: UNIX, shows PPID.)
  2. How to log out user `alice`? (`pkill -u alice`)
  3. What does load average 3.0 mean on a 2-CPU system? (Overloaded; processes waiting.)
  4. How to pause a process? (`kill -STOP <pid>` or `Ctrl+Z`.)
- **Exercises**:
  1. Start and manage a job:

     ```bash
     sleep 1000 &       # Start job
     jobs               # Check
     kill %1            # Terminate
     ```

  2. Monitor and adjust priority:

     ```bash
     top                # Find high CPU PID (e.g., 1234)
     renice 5 -p 1234   # Lower priority
     ```

  3. Log out a user:

     ```bash
     pgrep -l -u bob    # List processes
     pkill -u bob       # Log out
     ```
