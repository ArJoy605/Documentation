# Chapter 2: Access the Command Line

## Key Concepts (Expanded)

- **Shell Basics**: The shell interprets commands and acts as an interface to the kernel. Bash (Bourne Again SHell) is default in RHEL—it's scriptable and extensible.
- **Terminal Access**:
  - Local: GNOME Terminal (search in Activities), or virtual consoles (Ctrl+Alt+F1-F6; F1 is GUI).
  - Remote: Use SSH for secure connections (e.g., `ssh user@host`).
- **Command Structure**: `command [options/flags] [arguments]`
  - Options: Short (-v) or long (--verbose); can be combined (e.g., `ls -la`).
  - Arguments: Positional inputs (e.g., `cp file1 file2`).
  - Piping/Redirection: `|` for output to another command; `>` for file output; `<` for input.
- **Navigation**:
  - File system hierarchy: Root (`/`), /home (users), /etc (config), /var (logs), /usr (applications).
  - Paths: Absolute (/etc/passwd) vs. relative (../dir/file); special: ~ (home), . (current), .. (parent).
- **Environment Variables**: Dynamic values like `$PATH` (command search paths), `$USER` (current user), `$PWD` (current directory). Set with `export VAR=value`.
- **RHEL 9/10 Notes**: Bash 5.x includes better history management and tab completion for modern tools.

### Important Commands (With Examples)

- **Basic Navigation**:

  ```bash
  pwd  # Print Working Directory (e.g., /home/user)
  cd /etc  # Change to absolute path
  cd ~  # Go to home directory
  cd -  # Return to previous directory
  ls -l /var/log  # List detailed (permissions, owner, size, date)
  ls -a  # Show hidden files (starting with .)
  ls -lh  # Human-readable sizes (e.g., 4K instead of 4096)
  ```

- **Running and Viewing**:

  ```bash
  echo "Hello, $USER"  # Print with variable expansion (e.g., Hello, student)
  whoami  # Current username
  hostname  # System hostname
  date  # Current date/time
  cal  # Calendar (cal -y for full year)
  ```

- **Help and Manuals**:

  ```bash
  man ls  # Manual page (navigate: /search, n/next, q/quit)
  ls --help  # Quick usage summary
  info bash  # Interactive info pages (more detailed than man)
  apropos keyword  # Search man pages (e.g., apropos directory)
  ```

- **Advanced Shell Features**:

  ```bash
  history  # View command history (e.g., !5 to rerun 5th command)
  alias ll='ls -l'  # Create shortcut (add to ~/.bashrc for persistence)
  export PATH=$PATH:/new/bin  # Add directory to PATH
  ```

### Practical Examples

- **Scenario: Quick System Check**: `whoami && hostname && pwd` (chains commands with && for success-only execution).
- **Daily Use: Scripting Basics**: Write a simple script: `echo '#!/bin/bash' > myscript.sh; echo 'echo Hello' >> myscript.sh; chmod +x myscript.sh; ./myscript.sh`.
- **Common Pitfalls**: Typos in paths (case-sensitive); running without sudo when needed—leads to "Permission denied". Use `sudo !!` to rerun last command with sudo.

### Best Practices and Tips

- Use tab completion to avoid errors and speed up typing.
- Customize prompt in ~/.bash_profile (e.g., PS1='[\u@\h \W]\$ ' for user@host cwd$).
- Secure remote access: Prefer SSH keys over passwords (generate with `ssh-keygen`).
- For efficiency: Learn keyboard shortcuts—Ctrl+C (interrupt), Ctrl+Z (suspend), fg (resume).

### Revision Quiz/Notes

- What does `cd ..` do? (Moves to parent directory)
- Explain `$PATH`: Directories searched for executables; view with `echo $PATH`.
- Quick Exercise: Create an alias for `ls -la` and test it.
- Self-Test: Differentiate man and info (man: concise; info: hyperlinked, detailed).

---

This updated version is more detailed with expanded explanations, additional examples, and practical sections. If you'd like even more depth (e.g., specific labs or RHEL 10 nuances), or to proceed to chapters 3 and 4, let me know! Practice in a RHEL VM for best retention.
