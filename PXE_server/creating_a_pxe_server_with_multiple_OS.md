# Creating a PXE Server with Multiple Operating Systems

To support multiple operating systems (OSes) on your PXE server for unattended client installations, you need to modify the setup to include additional OS ISOs, their respective kernel and initrd files, Kickstart files for each OS, and an updated PXE boot menu to allow clients to select the desired OS. This guide will extend your existing Oracle Linux 8 PXE server setup to support multiple OSes, maintaining the host-only network configuration and unattended installation approach with SELinux and firewalld disabled. I’ll assume you want to add another OS (e.g., Oracle Linux 9 or another Linux distribution like CentOS Stream 9) alongside Oracle Linux 8, but the steps are generalizable to other OSes.

## Prerequisites

Please ensure you have used the original PXE server setup guide to create a working PXE server with Oracle Linux 8. This guide builds upon that setup.

### Overview of Changes

1. **Mount Additional ISOs**: Mount each additional OS ISO and serve it via HTTP, with persistent `/etc/fstab` entries.
2. **Extract Kernel and Initrd**: Copy the kernel (`vmlinuz`) and initial ramdisk (`initrd.img`) for each OS to the TFTP root.
3. **Create Kickstart Files**: Provide a Kickstart file for each OS to enable unattended installation.
4. **Update PXE Boot Menu**: Modify the PXE menu (`/var/lib/tftpboot/pxelinux.cfg/default`) to list all OSes, allowing clients to choose which to install.
5. **Verify and Test**: Ensure all services (DHCP, TFTP, HTTP) are running and test booting clients with each OS option.

### Detailed Steps to Add Support for Multiple OSes

#### Step 1: Prepare Additional OS ISOs

For each additional OS (e.g., Oracle Linux 9), obtain the ISO and place it on the PXE server. For example, assume the Oracle Linux 9 ISO is at `/YourAbsolutePath/OracleLinux-R9-U1-x86_64-dvd.iso`.

#### Step 2: Mount Additional ISOs and Serve via HTTP

Create separate mount points and HTTP directories for each OS to avoid conflicts with the existing Oracle Linux 8 setup.

```bash
# Create mount points and HTTP directories for Oracle Linux 9
mkdir -p /mnt/ol9iso
mkdir -p /var/www/html/ol9

# Mount the Oracle Linux 9 ISO
mount -o loop /YourAbsolutePath/OracleLinux-R9-U1-x86_64-dvd.iso /mnt/ol9iso
mount --bind /mnt/ol9iso /var/www/html/ol9

# Add to /etc/fstab for persistence
echo "/YourAbsolutePath/OracleLinux-R9-U1-x86_64-dvd.iso /mnt/ol9iso iso9660 loop,ro 0 0" >> /etc/fstab
echo "/mnt/ol9iso /var/www/html/ol9 none bind 0 0" >> /etc/fstab
```

**Notes**:

- Replace `/YourAbsolutePath/OracleLinux-R9-U1-x86_64-dvd.iso` with the actual path to the ISO.
- Use unique mount points (e.g., `/mnt/ol9iso`) and HTTP paths (e.g., `/var/www/html/ol9`) for each OS to avoid conflicts.
- Optionally, edit `/etc/fstab` using `vim` or `nano` instead of `echo`.

Verify the mounts:

```bash
cat /etc/fstab
umount /var/www/html/ol9
umount /mnt/ol9iso
mount -a
ls /var/www/html/ol9  # Should list ISO contents (e.g., BaseOS, AppStream)
curl http://192.168.56.10/ol9/  # Test HTTP access
```

Repeat this step for each additional OS, using distinct paths (e.g., `/mnt/centos9iso` and `/var/www/html/centos9` for CentOS Stream 9).

#### Step 3: Extract Kernel and Initrd for Each OS

Copy the kernel and initrd files for each additional OS to the TFTP root, using unique filenames to avoid overwriting Oracle Linux 8 files.

For Oracle Linux 9:

```bash
cp /mnt/ol9iso/images/pxeboot/vmlinuz /var/lib/tftpboot/vmlinuz-ol9
cp /mnt/ol9iso/images/pxeboot/initrd.img /var/lib/tftpboot/initrd-ol9.img
```

**Notes**:

- Use descriptive names (e.g., `vmlinuz-ol9`, `initrd-ol9.img`) to distinguish from Oracle Linux 8 files (`vmlinuz`, `initrd.img`).
- For other OSes, locate the kernel and initrd in the ISO’s PXE boot directory (e.g., `/images/pxeboot/` for most Red Hat-based distributions).
- Verify the files:

  ```bash
  ls -l /var/lib/tftpboot/
  ```

#### Step 4: Create Kickstart Files for Each OS

Create a Kickstart file for each additional OS, tailored to its version and requirements. For Oracle Linux 9, create `/var/www/html/ks-ol9.cfg`:

```bash
cat >/var/www/html/ks-ol9.cfg <<'EOF'
#version=OL9
install
lang en_US.UTF-8
keyboard us
timezone Asia/Dhaka --isUtc
rootpw --plaintext oracle123
reboot

url --url="http://192.168.56.10/ol9"

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

For Rocky Linux 10, create `/var/www/html/ks-rocky10.cfg`:

```bash
#version=RHEL10
# Rocky Linux 10 Kickstart for PXE with GUI

# Use local PXE repo (BaseOS + AppStream)
url --url="http://192.168.56.11/rocky10"
repo --name=AppStream --baseurl=http://192.168.56.11/rocky10/AppStream

# Language/keyboard
lang en_US.UTF-8
keyboard us

# Network config
network --bootproto=dhcp --device=link --activate --hostname=rocky10.localdomain

# Timezone
timezone Asia/Dhaka --utc

# Bootloader
bootloader --location=mbr --boot-drive=sda

# Partitioning (EFI + LVM)
zerombr
clearpart --all --initlabel
part /boot/efi --fstype=efi --size=600
part /boot     --fstype=xfs --size=1024
part pv.01     --grow --size=1
volgroup rhel pv.01
logvol /       --vgname=rhel --name=root --fstype=xfs --size=10240 --grow
logvol swap    --vgname=rhel --name=swap --fstype=swap --size=2048

# Root password disabled, sudo user instead
rootpw --lock
user --name=joy --groups=wheel --plaintext --password=joy123098

# Enable GUI installer
graphical
firstboot --enable

# SELinux and Firewall
selinux --disabled
firewall --disabled

# Services
services --enabled=sshd,chronyd,NetworkManager

# Packages
%packages
@^minimal-environment
@standard
@guest-agents
kernel-devel
%end

# Post-install tasks
%post
dnf config-manager --set-enabled crb
%end

# Reboot after installation
reboot

```

**Notes**:

- Adjust the `#version` (e.g., `OL9`) and `url` (e.g., `http://192.168.56.10/ol9`) to match the OS and HTTP path.
- For other distributions (e.g., CentOS Stream 9), ensure the Kickstart syntax matches the OS version and package groups (e.g., `@graphical-server-environment` for CentOS).
- Optionally, edit the file using `vim` or `nano`.
- Verify accessibility:

  ```bash
  curl http://192.168.56.10/ks-ol9.cfg
  ```

Repeat for each OS, using unique Kickstart filenames (e.g., `/var/www/html/ks-centos9.cfg`).

#### Step 5: Update PXE Boot Menu

Modify `/var/lib/tftpboot/pxelinux.cfg/default` to include menu entries for all OSes. Update the existing menu to list both Oracle Linux 8 and Oracle Linux 9 (and any additional OSes).

```bash
cat >/var/lib/tftpboot/pxelinux.cfg/default <<'EOF'
default menu.c32
prompt 0
timeout 100
ONTIMEOUT install-ol8

menu title Oracle Linux PXE Boot Menu

label install-ol8
  menu label Install ^Oracle Linux 8
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=http://192.168.56.10/ol8 inst.ks=http://192.168.56.10/ks.cfg

label install-ol9
  menu label Install ^Oracle Linux 9
  kernel vmlinuz-ol9
  append initrd=initrd-ol9.img inst.stage2=http://192.168.56.10/ol9 inst.ks=http://192.168.56.10/ks-ol9.cfg
EOF
```

**Notes**:

- Increased `timeout` to 100 (10 seconds) to give users more time to select an OS.
- Set `ONTIMEOUT install-ol8` to default to Oracle Linux 8 if no selection is made.
- Added a new `label` section for Oracle Linux 9, pointing to its kernel (`vmlinuz-ol9`), initrd (`initrd-ol9.img`), stage2 URL (`http://192.168.56.10/ol9`), and Kickstart file (`ks-ol9.cfg`).
- For additional OSes, add more `label` sections with appropriate kernel, initrd, and URLs.
- Optionally, edit the file using `vim` or `nano`.
- Verify the menu:

  ```bash
  cat /var/lib/tftpboot/pxelinux.cfg/default
  ```

#### Step 6: Verify Services and Test

Ensure all services are running:

```bash
systemctl status dhcpd tftp.socket httpd
```

Test HTTP access for all OSes:

```bash
curl http://192.168.56.10/ol8/
curl http://192.168.56.10/ks.cfg
curl http://192.168.56.10/ol9/
curl http://192.168.56.10/ks-ol9.cfg
```

Test TFTP access for all kernel and initrd files:

```bash
tftp 192.168.56.10 -c get vmlinuz
tftp 192.168.56.10 -c get initrd.img
tftp 192.168.56.10 -c get vmlinuz-ol9
tftp 192.168.56.10 -c get initrd-ol9.img
```

Reboot the PXE server to confirm mount persistence:

```bash
reboot
# After reboot, check:
ls /var/www/html/ol8
ls /var/www/html/ol9
curl http://192.168.56.10/ol8/
curl http://192.168.56.10/ol9/
```

#### Step 7: Test PXE Client Boot

1. Power on the client VM with network boot enabled.
2. The PXE menu should display options for all OSes (e.g., “Install Oracle Linux 8” and “Install Oracle Linux 9”).
3. Select an OS or let it auto-boot to the default (Oracle Linux 8 after 10 seconds).
4. The client should:
   - Receive a DHCP IP from `192.168.56.100–200`.
   - Download `pxelinux.0`, `ldlinux.c32`, `menu.c32`, and the selected OS’s `vmlinuz` and `initrd`.
   - Fetch the Kickstart file and perform the unattended installation.
5. Monitor logs for issues:

   ```bash
   journalctl -u dhcpd -f
   journalctl -u tftp.socket -f
   journalctl -u httpd -f
   ```

6. Post-install, verify login credentials (e.g., `root/oracle123`, `oluser/oracle123`).

### Updated PXE Boot Menu Example

Here’s an example `/var/lib/tftpboot/pxelinux.cfg/default` for Oracle Linux 8, Oracle Linux 9, and a hypothetical CentOS Stream 9:

```bash
default menu.c32
prompt 0
timeout 100
ONTIMEOUT install-ol8

menu title Multi-OS PXE Boot Menu

label install-ol8
  menu label Install ^Oracle Linux 8
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=http://192.168.56.10/ol8 inst.ks=http://192.168.56.10/ks.cfg

label install-ol9
  menu label Install ^Oracle Linux 9
  kernel vmlinuz-ol9
  append initrd=initrd-ol9.img inst.stage2=http://192.168.56.10/ol9 inst.ks=http://192.168.56.10/ks-ol9.cfg

label install-centos9
  menu label Install ^CentOS Stream 9
  kernel vmlinuz-centos9
  append initrd=initrd-centos9.img inst.stage2=http://192.168.56.10/centos9 inst.ks=http://192.168.56.10/ks-centos9.cfg`

```

### Additional Notes

- **ISO Paths**: Ensure each ISO path is correct and persistent in `/etc/fstab`.
- **Kickstart Compatibility**: Different OSes may require specific Kickstart options or package groups. For example, CentOS Stream 9 uses `@graphical-server-environment` instead of `@Server with GUI`.
- **Storage Requirements**: Ensure the PXE server has enough disk space for multiple ISOs.
- **Client RAM**: Some OSes (e.g., Oracle Linux 9) may require more RAM (4 GB recommended) for GUI installation.
- **TFTP Load**: Serving multiple kernel and initrd files increases TFTP load; ensure the server’s performance is adequate.
- **Menu Customization**: Enhance the PXE menu with `menu.c32` or `vesamenu.c32` for better visuals or add submenus for organization if supporting many OSes.
