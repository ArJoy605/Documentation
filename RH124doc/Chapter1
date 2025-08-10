# RH124 Revision Notebook: Red Hat System Administration I

This notebook summarizes key concepts, commands, and tips from the RH124 course for quick revision and daily use. It's based on Red Hat Enterprise Linux (RHEL) 9/10. I've expanded it with more detailed explanations, additional commands, practical examples, common pitfalls, best practices, and revision exercises to make it more comprehensive for revision and real-world application. Focus on practical application. I'll expand this notebook chapter by chapter as needed.

## Chapter 1: Get Started with Red Hat Enterprise Linux

### Key Concepts (Expanded)

- **Linux Overview**: Linux is a free, open-source operating system kernel developed by Linus Torvalds in 1991, inspired by UNIX. It's modular, multi-user, and multitasking, making it ideal for servers, desktops, and cloud environments. Key advantages include stability, security (through community audits), and customizability.
- **Open-Source Principles**:
  - Governed by licenses like GNU General Public License (GPL), allowing users to view, modify, and redistribute source code.
  - Community collaboration via projects like kernel.org or GitHub.
  - Benefits for enterprises: Lower costs, rapid bug fixes, and vendor independence; drawbacks: Potential lack of official support without commercial backing.
- **Distributions (Distros)**: Linux kernel bundled with tools, libraries, and applications. Popular ones include Ubuntu (user-friendly), Debian (stable), and enterprise-focused like RHEL.
- **RHEL Ecosystem**:
  - Red Hat Enterprise Linux: Commercial distro with 10-year support lifecycle, certifications (e.g., FIPS for security), and integration with tools like Ansible for automation.
  - Free upstream: Fedora (testing ground for RHEL features); CentOS Stream (development branch for RHEL).
  - Key components: Linux kernel, GNU Core Utilities (e.g., ls, cp), systemd (for service management), SELinux (enhanced security), and GNOME (default desktop).
  - RHEL 9/10 Updates: RHEL 10 introduces AI-assisted tools like RHEL Lightspeed for natural language queries in administration, improved container support with Podman, and enhanced hardware compatibility.
- **Installation Basics**:
  - Use Anaconda installer (GUI or text mode) from ISO images downloaded from Red Hat Customer Portal.
  - Steps: Boot from media, select language/timezone, configure storage (e.g., LVM for flexible partitioning), set root password, create users, and install packages.
  - Post-install: Register system for updates, enable repositories, and configure firewall/network.
  - Minimum Requirements: 2 CPU cores, 2 GB RAM (4 GB recommended), 20 GB disk space for server installs.

### Important Commands and Tools (With Examples)

- **System Information**:

  ```bash
  uname -a  # Displays kernel version, architecture, and hostname (e.g., Linux server1 5.14.0-70.el9.x86_64 ...)
  cat /etc/redhat-release  # Shows RHEL version (e.g., Red Hat Enterprise Linux release 9.0 (Plow))
  lsb_release -a  # Detailed distro info (requires redhat-lsb-core package)
  hostnamectl  # View or set hostname and system details
  ```

- **Update System**:

  ```bash
  sudo dnf check-update  # List available updates
  sudo dnf update  # Apply updates (use --security for security patches only)
  ```

- **Registration**:

  ```bash
  sudo subscription-manager register --username=user --password=pass  # Register with Red Hat
  sudo subscription-manager attach --auto  # Attach subscription pool
  ```

### Practical Examples

- **Scenario: Verify System After Install**: Run `uname -a` and `cat /etc/redhat-release` to confirm version and architecture before deploying applications.
- **Daily Use: Keeping System Updated**: Schedule cron jobs for `dnf update -y` to automate patches, reducing vulnerability exposure.
- **Common Pitfalls**: Forgetting to register the system leads to no updates; always check with `subscription-manager status`.

### Best Practices and Tips

- Use virtual machines (e.g., VirtualBox or KVM) for testing installations to avoid hardware risks.
- Enable automatic security updates via `dnf-automatic`.
- Explore Red Hat Documentation: Search Knowledgebase on access.redhat.com for troubleshooting.
- For RHEL 10: Leverage Lightspeed with `rhel lightspeed` for AI-guided commands (e.g., "how to update packages?").

### Revision Quiz/Notes

- What is the primary license for the Linux kernel? (GPLv2)
- Differentiate RHEL from Fedora: RHEL is stable/enterprise; Fedora is cutting-edge/community.
- Quick Exercise: Write a command to check if your system is registered (`subscription-manager status`).
- Self-Test: Explain why open-source is beneficial for security (peer review finds vulnerabilities faster).

---
