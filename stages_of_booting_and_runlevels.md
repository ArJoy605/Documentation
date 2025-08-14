# **stages of booting** in a typical Linux system

##

---

### 1. **Power-On and Firmware Initialization (BIOS/UEFI)**

* When you power on the machine, the firmware (BIOS or UEFI) initializes hardware components.
* Performs POST (Power-On Self Test) to check hardware status.
* Firmware locates and loads the bootloader from the configured boot device.

---

### 2. **Bootloader Stage**

* The bootloader (like GRUB) is loaded from MBR (BIOS) or EFI partition (UEFI).
* Bootloader presents a menu to select the OS/kernel version (optional).
* Loads the Linux kernel and initramfs (initial RAM filesystem) into memory.

---

### 3. **Kernel Initialization**

* The Linux kernel initializes system hardware using drivers.
* Mounts the initramfs, which contains essential tools and drivers needed to mount the real root filesystem.
* Kernel mounts the real root filesystem specified by the bootloader.

---

### 4. **Init/Systemd Process Startup**

* Kernel starts the first process: `init` (SysV init) or more commonly `systemd` in modern systems.
* `systemd` initializes the system by starting necessary services, mounting filesystems, setting up network, etc.

---

### 5. **Runlevel/Target and Service Startup**

* System enters a specific runlevel or systemd target (e.g., multi-user.target, graphical.target).
* Starts background services like networking, graphical display manager, etc.
* Provides login prompt or graphical login screen.

---

### 6. **User Login**

* User logs in via console or GUI.
* User shell or desktop environment starts.

---

### Runlevels vs systemd Targets

---

### Runlevels vs systemd Targets (Summary)

| Traditional Runlevel | Description                          | systemd Target                  | Description                        |
| -------------------- | ------------------------------------ | ------------------------------- | ---------------------------------- |
| 0                    | Halt (shutdown)                      | poweroff.target                 | System shutdown                    |
| 1                    | Single-user mode (maintenance)       | rescue.target                   | Single-user mode                   |
| 2                    | Multi-user, no network (rarely used) | multi-user.target (without GUI) | Multi-user, non-graphical          |
| 3                    | Multi-user with networking           | multi-user.target               | Multi-user, text mode with network |
| 4                    | Undefined/user-definable             | (custom targets)                | User-defined/custom targets        |
| 5                    | Multi-user with networking + GUI     | graphical.target                | Multi-user, graphical desktop      |
| 6                    | Reboot                               | reboot.target                   | System reboot                      |

---

* **systemctl get-default** shows the current default target.
* **systemctl isolate \[target]** switches to the specified target immediately.
* systemd targets are more flexible and can be extended or customized easily.
