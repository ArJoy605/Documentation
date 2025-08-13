# RH124 Revision Notebook: Chapter 12 - Manage Networking

This notebook provides a detailed summary of Chapter 12 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on managing network configurations, troubleshooting connectivity, and securing network interfaces, with comprehensive explanations, commands, practical examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 12: Manage Networking

### Key Concepts

- **Networking in Linux**:
  - Networking enables communication between systems for services like SSH, web servers, or file sharing.
  - RHEL uses `NetworkManager` as the default tool for managing network configurations.
- **Network Interfaces**:
  - Types: Ethernet (`eth0`), Wi-Fi (`wlan0`), loopback (`lo`).
  - Identified by name (e.g., `enp0s3` for Ethernet).
  - Configured via files in `/etc/NetworkManager/system-connections/` or CLI tools.
- **IP Addressing**:
  - **IPv4**: 32-bit addresses (e.g., `192.168.1.100`).
  - **IPv6**: 128-bit addresses (e.g., `2001:db8::1`).
  - Static (manually set) or dynamic (via DHCP).
  - Subnet mask (e.g., `/24` or `255.255.255.0`) defines network range.
- **Key Configuration Files**:
  - `/etc/hosts`: Maps hostnames to IP addresses for local resolution.
  - `/etc/resolv.conf`: Lists DNS servers for name resolution.
  - `/etc/sysconfig/network-scripts/` (legacy, less common in RHEL 9/10).
- **NetworkManager Tools**:
  - `nmcli`: Command-line interface for managing network settings.
  - `nmtui`: Text-based UI for network configuration.
  - `nm-connection-editor`: GUI tool (not covered in RH124).
- **Troubleshooting**:
  - Tools like `ping`, `traceroute`, `ss`, and `ip` diagnose connectivity issues.
  - Firewall (`firewalld`) and SELinux may block connections.
- **RHEL 9/10 Notes**:
  - RHEL 10 improves `NetworkManager` with better IPv6 support and performance.
  - RHEL Lightspeed (RHEL 10) can suggest networking commands (e.g., `rhel lightspeed "configure static IP"` suggests `nmcli` commands).

### Important Commands

- **Viewing Network Status**:

  ```bash
  nmcli  # Show all connections and status
  nmcli device status  # List devices and their state
  nmcli connection show  # List configured connections
  ip addr show  # Show IP addresses and interfaces
  ip link show  # Show network interfaces
  ```

- **Managing Connections**:

  ```bash
  nmcli con up eth0  # Activate connection eth0
  nmcli con down eth0  # Deactivate connection
  nmcli con show "System eth0"  # Show details of connection
  ```

- **Configuring IP Addresses**:

  ```bash
  nmcli con mod "System eth0" ipv4.addresses 192.168.1.100/24  # Set static IPv4
  nmcli con mod "System eth0" ipv4.gateway 192.168.1.1  # Set gateway
  nmcli con mod "System eth0" ipv4.dns "8.8.8.8,8.8.4.4"  # Set DNS
  nmcli con mod "System eth0" ipv4.method manual  # Enable static IP
  nmcli con mod "System eth0" ipv4.method auto  # Enable DHCP
  nmcli con up "System eth0"  # Apply changes
  ```

- **Editing Hosts File**:

  ```bash
  sudo vim /etc/hosts  # Add entries (e.g., 192.168.1.100 server1.example.com)
  ```

- **Troubleshooting Connectivity**:

  ```bash
  ping 8.8.8.8  # Test connectivity to external IP
  ping google.com  # Test DNS resolution
  traceroute google.com  # Trace route to host
  ss -tuln  # List open ports (e.g., port 22 for SSH)
  dig google.com  # Query DNS details
  ```

- **Firewall Checks**:

  ```bash
  sudo firewall-cmd --list-all  # Show active firewall zone
  sudo firewall-cmd --add-service=ssh --permanent  # Allow SSH
  sudo firewall-cmd --reload  # Apply firewall changes
  ```

  Got it! You want to create a note. Since you mentioned adding a new network connection with `nmcli` earlier, I’ll assume you want a quick note related to that task, but feel free to clarify if you meant something else. Here’s a concise note summarizing the `nmcli` process for adding a network connection in Linux:

---

- **Note: Adding a Network Connection with nmcli**

- **Purpose**: Add a new network connection (e.g., Ethernet, Wi-Fi) using `nmcli`.  
- **Basic Command**:

  ```bash
  nmcli con add type <type> ifname <interface> con-name <name> [options]
  ```  

- **Examples**:  
  - Ethernet (DHCP): `nmcli con add type ethernet ifname enp0s3 con-name my-ethernet`  
  - Ethernet (Static IP): `nmcli con add type ethernet ifname enp0s3 con-name my-ethernet ipv4.method manual ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8`  
  - Wi-Fi: `nmcli con add type wifi ifname wlan0 con-name my-wifi ssid MyWiFi wifi-sec.key-mgmt wpa-psk wifi-sec.psk MyPassword`  
- **Activate**: `nmcli con up <con-name>`  
- **Verify**: `nmcli con show` or `nmcli device status`  
- **Tip**: Check interface names with `ip link show`.  

---

### Practical Examples

- **Scenario: Configure Static IP**:
  - Set static IP `192.168.1.100/24` for `eth0`:

    ```bash
    sudo nmcli con mod "System eth0" ipv4.addresses 192.168.1.100/24
    sudo nmcli con mod "System eth0" ipv4.gateway 192.168.1.1
    sudo nmcli con mod "System eth0" ipv4.dns 8.8.8.8
    sudo nmcli con mod "System eth0" ipv4.method manual
    sudo nmcli con up "System eth0"
    ip addr show eth0  # Verify
    ```

- **Scenario: Enable DHCP**:
  - Switch `eth0` to DHCP:

    ```bash
    sudo nmcli con mod "System eth0" ipv4.method auto
    sudo nmcli con up "System eth0"
    ip addr show eth0  # Check assigned IP
    ```

- **Scenario: Troubleshoot No Connectivity**:
  - Test internet: `ping 8.8.8.8` (if fails, check interface).
  - Check interface status: `nmcli device status`.
  - Verify DNS: `cat /etc/resolv.conf` or `dig google.com`.
  - Check firewall: `sudo firewall-cmd --list-services` (ensure `ssh` or required service is allowed).
- **Scenario: Add Hostname Mapping**:
  - Map `server1` to `192.168.1.100`:

    ```bash
    sudo vim /etc/hosts
    # Add: 192.168.1.100 server1.example.com server1
    ping server1  # Test resolution
    ```

- **Daily Use: Monitor Open Ports**:
  - Check services listening:

    ```bash
    ss -tuln
    ```

    Output: Shows ports (e.g., `:22` for SSH, `:80` for HTTP).
- **Scenario: Allow HTTP in Firewall**:
  - Enable web traffic:

    ```bash
    sudo firewall-cmd --add-service=http --permanent
    sudo firewall-cmd --reload
    sudo firewall-cmd --list-services
    ```

### Common Pitfalls

- **Forgetting `sudo`**: Network configuration requires root privileges (e.g., `sudo nmcli`).
- **Not Activating Changes**: After modifying a connection, run `nmcli con up` to apply.
- **Firewall Blocking**: Missing firewall rules block services (e.g., SSH on port 22). Check with `firewall-cmd --list-all`.
- **DNS Issues**: Incorrect `/etc/resolv.conf` or missing DNS servers cause name resolution failures. Verify with `dig` or `nslookup`.
- **Interface Naming**: RHEL uses predictable names (e.g., `enp0s3`); legacy names (e.g., `eth0`) may differ. Check with `ip link`.
- **Overwriting Configs**: Manual edits to `/etc/sysconfig/network-scripts/` can conflict with `NetworkManager`. Use `nmcli` instead.

### Best Practices and Tips

- **Use `nmcli` for Consistency**: Avoid manual file edits; `nmcli` integrates with `NetworkManager`.
- **Verify Changes**: Check IP with `ip addr` and connectivity with `ping` after configuration.
- **Backup Configs**: Save network settings:

  ```bash
  sudo cp -r /etc/NetworkManager/system-connections /backup/network-$(date +%F)
  ```

- **Test Connectivity**: Use `ping 8.8.8.8` for IP-level and `ping google.com` for DNS.
- **Secure Firewall**: Only open necessary ports (e.g., `sudo firewall-cmd --add-service=ssh`).
- **RHEL 10 Tip**: Use Lightspeed for networking tasks (e.g., `rhel lightspeed "set static IP"` suggests `nmcli` commands).
- **Automate Checks**: Script network status:

  ```bash
  nmcli device status > network_status.log
  ip addr >> network_status.log
  ```

### Revision Quiz/Notes

- **Questions**:
  - What command sets a static IP for `eth0`? (`nmcli con mod "System eth0" ipv4.addresses 192.168.1.100/24`)
  - How do you check open ports? (`ss -tuln`)
  - What’s the purpose of `/etc/hosts`? (Local hostname-to-IP mapping.)
- **Quick Exercise**:
  - Set a static IP `192.168.1.200/24` for `enp0s3`, set DNS to `8.8.8.8`, and verify:

    ```bash
    sudo nmcli con mod "System enp0s3" ipv4.addresses 192.168.1.200/24
    sudo nmcli con mod "System enp0s3" ipv4.dns 8.8.8.8
    sudo nmcli con mod "System enp0s3" ipv4.method manual
    sudo nmcli con up "System enp0s3"
    ip addr show enp0s3
    ```

- **Self-Test**:
  - Explain the output of `nmcli device status` (shows device, type, state, connection).
  - Why check the firewall after network changes? (Ensures services like SSH are allowed.)

---
