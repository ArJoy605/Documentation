# RH124 Revision Notebook: Chapter 14 - Install and Update Software Packages

This notebook provides a detailed summary of Chapter 14 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on installing, updating, and managing software packages using the DNF package manager, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 14: Install and Update Software Packages

### Key Concepts
- **Package Management in RHEL**:
  - Software is distributed as **packages** (`.rpm` files) containing binaries, libraries, configuration files, and documentation.
  - Managed by **DNF** (Dandified Yum), the default package manager in RHEL 9/10, replacing Yum.
  - Packages are sourced from **repositories**, which are collections of RPMs (e.g., Red Hat’s official repos, EPEL).
- **DNF Features**:
  - Installs, updates, removes, and queries packages.
  - Resolves dependencies automatically.
  - Supports groups for installing related packages (e.g., "Web Server").
- **Repositories**:
  - Configured in `/etc/yum.repos.d/` (`.repo` files).
  - Key RHEL repos: **BaseOS** (core OS) and **AppStream** (additional apps, modules).
  - Requires active Red Hat subscription for access (`subscription-manager`).
- **RPM (Red Hat Package Manager)**:
  - Low-level tool for direct RPM file management (less common than DNF).
  - Used for querying or installing local `.rpm` files.
- **RHEL 9/10 Notes**:
  - RHEL 10 enhances DNF with faster dependency resolution and modular content (AppStream).
  - RHEL Lightspeed (RHEL 10) can suggest package management commands (e.g., `rhel lightspeed "install web server"` suggests `dnf install httpd`).

### Important Commands
- **Managing Repositories**:
  ```
  subscription-manager repos --list  # List available repositories
  subscription-manager repos --enable rhel-9-for-x86_64-appstream-rpms  # Enable a repo
  dnf repolist  # Show enabled repositories
  dnf repoinfo  # Detailed repository info
  ```
- **Installing Packages**:
  ```
  dnf install httpd  # Install httpd package
  dnf install -y vim  # Install without prompting
  dnf groupinstall "Web Server"  # Install package group
  ```
- **Updating Packages**:
  ```
  dnf check-update  # List available updates
  dnf update  # Update all packages
  dnf update --security  # Apply security updates only
  dnf upgrade  # Same as update (full system upgrade)
  ```
- **Removing Packages**:
  ```
  dnf remove httpd  # Remove httpd package
  dnf autoremove  # Remove unneeded dependencies
  ```
- **Querying Packages**:
  ```
  dnf list installed  # List installed packages
  dnf list available  # List available packages
  dnf search vim  # Search for packages by keyword
  dnf info httpd  # Show package details
  dnf provides /usr/bin/vim  # Find package providing a file
  ```
- **RPM Commands (Low-Level)**:
  ```
  rpm -ivh package.rpm  # Install local RPM file
  rpm -qa  # List all installed packages
  rpm -qf /usr/bin/ls  # Find package owning a file
  rpm -qi httpd  # Show package info
  ```
- **Managing Modules (AppStream)**:
  ```
  dnf module list  # List available modules
  dnf module install nodejs:18  # Install Node.js 18 stream
  dnf module reset nodejs  # Reset module to default
  ```

### Practical Examples
- **Scenario: Install a Web Server**:
  - Install Apache:
    ```
    sudo dnf install -y httpd
    sudo systemctl enable --now httpd
    ```
  - Verify: `systemctl status httpd` and `curl http://localhost`.
- **Scenario: Update System**:
  - Check for updates:
    ```
    dnf check-update
    ```
  - Apply all updates:
    ```
    sudo dnf update -y
    ```
  - Reboot if kernel updated: `sudo reboot`.
- **Scenario: Enable a Repository**:
  - Enable AppStream for additional packages:
    ```
    sudo subscription-manager repos --enable rhel-9-for-x86_64-appstream-rpms
    dnf repolist
    ```
- **Scenario: Install a Package Group**:
  - Install "Development Tools":
    ```
    sudo dnf groupinstall "Development Tools"
    dnf groupinfo "Development Tools"  # Verify installed packages
    ```
- **Daily Use: Find a Command’s Package**:
  - Identify package providing `dig`:
    ```
    dnf provides /usr/bin/dig
    ```
    Output: Shows `bind-utils`.
  - Install: `sudo dnf install bind-utils`.
- **Scenario: Install Local RPM**:
  - Install a downloaded RPM:
    ```
    sudo rpm -ivh package.rpm
    ```
  - Or use DNF for dependency resolution:
    ```
    sudo dnf install package.rpm
    ```

### Common Pitfalls
- **No Subscription**: Without an active Red Hat subscription, DNF cannot access repos. Verify: `subscription-manager status`.
- **Repo Misconfiguration**: Missing or disabled repos cause "No package available" errors. Check: `dnf repolist`.
- **Dependency Conflicts**: Rare, but can occur with third-party repos. Use `dnf resolve` or remove conflicting packages.
- **Using `rpm` Instead of DNF**: `rpm` doesn’t handle dependencies, leading to errors. Prefer `dnf install` for local RPMs.
- **Forgetting `-y`**: DNF prompts for confirmation; use `-y` for scripts to avoid hangs.
- **Outdated Cache**: Stale DNF cache can list old packages. Refresh: `sudo dnf clean all; sudo dnf makecache`.

### Best Practices and Tips
- **Keep System Updated**: Regularly run `sudo dnf update --security` for security patches.
- **Enable Necessary Repos**: Use `subscription-manager` to enable BaseOS and AppStream for full package access.
- **Use Groups for Consistency**: Install related tools with `dnf groupinstall` (e.g., "Web Server").
- **Verify Packages**: Check package details before installing: `dnf info package`.
- **Automate Updates**: Schedule updates with cron:
  ```
  # /etc/cron.daily/update.sh
  #!/bin/bash
  dnf update -y --security
  ```
  Run: `chmod +x /etc/cron.daily/update.sh`.
- **RHEL 10 Tip**: Use Lightspeed for package tasks (e.g., `rhel lightspeed "install SSH server"` suggests `dnf install openssh-server`).
- **Clean Up**: Remove unneeded packages: `dnf autoremove` and clear cache: `dnf clean all`.

### Revision Quiz/Notes
- **Questions**:
  - What command lists installed packages? (`dnf list installed` or `rpm -qa`)
  - How do you install a specific module stream? (`dnf module install nodejs:18`)
  - What’s the difference between `dnf update` and `dnf upgrade`? (Functionally identical in RHEL.)
- **Quick Exercise**:
  - Install `tree`, enable it as a service if applicable, and verify:
    ```
    sudo dnf install -y tree
    tree /etc | head -n 10
    ```
- **Self-Test**:
  - Explain the role of `/etc/yum.repos.d/` (stores repository configurations).
  - Why use `dnf` over `rpm`? (DNF resolves dependencies automatically.)

---

This detailed note on Chapter 14 equips you for managing software packages in RHEL. Practice these commands in a RHEL virtual machine (e.g., via KVM or VirtualBox) to build proficiency. If you want to add other chapters (e.g., Chapter 15) to this notebook, combine with previous chapters, or need specific examples (e.g., scripting package management), let me know!