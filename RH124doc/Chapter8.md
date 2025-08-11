# RH124 Revision Notebook: Chapter 8 - Monitor and Manage Linux Processes

This notebook provides a detailed summary of Chapter 8 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on monitoring and managing Linux processes, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 8: Monitor and Manage Linux Processes

### Key Concepts

- **Processes in Linux**:
  - A process is a running instance of a program, identified by a unique Process ID (PID).
  - Types: User processes (e.g., `vim`), system processes (e.g., `systemd`), and daemons (background services like `sshd`).
  - Attributes: PID, parent process ID (PPID), user, priority, memory usage, and state (e.g., running, sleeping, zombie).
- **Process States**:
  - **Running (R)**: Actively executing.
  - **Sleeping (S)**: Waiting for an event (interruptible).
  - **Uninterruptible Sleep (D)**: Waiting for I/O (e.g., disk access).
  - **Zombie (Z)**: Terminated but not yet reaped by parent.
  - **Stopped (T)**: Paused (e.g., by `Ctrl+Z`).
- **Process Monitoring**:
  - Tools like `ps`, `top`, and `htop` display process details (PID, CPU/memory usage, user).
  - `/proc` directory contains runtime process information (e.g., `/proc/1234` for PID 1234).
- **Process Management**:
  - Control processes by sending signals (e.g., terminate, suspend, resume).
  - Adjust process priority to manage CPU allocation.
- **Foreground vs. Background**:
  - Foreground processes: Interact with the terminal (e.g., `vim`).
  - Background processes: Run without terminal input (e.g., `command &`).
- **RHEL 9/10 Notes**:
  - Improved `systemd` integration for process management.
  - RHEL 10’s Lightspeed can suggest process-related commands (e.g., `rhel lightspeed "find high CPU processes"` suggests `top` or `ps aux --sort %cpu`).

### Important Commands

- **Monitoring Processes**:

  ```bash
  ps  # Show processes for current shell
  ps aux  # Show all processes (a: all users, u: user-oriented, x: no terminal)
  ps -ef  # Show all processes in full format
  top  # Interactive process viewer (q to quit, k to kill, r to renice)
  htop  # Enhanced interactive viewer (if installed)
  ```

- **Filtering Processes**:

  ```bash
  ps -u user1  # Show processes for user1
  ps -C sshd  # Show processes by command name (sshd)
  pgrep sshd  # List PIDs for sshd
  ```

- **Managing Processes**:

  ```bash
  kill 1234  # Terminate process with PID 1234 (SIGTERM)
  kill -9 1234  # Force kill (SIGKILL, use cautiously)
  pkill sshd  # Kill processes by name
  killall httpd  # Kill all httpd processes
  ```

- **Background and Foreground**:

  ```bash
  sleep 100 &  # Run sleep in background
  jobs  # List background jobs
  fg %1  # Bring job 1 to foreground
  bg %1  # Resume job 1 in background
  Ctrl+Z  # Suspend foreground process
  ```

- **Adjusting Priority**:

  ```bash
  nice -n 10 command  # Run command with lower priority (+10)
  renice 5 1234  # Set priority of PID 1234 to 5
  top  # Press r, enter PID, set priority
  ```

- **Process Information**:

  ```bash
  cat /proc/1234/stat  # View process details (e.g., state, memory)
  lsof -p 1234  # List open files for PID 1234
  ```

### Practical Examples

- **Scenario: Monitor System Load**:
  - Check running processes: `top` (sort by CPU with `P`, memory with `M`).
  - Identify high CPU usage: `ps aux --sort %cpu | head`.
  - Output: Shows top processes (e.g., `httpd` using 20% CPU).
- **Scenario: Terminate a Hung Process**:
  - Find PID of a frozen `firefox`: `pgrep firefox` (e.g., PID 5678).
  - Terminate: `kill 5678`. If it persists: `kill -9 5678`.
  - Verify: `pgrep firefox` (no output if killed).
- **Scenario: Run a Long Task in Background**:
  - Start a backup: `tar czf /backup/data.tar.gz /home &`.
  - Check status: `jobs`.
  - Bring to foreground if needed: `fg %1`.
- **Scenario: Adjust Priority for a Batch Job**:
  - Run a low-priority task: `nice -n 15 python script.py`.
  - Change priority of running process (PID 2345): `renice 10 2345`.
  - Verify: `top` (check `NI` column for nice value).
- **Daily Use: Find Resource-Hogging Processes**:
  - List top memory users: `ps aux --sort -%mem | head -n 5`.
  - Output: Shows processes like `java` consuming 1GB RAM.
- **Scenario: Troubleshoot Zombie Processes**:
  - Find zombies: `ps aux | grep ' Z '`.
  - Kill parent process (e.g., PID 1234): `kill 1234`.

### Common Pitfalls

- **Using SIGKILL (-9)**: Force-killing with `kill -9` can leave resources unreleased. Try `kill` (SIGTERM) first.
- **Ignoring Background Jobs**: Forgetting jobs (`jobs`) can lead to orphaned processes. Use `fg` or `kill %1`.
- **Incorrect PID**: Killing the wrong PID can disrupt services. Verify with `ps -C command` or `pgrep`.
- **High Priority Misuse**: Setting `nice -n -20` requires root and can starve other processes. Use positive values (e.g., 10) for non-critical tasks.
- **Missing `htop`**: `htop` isn’t installed by default; install with `sudo dnf install htop`.
- **Zombie Processes**: Zombies indicate a parent process issue. Find parent with `ps -ef | grep <zombie_pid>` and terminate it.

### Best Practices and Tips

- **Use `top` or `htop` for Real-Time Monitoring**: Press `f` in `htop` to customize fields (e.g., show CPU, memory).
- **Start with SIGTERM**: Always try `kill` before `kill -9` to allow graceful termination.
- **Automate Monitoring**: Script checks for high CPU usage:

  ```bash
  ps aux --sort %cpu | head -n 5 > /tmp/high_cpu.log
  ```

- **Manage Background Jobs**: Use `&` for long-running tasks and `jobs` to track them.
- **Check `/proc` for Details**: Explore `/proc/<pid>` for memory, status, or open files.
- **RHEL 10 Tip**: Use Lightspeed for process queries (e.g., `rhel lightspeed "kill a hung process"` suggests `kill` or `pkill`).
- **Clean Up Zombies**: Regularly check for zombies with `ps aux | grep Z` to prevent resource leaks.

### Revision Quiz/Notes

- **Questions**:
  - What command shows all processes for user `jdoe`? (`ps -u jdoe`)
  - What’s the difference between `kill` and `kill -9`? (`kill` sends SIGTERM for graceful exit; `-9` sends SIGKILL for immediate termination.)
  - How do you lower a process’s priority? (`nice -n 10 command` or `renice 10 <pid>`)
- **Quick Exercise**:
  - Start a background process, check its status, and terminate it:

    ```bash
    sleep 1000 &
    jobs
    pgrep sleep
    pkill sleep
    ```

- **Self-Test**:
  - Explain the `ps aux` columns (USER, PID, %CPU, %MEM, STAT, COMMAND).
  - Why avoid `kill -9`? (Risks resource leaks or data corruption.)

---
