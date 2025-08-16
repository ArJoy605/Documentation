# Chapter 12: Manage Networking

## Key Concepts

- **Networking in Linux**:
  - Enables communication for services like SSH, web servers, and file sharing.
  - Managed by `NetworkManager` in RHEL, using `nmcli` (CLI) or `nmtui` (text UI).
- **NetworkManager**:
  - Default tool for network configuration in RHEL 9/10.
  - Stores settings in `/etc/NetworkManager/system-connections/`.
  - Supports dynamic (DHCP) and static IP configurations.
- **Key Files**:
  - `/etc/hosts`: Local hostname-to-IP mappings.
  - `/etc/resolv.conf`: DNS server list (managed by `NetworkManager`).
  - `/etc/sysconfig/network-scripts/` (legacy, rarely used in RHEL 9/10).
- **RHEL 9/10 Notes**:
  - Improved IPv6 support and `NetworkManager` performance in RHEL 10.
  - RHEL Lightspeed suggests commands (e.g., `rhel lightspeed "set static IP"` suggests `nmcli` commands).

### Describing Networking Concepts

- **TCP/IP Network Model**:
  - **Application Layer**: Protocols like HTTP, SSH (e.g., `sshd` on port 22).
  - **Transport Layer**: TCP (reliable, connection-oriented) or UDP (connectionless).
  - **Network Layer**: IP (IPv4/IPv6) for addressing and routing.
  - **Link Layer**: Ethernet/Wi-Fi for physical connections.
- **Key Terms**:
  - **IP Address**: Unique identifier for a device (e.g., `192.168.1.100`).
  - **Subnet Mask**: Defines network range (e.g., `/24` or `255.255.255.0`).
  - **Gateway**: Routes traffic to other networks (e.g., `192.168.1.1`).
  - **DNS**: Resolves names to IPs (e.g., `8.8.8.8`).
- **Example**:
  - View TCP/IP connections:

    ```bash
    ss -tunap
    ```

    Output: Shows TCP/UDP sockets, ports, and processes (e.g., `sshd` on `:22`).

### Describing Network Interface Names

- **Naming Convention**:
  - **Predictable Names**: Used in RHEL 9/10 (e.g., `enp0s3` for Ethernet, `wlan0` for Wi-Fi, `lo` for loopback).
  - Based on hardware location (e.g., `enp0s3`: PCI slot 0, port 3).
  - Legacy: `eth0`, `wlan0` (less common).
- **Commands**:
  - `ip link show`: List interfaces.
  - `nmcli device status`: Show device and connection status.
- **Examples**:
  1. List interfaces:

     ```bash
     ip link show
     ```

     Output: Shows `lo`, `enp0s3`, etc., with state (UP/DOWN).
  2. Check device status:

     ```bash
     nmcli device status
     ```

  3. Corner Case: Interface not found:

     ```bash
     nmcli con up eth0
     # Error: Unknown connection
     # Fix: Check name with ip link show
     ```

### IPv4 Networking

- **IPv4 Addresses**:
  - 32-bit, formatted as `xxx.xxx.xxx.xxx` (e.g., `192.168.1.100`).
  - Classes: Private (e.g., `192.168.0.0/16`, `10.0.0.0/8`) vs. public.
  - Subnet mask: Defines network/host portions (e.g., `/24` = 256 addresses).
- **IPv4 Routing**:
  - Routes traffic via a gateway (e.g., `192.168.1.1`).
  - Default route: Used when destination is outside local network.
- **Examples**:
  1. View IP addresses:

     ```bash
     ip addr show
     ```

  2. Check routing table:

     ```bash
     ip route show # Show routing table
     ip route get  # Show routing table
     ```

     Output: Shows `default via 192.168.1.1`.
  3. Corner Case: No route to host:

     ```bash
     ping 8.8.8.8
     # Error: Network unreachable
     # Fix: Set gateway: nmcli con mod "System enp0s3" ipv4.gateway 192.168.1.1
     ```

### IPv4 Address and Routing Configuration

- **Purpose**: Configure static or dynamic IPv4 settings.
- **Commands**:
  - Static: `nmcli con mod <name> ipv4.addresses <IP> ipv4.method manual`.
  - Dynamic: `nmcli con mod <name> ipv4.method auto`.
- **Examples**:
  1. Set static IP:

     ```bash
     sudo nmcli con mod "System enp0s3" ipv4.addresses 192.168.1.100/24
     sudo nmcli con mod "System enp0s3" ipv4.gateway 192.168.1.1
     sudo nmcli con mod "System enp0s3" ipv4.dns 8.8.8.8
     sudo nmcli con mod "System enp0s3" ipv4.method manual
     sudo nmcli con up "System enp0s3"
     ```

  2. Switch to DHCP:

     ```bash
     sudo nmcli con mod "System enp0s3" ipv4.method auto
     sudo nmcli con up "System enp0s3"
     ```

  3. Corner Case: Invalid IP:

     ```bash
     nmcli con mod "System enp0s3" ipv4.addresses 256.1.1.1
     # Error: Invalid IP
     # Fix: Use valid IP (e.g., 192.168.1.100)
     ```

### IPv6 Networking

- **IPv6 Addresses**:
  - 128-bit, formatted as eight groups of hexadecimal digits (e.g., `2001:db8::1`).
  - Types: Global, link-local (starts with `fe80::`), unique local.
- **IPv6 Subnetting**:
  - Uses CIDR notation (e.g., `/64` for 2^64 addresses).
  - Common for LANs: `/64`, larger for ISPs: `/48`.
- **IPv6 Address Configuration**:
  - Static or dynamic (SLAAC, DHCPv6).
  Here’s a clean **note on IPv6 address writing conventions** based on your text:

---

### IPv6 Address Notation Rules 📝

1. **Leading zeros** → Suppress them in each group.

   - Example: `2001:0db8:0001::1` → `2001:db8:1::1`

2. **Zero compression (::)** → Use `::` to replace the **longest** consecutive sequence of all-zeros groups.

   - Example: `2001:db8:0:0:0:0:0:1` → `2001:db8::1`

3. **Tie-breaking rule** → If multiple zero sequences are equal in length, shorten the **leftmost** one with `::`.

   - Example: `2001:0:0:1:0:0:0:1` → `2001::1:0:0:1`

4. **Single zero group** → Do **not** shorten with `::`.

   - Instead use `:0:`
   - Example: `2001:db8:0:1::1` (✔ good) vs `2001:db8::1:1` (✘ avoid for single group)

5. **Hex case** → Always use **lowercase letters** `a–f`.

   - Example: `2001:db8::dead:beef`

---

- **Example Commands**:
  1. View IPv6 addresses:

     ```bash
     ip -6 addr show
     ```

  2. Set static IPv6:

     ```bash
     sudo nmcli con mod "System enp0s3" ipv6.addresses 2001:db8::100/64
     sudo nmcli con mod "System enp0s3" ipv6.method manual
     sudo nmcli con up "System enp0s3"
     ```

  3. Enable SLAAC:

     ```bash
     sudo nmcli con mod "System enp0s3" ipv6.method auto
     sudo nmcli con up "System enp0s3"
     ```

  4. Corner Case: IPv6 disabled:

     ```bash
     ip -6 addr show
     # No output
     # Fix: Enable IPv6: nmcli con mod "System enp0s3" ipv6.method auto
     ```

### Validating Network Configuration

- **Purpose**: Ensure IP, routing, and DNS settings are correct.
- **Commands**:
  - `ip addr`, `ip route`, `nmcli con show`, `ping`, `dig`.
- **Examples**:
  1. Verify IP:

     ```bash
     ip addr show enp0s3
     ```

  2. Check routing:

     ```bash
     ip route show
     ```

  3. Test connectivity:

     ```bash
     ping -c 4 8.8.8.8
     ```

  4. Corner Case: DNS failure:

     ```bash
     ping google.com
     # Error: Name resolution failed
     # Fix: Set DNS: nmcli con mod "System enp0s3" ipv4.dns 8.8.8.8
     ```

### Gathering Network Interface Information

- **Commands**:
  - `nmcli device status`: Device state.
  - `ip link show`: Interface details.
  - `ethtool <interface>`: Link status, speed.
- **Examples**:
  1. Show device status:

     ```bash
     nmcli device status
     ```

  2. Check interface details:

     ```bash
     ip link show enp0s3
     ```

  3. View link speed:

     ```bash
     sudo ethtool enp0s3
     ```

  4. Corner Case: Interface down:

     ```bash
     ip link show enp0s3
     # State: DOWN
     # Fix: sudo nmcli con up "System enp0s3"
     ```

### Displaying IP Addresses

- **Commands**:
  - `ip addr show`: Show all IPs.
  - `nmcli con show <name>`: Show connection IPs.
- **Examples**:
  1. List all IPs:

     ```bash
     ip addr show
     ```

  2. Show specific connection:

     ```bash
     nmcli con show "System enp0s3"
     ```

  3. Corner Case: Multiple IPs:

     ```bash
     sudo nmcli con mod "System enp0s3" +ipv4.addresses 192.168.1.101/24
     ip addr show enp0s3  # Shows multiple IPs
     ```

### Displaying Performance Statistics

- **Commands**:
  - `ip -s link`: Interface traffic stats.
  - `nload`, `iftop` (install: `sudo dnf install nload iftop`).
- **Examples**:
  1. Show interface stats:

     ```bash
     ip -s link show enp0s3
     ```

  2. Monitor real-time traffic:

     ```bash
     sudo nload enp0s3
     ```

  3. Check per-connection stats:

     ```bash
     sudo iftop -i enp0s3
     ```

  4. Corner Case: Tools missing:

     ```bash
     sudo dnf install nload
     ```

### Checking Connectivity Between Hosts

- **Commands**:
  - `ping`: Test reachability.
  - `nc -zv <host> <port>`: Test port connectivity.
- **Examples**:
  1. Test external host:

     ```bash
     ping -c 4 8.8.8.8
     ```

  2. Test SSH port:

     ```bash
     nc -zv 192.168.1.100 22
     ```

  3. Corner Case: Ping fails:

     ```bash
     ping 8.8.8.8
     # Error: Network unreachable
     # Fix: Check gateway: ip route
     ```

### Troubleshooting Routing

- **Commands**:
  - `ip route`: View routing table.
  - `traceroute`: Trace path to destination.
- **Examples**:
  1. Check routes:

     ```bash
     ip route show
     ```

  2. Trace to Google:

     ```bash
     traceroute 8.8.8.8
     ```

  3. Corner Case: No default route:

     ```bash
     ip route show
     # No default via
     # Fix: nmcli con mod "System enp0s3" ipv4.gateway 192.168.1.1
     ```

### Tracing Routes Taken by Traffic

- **Command**: `traceroute` or `mtr` (install: `sudo dnf install mtr`).
- **Examples**:
  1. Trace to external host:

     ```bash
     traceroute google.com
     ```

  2. Real-time tracing:

     ```bash
     mtr 8.8.8.8
     ```

  3. Corner Case: Traceroute blocked:

     ```bash
     traceroute 8.8.8.8
     # Output: Stars (*)
     # Fix: Check firewall: sudo firewall-cmd --list-all
     ```

### Troubleshooting Ports and Services

- **Commands**:
  - `ss -tuln`: List open ports.
  - `firewall-cmd --list-services`: Check firewall rules.
  - `nc -zv`: Test port connectivity.
- **Examples**:
  1. Check open ports:

     ```bash
     ss -tuln
     ```

  2. Verify SSH service:

     ```bash
     nc -zv 192.168.1.100 22
     ```

  3. Corner Case: Port blocked:

     ```bash
     nc -zv 192.168.1.100 80
     # Error: Connection refused
     # Fix: sudo firewall-cmd --add-service=http --permanent
     ```

### Configuring Networking From the Command Line

- **Tool**: `nmcli` for CLI configuration.
- **Examples**:
  1. Add Ethernet connection:

     ```bash
     sudo nmcli con add type ethernet ifname enp0s3 con-name my-ethernet ipv4.method auto
     ```

  2. Add Wi-Fi connection:

     ```bash
     sudo nmcli con add type wifi ifname wlan0 con-name my-wifi ssid MyWiFi wifi-sec.key-mgmt wpa-psk wifi-sec.psk MyPassword
     ```

  3. Corner Case: Wrong interface:

     ```bash
     nmcli con add type ethernet ifname eth99
     # Error: Invalid device
     # Fix: ip link show
     ```

### Describing NetworkManager Concepts

- **Key Features**:
  - Manages connections (Ethernet, Wi-Fi, VPN).
  - Connection profiles: Stored in `/etc/NetworkManager/system-connections/`.
  - Methods: `manual` (static), `auto` (DHCP/SLAAC).
- **Tools**: `nmcli`, `nmtui`, `nm-connection-editor` (GUI, not in RH124).
- **Example**:
  - View connection details:

    ```bash
    nmcli con show "System enp0s3"
    ```

### Viewing Networking Information

- **Commands**:
  - `nmcli`, `ip addr`, `ip route`, `ss`.
- **Examples**:
  1. Show all connections:

     ```bash
     nmcli
     ```

  2. Detailed connection info:

     ```bash
     nmcli con show "System enp0s3"
     ```

  3. Corner Case: No connections:

     ```bash
     nmcli con show
     # Empty
     # Fix: nmcli con add type ethernet ifname enp0s3
     ```

### Adding a Network Connection

- **Command**: `nmcli con add`.
- **Examples**:
  1. Add static Ethernet:

     ```bash
     sudo nmcli con add type ethernet ifname enp0s3 con-name my-ethernet ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
     ```

  2. Add Wi-Fi:

     ```bash
     sudo nmcli con add type wifi ifname wlan0 con-name my-wifi ssid MyWiFi wifi-sec.key-mgmt wpa-psk wifi-sec.psk MyPassword
     ```

  3. Corner Case: Duplicate connection name:

     ```bash
     nmcli con add type ethernet ifname enp0s3 con-name my-ethernet
     # Error: Connection exists
     # Fix: Use unique name or delete: nmcli con delete my-ethernet
     ```

### Controlling Network Connections

- **Commands**:
  - `nmcli con up/down`: Activate/deactivate.
  - `nmcli device connect/disconnect`: Control device.
- **Examples**:
  1. Activate connection:

     ```bash
     sudo nmcli con up my-ethernet
     ```

  2. Deactivate:

     ```bash
     sudo nmcli con down my-ethernet
     ```

  3. Corner Case: Connection fails to activate:

     ```bash
     nmcli con up my-ethernet
     # Error: Activation failed
     # Fix: Check logs: journalctl -u NetworkManager
     ```

### Modifying Network Connection Settings

- **Command**: `nmcli con mod`.
- **Examples**:
  1. Change IP:

     ```bash
     sudo nmcli con mod my-ethernet ipv4.addresses 192.168.1.200/24
     sudo nmcli con up my-ethernet
     ```

  2. Add secondary DNS:

     ```bash
     sudo nmcli con mod my-ethernet +ipv4.dns 8.8.4.4
     ```

  3. Corner Case: Invalid setting:

     ```bash
     nmcli con mod my-ethernet ipv4.addresses 256.1.1.1
     # Error: Invalid IP
     # Fix: Use valid IP
     ```

### Deleting Network Connection

- **Command**: `nmcli con delete`.
- **Examples**:
  1. Delete connection:

     ```bash
     sudo nmcli con delete my-ethernet
     ```

  2. Verify:

     ```bash
     nmcli con show
     ```

  3. Corner Case: Connection in use:

     ```bash
     nmcli con delete my-ethernet
     # Error: Connection active
     # Fix: nmcli con down my-ethernet
     ```

### Who Can Modify Network Settings?

- **Permissions**:
  - Root (`sudo`) required for system-wide changes.
  - Users in `nmcli` group can manage user-specific connections.
- **Example**:
  - Check permissions:

    ```bash
    nmcli con show
    # Non-root may see limited connections
    sudo nmcli con show
    ```

### Summary of nmcli Commands

- **View**:
  - `nmcli`: Overview.
  - `nmcli device status`: Device state.
  - `nmcli con show`: Connection details.
- **Manage**:
  - `nmcli con add`: Add connection.
  - `nmcli con mod`: Modify settings.
  - `nmcli con up/down`: Activate/deactivate.
  - `nmcli con delete`: Remove connection.
- **Troubleshoot**:
  - `nmcli con show <name>`: Inspect settings.
  - `journalctl -u NetworkManager`: Check logs.

### Editing Network Configuration Files

- **Files**:
  - `/etc/NetworkManager/system-connections/*.nmconnection`: Connection profiles.
  - `/etc/hosts`: Hostname mappings.
  - `/etc/resolv.conf`: DNS servers (managed by `NetworkManager`).
- **Example**:
  - Edit connection file:

    ```bash
    sudo vim /etc/NetworkManager/system-connections/my-ethernet.nmconnection
    sudo nmcli con reload
    ```

### Modifying Network Configuration

- **Preferred Method**: `nmcli` or `nmtui` to avoid file conflicts.
- **Example**:
  - Modify via `nmcli`:

    ```bash
    sudo nmcli con mod my-ethernet ipv4.addresses 192.168.1.150/24
    sudo nmcli con up my-ethernet
    ```

### Configuring Host Names and Name Resolution

- **Files**:
  - `/etc/hostname`: System hostname.
  - `/etc/hosts`: Local name-to-IP mappings.
  - `/etc/resolv.conf`: DNS servers.
- **Commands**:
  - `hostnamectl`: Set hostname.
  - `nmcli con mod`: Set DNS.
- **Examples**:
  1. Set hostname:

     ```bash
     sudo hostnamectl set-hostname server1.example.com
     ```

  2. Add host mapping:

     ```bash
     sudo vim /etc/hosts
     # Add: 192.168.1.100 server1.example.com server1
     ```

  3. Set DNS:

     ```bash
     sudo nmcli con mod my-ethernet ipv4.dns "8.8.8.8,8.8.4.4"
     ```

  4. Corner Case: DNS not applied:

     ```bash
     cat /etc/resolv.conf
     # No DNS servers
     # Fix: nmcli con up my-ethernet
     ```

### Practical Examples

1. **Configure Static IPv4**:

   ```bash
   sudo nmcli con add type ethernet ifname enp0s3 con-name my-ethernet ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
   sudo nmcli con up my-ethernet
   ip addr show enp0s3
   ```

2. **Set Up Wi-Fi**:

   ```bash
   sudo nmcli con add type wifi ifname wlan0 con-name my-wifi ssid MyWiFi wifi-sec.key-mgmt wpa-psk wifi-sec.psk MyPassword
   sudo nmcli con up my-wifi
   ```

3. **Troubleshoot Connectivity**:

   ```bash
   ping 8.8.8.8
   ping google.com
   journalctl -u NetworkManager -b
   sudo firewall-cmd --list-all
   ```

4. **Add Hostname Mapping**:

   ```bash
   sudo vim /etc/hosts
   # Add: 192.168.1.200 server2.example.com
   ping server2.example.com
   ```

5. **Monitor Network Traffic**:

   ```bash
   sudo nload enp0s3
   sudo iftop -i enp0s3
   ```

### Common Pitfalls

- **No `sudo`**: Network changes require root (e.g., `sudo nmcli`).
- **Not Activating**: Run `nmcli con up` after modifications.
- **Firewall Blocks**: Check `firewalld`:

  ```bash
  sudo firewall-cmd --add-service=ssh --permanent
  ```

- **DNS Issues**: Verify `/etc/resolv.conf` or set DNS:

  ```bash
  sudo nmcli con mod my-ethernet ipv4.dns 8.8.8.8
  ```

- **Wrong Interface**: Use `ip link show` to confirm names.
- **SELinux**: May block connections:

  ```bash
  sudo restorecon -R /etc/NetworkManager
  ```
