# Chapter 4: Get Help in Red Hat Enterprise Linux

## Key Concepts

- **Importance of Help Systems**: RHEL provides multiple built-in and online resources to troubleshoot issues, learn commands, and understand system configurations, essential for efficient administration.
- **Types of Help Resources**:
  - **Local Documentation**: Available on the system via manual pages (`man`), info pages (`info`), and package documentation (e.g., `/usr/share/doc`).
  - **Online Resources**: Red Hat Customer Portal, Knowledgebase, and community forums for detailed guides and support.
  - **Command-Line Help**: Quick help via `--help` flags or built-in tools.
- **Manual Pages (`man`)**:
  - Structured documentation for commands, configuration files, and system calls.
  - Organized into sections (e.g., 1 for user commands, 5 for file formats).
  - Accessed via `man command` or `man section command` (e.g., `man 5 passwd`).
- **Info Pages (`info`)**:
  - More detailed, hyperlinked documentation compared to `man`.
  - Useful for GNU tools (e.g., `info coreutils`).

### Important Commands

- **Manual Pages**:

  ```bash
  man ls  # View manual for ls command
  man 5 passwd  # View man page for /etc/passwd file format
  man -k keyword  # Search man pages (same as apropos keyword)
  man man  # View man page about man itself
  ```

  - Navigation: `/pattern` to search, `n` for next match, `q` to quit.
- **Info Pages**:

  ```bash
  info ls  # View info page for ls
  info coreutils  # View info for coreutils package
  ```

  - Navigation: Arrow keys, `n` (next node), `p` (previous), `u` (up), `q` (quit), `Tab` to jump between links.
- **Quick Help**:

  ```bash
  ls --help  # Display brief usage and options for ls
  help cd  # Show help for Bash built-in commands
  ```

- **Search for Commands**:

  ```bash
  apropos file  # List man pages related to "file"
  whatis ls  # Brief description of ls command
  ```

- **Package Documentation**:

  ```bash
  ls /usr/share/doc  # List available package docs
  cat /usr/share/doc/bash/README  # View README for bash
  ```

- **Red Hat Subscription Info**:

  ```bash
  subscription-manager status  # Check subscription status
  subscription-manager repos --list  # List available repositories
  ```

### Practical Examples

- **Scenario: Learn a Command’s Options**:
  - Find options for `cp`: `man cp` or `cp --help`.
  - Search for "recursive" in man page: `/recursive`, press `n` to find matches.
  - Example: Discover `-r` for recursive copying.
- **Scenario: Understand a Config File**:
  - Check `/etc/passwd` format: `man 5 passwd`.
  - Output explains fields (e.g., username:password:UID:GID:comment:home:shell).
- **Scenario: Find a Command for a Task**:
  - Need a command to copy files: `apropos copy`.
  - Output lists `cp`, `scp`, etc. Then use `man cp` for details.
- **Scenario: Explore Package Docs**:
  - Check documentation for `httpd` (Apache):

    ```bash
    ls /usr/share/doc/httpd
    cat /usr/share/doc/httpd/CHANGES
    ```

  - Find example configs or version changes.
- **Scenario: Troubleshoot with Red Hat Portal**:
  - Issue: Failed to update packages.
  - Check subscription: `subscription-manager status`.
  - Search Knowledgebase: Visit [access.redhat.com](https://access.redhat.com), search "subscription-manager failed", follow suggested fixes (e.g., `subscription-manager refresh`).
- **Daily Use: Quick Reference**:
  - Create a cheat sheet: `echo "man ls: command help" > ~/cheatsheet.txt; echo "apropos: search commands" >> ~/cheatsheet.txt`.
  - View: `cat ~/cheatsheet.txt`.

### Common Pitfalls

- **Wrong Man Section**: `man passwd` (section 1, command) vs. `man 5 passwd` (file format). Always specify section if needed.
- **Missing Man Pages**: If `man` returns "No manual entry", ensure package is installed (e.g., `sudo dnf install man-pages`) or check online.
- **Info Navigation**: `info` can be complex due to hyperlinks. Practice with `info info` to learn navigation.
- **Ignoring `--help`**: Some commands lack `man` pages but have `--help` (e.g., `systemctl --help`).
- **Subscription Issues**: Online Red Hat resources require an active subscription. Verify with `subscription-manager status`.
- **Overlooking `/usr/share/doc`**: Contains valuable READMEs and examples, often missed by beginners.
