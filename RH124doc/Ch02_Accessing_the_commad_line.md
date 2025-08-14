# Chapter 2: Access the Command Line

## Accessing the Command Line

### Introduction to the Bash Shell

The Bash (Bourne Again SHell) is the default command-line interpreter in Red Hat Enterprise Linux (RHEL). It serves as an interface between the user and the Linux kernel, interpreting commands and executing them. Bash is scriptable, extensible, and supports features like tab completion, command history, and scripting for automation.

### Shell Basics

- The shell processes commands typed in a terminal, enabling interaction with the operating system.
- Commands follow a structure: `command [options] [arguments]` (e.g., `ls -l /home`).
- Key features include piping (`|`), redirection (`>`, `<`), and environment variables (e.g., `$PATH`, `$USER`).
- The prompt (e.g., `[user@host ~]$`) indicates the user, hostname, and current directory.

### Logging in to a Local System

- **Physical Console**: Log in directly via the system’s keyboard and monitor. After booting, a login prompt appears (e.g., `server login:`). Enter username and password.
- **Virtual Consoles**: Access additional text-based consoles with `Ctrl+Alt+F2` through `F6` (F1 is typically the GUI). Log in with credentials.
- **GUI Login**: On systems with a graphical interface, log in via the GNOME desktop login screen.

## Logging in over the Network

- **SSH (Secure Shell)**: Use `ssh user@hostname` for secure remote access (e.g., `ssh student@server1.example.com`).
  - Requires SSH server (`sshd`) running on the target system.
  - Password or SSH key authentication is needed.
- Ensure network connectivity and correct hostname/IP address.

### Logging Out

- **Text Console**: Type `exit` or press `Ctrl+D` to log out of a shell session.
- **GUI**: Click the user menu in GNOME (top-right corner) and select "Log Out."
- **Remote SSH**: Use `exit` to close the session and return to the local shell.

## Accessing the Command Line Using the Desktop

### Introduction to GNOME Desktop Environment

GNOME is the default graphical desktop environment in RHEL, providing an intuitive interface with:

- **Activities Overview**: Accessed via the "Activities" button or `Super` (Windows) key, showing open applications and a search bar.
- **Top Bar**: Displays system status, user menu, and notifications.
- **Dash**: A dock-like interface for launching favorite or pinned applications.

### Workspaces

- GNOME supports multiple virtual desktops (workspaces) for organizing tasks.
- Access: Click "Activities" or press `Super+Tab`, then scroll or use `Ctrl+Alt+Up/Down` to switch between workspaces.
- Drag windows between workspaces in the Activities Overview to manage multiple tasks efficiently.

### Starting a Terminal

- **From GNOME**:
  - Click "Activities," type `terminal` in the search bar, and press `Enter` to open GNOME Terminal.
  - Alternatively, use the keyboard shortcut `Ctrl+Alt+T` (if configured).
- **Virtual Consoles**: Switch to a text-based terminal with `Ctrl+Alt+F2` through `F6`.
- GNOME Terminal supports multiple tabs (`Ctrl+Shift+T`) and profiles for customization.

### Powering Off or Rebooting the System

- **GUI**: Click the user menu (top-right corner), select "Power Off / Log Out," then choose "Power Off" or "Restart."
- **Command Line**:

  ```bash
  sudo poweroff  # Shut down the system
  sudo reboot    # Restart the system
  ```

- Requires root privileges or appropriate permissions via `sudo`.

## Executing Commands Using the Bash Shell

### Basic Command Syntax

- Commands follow the format: `command [options] [arguments]`.
  - **Options**: Modify behavior, using short (`-l`) or long (`--long`) forms (e.g., `ls -la` or `ls --all --long`).
  - **Arguments**: Specify targets (e.g., `cp file1 file2` copies `file1` to `file2`).
- Commands are case-sensitive and executed when `Enter` is pressed.

### Example of Simple Commands

- **Navigation and Information**:

  ```bash
  pwd          # Print current directory (e.g., /home/user)
  whoami       # Display current username (e.g., student)
  hostname     # Show system hostname (e.g., server1.example.com)
  date         # Display current date and time
  ```

- **File Listing**:

  ```bash
  ls           # List files in current directory
  ls -l        # Detailed listing (permissions, owner, size)
  ls -a        # Show hidden files (starting with .)
  ls -lh       # Human-readable file sizes (e.g., 4K instead of 4096)
  ```

### Viewing the Contents of Files

- **Common Commands**:

  ```bash
  cat file.txt     # Display entire file content
  less file.txt    # View file interactively (navigate with arrows, q to quit)
  head file.txt    # Show first 10 lines (use -n 5 for 5 lines)
  tail file.txt    # Show last 10 lines (use -n 5 for 5 lines)
  ```

- Example: `cat /etc/redhat-release` to check RHEL version.

### Tab Completion

- Press `Tab` to auto-complete commands, filenames, or paths.
- Double-tap `Tab` to list possible completions (e.g., type `ls /et` and press `Tab` twice to see `/etc`).
- Saves time and reduces typing errors.

### Continuing Long Commands on Another Line

- Use a backslash (`\`) at the end of a line to continue a command:

  ```bash
  ls -l /var/log \
  /etc
  ```

- The shell waits for the next line to complete the command.

### Command History

- View previous commands with:

  ```bash
  history  # List all commands in history
  ```

- Reuse commands:
  - `!n`: Run command number `n` from history (e.g., `!5`).
  - `!!`: Rerun the last command.
  - `Up Arrow`: Scroll through previous commands.
- Edit `~/.bash_history` to persist history across sessions.

### Editing the Command Line

- **Key Shortcuts**:
| Shortcut | Description                                      |
|----------|--------------------------------------------------|
| Ctrl+A   | Move cursor to start of line                     |
| Ctrl+E   | Move cursor to end of line                       |
| Ctrl+U   | Clear from cursor to start of line               |
| Ctrl+K   | Clear from cursor to end of line                 |
| Ctrl+C   | Interrupt current command                        |
| Ctrl+Z   | Suspend current command (resume with `fg`)       |

- Example: Type `ls /etc`, press `Ctrl+A` to jump to the start, then add `sudo` to make `sudo ls /etc`.

### Practical Examples

- **Scenario: Quick System Check**:

  ```bash
  whoami && hostname && pwd  # Chain commands to display user, host, and directory
  ```

- **Daily Use: View Log Files**:

  ```bash
  less /var/log/messages  # Browse system logs interactively
  ```

- **Common Pitfalls**:
  - Case sensitivity in commands/paths (e.g., `LS` fails; use `ls`).
  - Missing permissions: Use `sudo !!` to rerun a failed command with elevated privileges.
