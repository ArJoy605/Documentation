# 🔑 VMware Workstation Installation & Secure Boot Module Signing (Ubuntu)

---

## 1. Install Dependencies

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r)
```

---

## 2. Download & Install VMware

* Get the **VMware Workstation bundle** from Broadcom/VMware site.
* Example (v17.6.1):

```bash
chmod +x VMware-Workstation-Full-17.6.1-24319023.x86_64.bundle
sudo ./VMware-Workstation-Full-17.6.1-24319023.x86_64.bundle
```

➡️ If **Secure Boot is enabled**, you’ll see errors about `vmmon` and `vmnet` being unsigned.

---

## 3. Generate & Register MOK Key

1. **Generate key pair (one-liner):**

```bash
sudo openssl req -new -x509 -newkey rsa:2048 -keyout ~/MOK.priv -outform DER -out ~/MOK.der -nodes -days 36500 -subj "/CN=VMware/"
```

### 🔎 Notes

* You can change the path `~/MOK.priv` and `~/MOK.der` if needed.

1. **Register the public key:**

```bash
sudo mokutil --import ~/MOK.der
```

* Choose a password.
* Reboot → In UEFI menu, **enroll the key**.

---

## 4. Sign VMware Kernel Modules

Sign both `vmmon` and `vmnet`:

```bash
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ~/MOK.priv ~/MOK.der $(modinfo -n vmmon)
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ~/MOK.priv ~/MOK.der $(modinfo -n vmnet)
```

### 🔎 Note

* You have to change the path `~/MOK.priv` and `~/MOK.der` if you have generated them in a different location.
* `$(modinfo -n vmmon)` → automatically gives the full path to `vmmon.ko`
* `$(modinfo -n vmnet)` → automatically gives the full path to `vmnet.ko`
* `uname -r` → expands to your **current running kernel version**

---

## 5. After VMware Upgrade / Kernel Upgrade

1. Reinstall/upgrade VMware bundle:

```bash
chmod +x VMware-Workstation-Full-*.bundle
sudo ./VMware-Workstation-Full-*.bundle
```

1. **Re-sign modules** (reuse the same MOK key):

```bash
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ~/MOK.priv ~/MOK.der $(modinfo -n vmmon)
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ~/MOK.priv ~/MOK.der $(modinfo -n vmnet)
```

### 🔎Note

* You have to change the path `~/MOK.priv` and `~/MOK.der` if you have generated them in a different location.

---

✅ **Summary Workflow**

1. Install headers + VMware.
2. Generate MOK key → Import → Enroll in UEFI.
3. Sign `vmmon` & `vmnet`.
4. After every **kernel or VMware update**, re-run the signing commands.

---
