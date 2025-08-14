# Chapter 1: Get Started with Red Hat Enterprise Linux

## What is Linux?

Linux is a free, open-source operating system kernel first created by Linus Torvalds in 1991. Inspired by UNIX, it is designed to be modular, multi-user, and multitasking, making it suitable for a wide range of environments, including servers, desktops, cloud systems, and embedded devices. The kernel is the core component that manages hardware resources and enables software to run efficiently.

## Why Should I Learn Linux?

- **Career Opportunities**: Linux powers most servers, cloud infrastructure (e.g., AWS, Google Cloud), and supercomputers, making Linux skills highly sought after in IT roles like system administration, DevOps, and cybersecurity.
- **Flexibility**: Linux runs on diverse hardware, from Raspberry Pi to enterprise servers, offering versatility in deployments.
- **Cost-Effective**: Free to use and distribute, reducing software costs for individuals and organizations.
- **Community and Support**: A global community provides extensive documentation, forums, and enterprise support (e.g., Red Hat subscriptions).

## What Makes Linux Great?

- **Stability**: Linux systems are known for running reliably for years without needing reboots, ideal for critical infrastructure.
- **Security**: Open-source nature allows community audits to identify and fix vulnerabilities quickly. Features like SELinux add robust security controls.
- **Customizability**: Users can tailor Linux to specific needs, from minimal server setups to full-featured desktop environments.
- **Performance**: Efficient resource usage makes Linux ideal for high-performance computing and low-resource devices.
- **Scalability**: Scales seamlessly from small IoT devices to large data centers.

## What is Open Source Software?

Open-source software (OSS) is software with source code that is publicly available, allowing users to view, modify, and redistribute it under specific licenses. This collaborative model fosters innovation, rapid bug fixes, and transparency. Examples include the Linux kernel, Apache web server, and Firefox browser.

## Types of Open Source Licenses

Open-source licenses define how software can be used, modified, and shared. Common licenses include:

- **GNU General Public License (GPL)**: Ensures derivative works remain open-source (e.g., Linux kernel uses GPLv2).
- **MIT License**: Permissive, allowing broad use with minimal restrictions, as long as the license is included.
- **Apache License**: Permissive, with patent protection for contributors, used by projects like Apache HTTP Server.
- **BSD License**: Highly permissive, allowing use in proprietary software with few conditions.
- **Mozilla Public License (MPL)**: Balances open-source and proprietary use, requiring modified files to remain open-source.

## Who Develops Open Source Software?

Open-source software is developed by a diverse group:

- **Individual Contributors**: Volunteers who code, test, or document projects out of passion or to build skills.
- **Communities**: Global groups collaborating via platforms like GitHub or kernel.org.
- **Companies**: Firms like Red Hat, Google, and IBM contribute code to projects they rely on (e.g., Red Hat’s contributions to the Linux kernel).
- **Non-Profits**: Organizations like the Linux Foundation support development through funding and coordination.

## Who is Red Hat?

Red Hat is a leading open-source software company founded in 1993, known for:

- Developing and supporting Red Hat Enterprise Linux (RHEL), a commercial Linux distribution for enterprises.
- Providing subscription-based support, training, and certifications.
- Contributing to open-source projects like the Linux kernel, GNOME, and Kubernetes.
- Offering tools like Ansible (automation), OpenShift (container orchestration), and Podman (container management).
- Acquired by IBM in 2019, Red Hat remains a key player in enterprise open-source solutions.

## What is a Linux Distribution?

A Linux distribution (distro) is a complete operating system built around the Linux kernel, bundled with tools, libraries, applications, and a package manager. Distros cater to different use cases:

- **User-Friendly**: Ubuntu, Linux Mint (desktop-focused).
- **Stable**: Debian, RHEL (enterprise-focused).
- **Cutting-Edge**: Fedora, Arch Linux (for enthusiasts).
Each distro includes a desktop environment (e.g., GNOME, KDE), utilities (e.g., GNU Core Utilities), and package managers (e.g., dnf for RHEL, apt for Ubuntu).

## Development of Red Hat Enterprise Linux

- **Origins**: RHEL evolved from Red Hat Linux, a community distro, into an enterprise-focused product in 2002.
- **Development Model**: Built on open-source principles, with contributions from Red Hat engineers, the community, and partners.
- **Fedora as Upstream**: Fedora, a community-driven distro, serves as a testing ground for features that may later appear in RHEL.
- **CentOS Stream**: A rolling-release distro that acts as a development branch for future RHEL releases, allowing community input.
- **RHEL Lifecycle**: Each major RHEL version (e.g., RHEL 9, released 2022) has a 10-year support cycle, with regular updates for security, bug fixes, and new features.
- **Recent Innovations**: RHEL 10 (in development) introduces AI-assisted tools like RHEL Lightspeed for natural language-based administration and enhanced container support with Podman.

### Practical Examples

- **Check RHEL/OS Version**:

  ```bash
  cat /etc/redhat-release  # Only works for RHEL and displays the RHEL version
  cat /etc/os-release  # Provides detailed OS information (works for all Linux distros)
  ```
