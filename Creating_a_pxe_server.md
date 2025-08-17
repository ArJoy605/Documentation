# Full PXE Server Setup & Client Install (Oracle Linux 8)

This guide sets up a PXE server on Oracle Linux 8 for unattended client installation of Oracle Linux 8 with a GUI, using a host-only network. SELinux and firewalld are disabled on both server and client. Mounts are made persistent via `/etc/fstab`.

## 0 VM Setup

**PXE Server VM**:

- OS: Oracle Linux 8 (minimal, no GUI)
- RAM: 2–4 GB, CPU: 2 vCPU
- Disk: 40 GB
- Storage: Attach Oracle Linux 8 ISO (e.g., `/root/OracleLinux-R8-U8-x86_64-dvd.iso`)
- Network:
  - Adapter 1: NAT (optional, for internet)
  - Adapter 2: Host-only (vboxnet0, static IP `192.168.56.10/24`)

**Client VM**:

- No OS (PXE boot only)
- RAM: 4 GB, Disk: 20 GB
- Network: Adapter 1: Host-only (vboxnet0)
- Boot order: Network first

## 1 Verify PXE Server Network

Check the host-only NIC (e.g., `enp0s8`) and confirm IP:

```bash
ip a
ping 192.168.56.10  # From host machine
```

Ensure the host-only NIC has `192.168.56.10/24`. Note the NIC name (e.g., `enp0s8`) for Step 3.

## 2 Disable SELinux and firewalld (Server)

Disable SELinux and firewalld:

```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
setenforce 0
systemctl disable --now firewalld
```

## 3 Install Required Packages

Install minimal packages for PXE:

```bash
dnf install -y dhcp-server tftp-server syslinux syslinux-tftpboot httpd
```

- `dhcp-server`: Provides DHCP for PXE clients.
- `tftp-server`: Serves PXE boot files.
- `syslinux-tftpboot`: Provides PXE bootloader modules (`pxelinux.0`, `ldlinux.c32`, etc.).
- `httpd`: Serves the Oracle Linux ISO and Kickstart file.

## 4 Configure DHCP (Host-only Network)

Bind DHCP to the host-only NIC (replace `enp0s8` with your NIC name from `ip a`):

```bash
echo "DHCPDARGS=enp0s8" > /etc/sysconfig/dhcpd
```

Create `/etc/dhcp/dhcpd.conf`:

```bash
cat >/etc/dhcp/dhcpd.conf <<'EOF'
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.56.0 netmask 255.255.255.0 {
  range 192.168.56.100 192.168.56.200;
  option routers 192.168.56.1;
  option broadcast-address 192.168.56.255;
  option domain-name-servers 8.8.8.8;
  next-server 192.168.56.10;
  filename "pxelinux.0";
}
EOF
```

Enable and start DHCP:

```bash
systemctl enable --now dhcpd
journalctl -xeu dhcpd --no-pager | tail -n 50
```

Verify DHCP is running and bound to `192.168.56.10`.

## 5 Configure TFTP & PXE Boot Files

Enable TFTP:

```bash
systemctl enable --now tftp.socket
```

Set up TFTP root and copy PXE boot files from `syslinux-tftpboot`:

```bash
mkdir -p /var/lib/tftpboot/pxelinux.cfg
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
cp /usr/share/syslinux/ldlinux.c32 /var/lib/tftpboot/
cp /usr/share/syslinux/menu.c32 /var/lib/tftpboot/
cp /usr/share/syslinux/vesamenu.c32 /var/lib/tftpboot/
cp /usr/share/syslinux/libutil.c32 /var/lib/tftpboot/
```

## 6 Mount ISO & Serve via HTTP (Bind Mount, Persistent)

Mount the Oracle Linux 8 ISO and make it persistent:

```bash
mkdir -p /mnt/ol8iso
mkdir -p /var/www/html/ol8
mount -o loop /YOURPATH/OracleLinux-R8-U8-x86_64-dvd.iso /mnt/ol8iso
mount --bind /mnt/ol8iso /var/www/html/ol8

# Add to /etc/fstab for persistence
echo "/root/OracleLinux-R8-U8-x86_64-dvd.iso /mnt/ol8iso iso9660 loop,ro 0 0" >> /etc/fstab
echo "/mnt/ol8iso /var/www/html/ol8 none bind 0 0" >> /etc/fstab

# Verify fstab entries
cat /etc/fstab

# Test mounts
umount /var/www/html/ol8
umount /mnt/ol8iso
mount -a
ls /var/www/html/ol8  # Should list ISO contents (e.g., BaseOS, AppStream)
```

Enable and start HTTP server:

```bash
systemctl enable --now httpd
curl http://192.168.56.10/ol8/  # Test access
```

**Note**: Replace `/root/OracleLinux-R8-U8-x86_64-dvd.iso` with the actual path to your ISO.

## 7 Extract PXE Boot Kernel & Initrd

Copy kernel and initrd for PXE booting:

```bash
cp /mnt/ol8iso/images/pxeboot/vmlinuz /var/lib/tftpboot/
cp /mnt/ol8iso/images/pxeboot/initrd.img /var/lib/tftpboot/
```

## 8 Kickstart File (Unattended Install)

Create `/var/www/html/ks.cfg` for unattended GUI install with SELinux disabled:

```bash
cat >/var/www/html/ks.cfg <<'EOF'
#version=OL8
install
lang en_US.UTF-8
keyboard us
timezone Asia/Dhaka --isUtc
rootpw --plaintext oracle123
reboot

url --url="http://192.168.56.10/ol8"

bootloader --location=mbr
zerombr
clearpart --all --initlabel
autopart --type=lvm

network --bootproto=dhcp --activate
auth --enableshadow --passalgo=sha512
selinux --disabled
services --enabled=sshd

user --name=oluser --groups=wheel --password=oracle123 --plaintext

%packages
@^Server with GUI
%end
EOF
```

Verify Kickstart accessibility:

```bash
curl http://192.168.56.10/ks.cfg
```

## 9 PXE Boot Menu

Create `/var/lib/tftpboot/pxelinux.cfg/default`:

```bash
cat >/var/lib/tftpboot/pxelinux.cfg/default <<'EOF'
default menu.c32
prompt 0
timeout 50
ONTIMEOUT install

menu title Oracle Linux 8 PXE Boot

label install
  menu label Install ^Oracle Linux 8
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=http://192.168.56.10/ol8 inst.ks=http://192.168.56.10/ks.cfg
EOF
```

## 10 Test & Boot PXE Client

1. Power on the client VM.
2. Client should:
   - Receive DHCP IP from `192.168.56.100–200`.
   - Download `pxelinux.0`, `ldlinux.c32`, and display the PXE menu.
   - Auto-boot after 5 seconds, load `vmlinuz` and `initrd.img`.
   - Fetch Kickstart via HTTP and perform unattended Oracle Linux 8 GUI install.
3. Post-install login:
   - Root: `root / oracle123`
   - User: `oluser / oracle123`

## 11 Verify Setup

Check services on the PXE server:

```bash
systemctl status dhcpd tftp.socket httpd
```

From the host, test:

```bash
curl http://192.168.56.10/ol8/
curl http://192.168.56.10/ks.cfg
```

Reboot the PXE server to confirm mount persistence:

```bash
reboot
# After reboot, check:
ls /var/www/html/ol8
curl http://192.168.56.10/ol8/
```

## Notes

- **ISO Path**: Ensure the ISO path (`/root/OracleLinux-R8-U8-x86_64-dvd.iso`) is correct and persistent across reboots.
- **NIC Name**: Replace `enp0s8` with the actual host-only NIC name from `ip a`.
- **SELinux/Firewalld**: Disabled on the server (Step 2) and client (Kickstart).
- **Persistence**: `/etc/fstab` entries ensure ISO and bind mounts persist across reboots.
- **Packages**: Only `dhcp-server`, `tftp-server`, `syslinux`, `syslinux-tftpboot`, and `httpd` are used.
- **Bind Mount**: Avoids copying 8 GB ISO contents.
- **Kickstart**: Installs `Server with GUI`, disables SELinux, and enables SSH.
