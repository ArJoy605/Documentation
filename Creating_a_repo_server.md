# Creating a Repo Server

Nice — below is a **detailed, step-by-step lab guide** you can follow (or copy/paste) to reproduce your repo server (`Server-01`) and client (`web-01`) setup. I’ll assume:

* You already copied the ISO to the repo server (example path: `/home/joy/Downloads/OracleLinux-R8-U10-x86_64-dvd.iso`)
* SELinux and firewalld are disabled for the test (you said they are disabled)
* You want to serve the ISO over HTTP **without duplicating the ISO file** (we’ll use loop + bind mount)

Follow each step on the appropriate machine.

---

## Server-01 (repo server) — make ISO available over HTTP

1. **Create mount points**

```bash
sudo mkdir -p /mnt/localrepo
sudo mkdir -p /var/www/html/localrepo
```

1. **Mount the ISO with loop (use the actual full path to the ISO)**

> Replace `/home/joy/Downloads/OracleLinux-...iso` with the real full path on your machine.

```bash
sudo mount -o loop,ro /home/joy/Downloads/OracleLinux-R8-U10-x86_64-dvd.iso /mnt/localrepo
# verify:
ls -l /mnt/localrepo
df -h /mnt/localrepo
```

If you get a loop device error, see Troubleshooting section below.

1. **Bind mount into Apache webroot (no copy, no extra space)**

```bash
sudo mount --bind /mnt/localrepo /var/www/html/localrepo
ls -l /var/www/html/localrepo
```

1. **(Optional) Make mounts persistent in `/etc/fstab`**
   Open `/etc/fstab` with `sudo vim /etc/fstab` and add **two** lines (use the actual ISO path):

```bash
/home/joy/Downloads/OracleLinux-R8-U10-x86_64-dvd.iso  /mnt/localrepo  iso9660  loop,ro  0  0
/mnt/localrepo  /var/www/html/localrepo  none  bind  0  0
```

Then reload systemd mounts and try them:

```bash
sudo systemctl daemon-reload
sudo mount -a
# check mounts:
mount | grep localrepo
```

> **Important:** the ISO loop mount line must appear **before** the bind mount line (order in `/etc/fstab` matters for `mount -a`).

1. **Install and start Apache**

```bash
sudo yum install -y httpd        # or dnf install -y httpd
sudo systemctl enable --now httpd
# check Apache:
sudo systemctl status httpd
```

1. **Configure Apache to serve the repo cleanly (optional but helpful)**
   Create `/etc/httpd/conf.d/localrepo.conf`:

```apache
Alias /localrepo /var/www/html/localrepo

<Directory /var/www/html/localrepo>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

Then reload:

```bash
sudo systemctl restart httpd
```

1. **Permissions (if SELinux were enabled)**
   Since SELinux is disabled for your tests, you can skip this. If later you re-enable SELinux, run:

```bash
sudo chown -R root:root /var/www/html/localrepo
sudo chmod -R a+rX /var/www/html/localrepo
sudo chcon -R -t httpd_sys_content_t /var/www/html/localrepo
```

1. **Test from the repo server (HTTP)**

```bash
curl -I http://localhost/localrepo/        # check headers
curl http://localhost/localrepo/ | head -n 50  # list some output
# or from another machine:
curl -I http://<Server-01-IP>/localrepo/
```

---

## web-01 (client) — point to the repo and install `httpd`

1. **Make sure network connectivity**

```bash
ping -c 3 <Server-01-IP>
# or if you added hosts: ping -c 3 webserver.joy.local
```

1. **Create local repo file** — create `/etc/yum.repos.d/local-ol8.repo` with these stanzas:

```ini
[ol8_BaseOS]
name=OL8 BaseOS - Local
baseurl=http://<Server-01-IP>/localrepo/BaseOS/
enabled=1
gpgcheck=0

[ol8_AppStream]
name=OL8 AppStream - Local
baseurl=http://<Server-01-IP>/localrepo/AppStream/
enabled=1
gpgcheck=0
```

> Replace `<Server-01-IP>` with the repo server IP or hostname (e.g., `webserver.joy.local` if `/etc/hosts` is set).

1. **Clean cache & list repos**

```bash
sudo yum clean all
sudo yum repolist
```

You should see `ol8_BaseOS` and `ol8_AppStream` (or at least `localrepo` entries). If not, check the URL in a browser/curl from the client.

1. **Install httpd from the local repo**

```bash
sudo yum install -y httpd
# verify:
rpm -q httpd
```

If `yum` fails saying package not found, try adjusting baseurl (see Troubleshooting).

---

## Quick common checks & helpful commands

* Show mounted loop devices:

```bash
mount | grep /mnt/localrepo
losetup -a
```

* Show Apache logs:

```bash
sudo tail -n 100 /var/log/httpd/error_log
sudo tail -n 100 /var/log/httpd/access_log
```

* List repodata to ensure repo metadata exists:

```bash
ls -l /var/www/html/localrepo/BaseOS/repodata
ls -l /var/www/html/localrepo/AppStream/repodata
```

---

## Troubleshooting

### 1. `failed to setup loop device` when running `mount -a`

* Cause: `/etc/fstab` line used `/path/to/OracleLinux.iso` placeholder or wrong path, or mount was attempted before the file existed.
* Fix:

  * Make sure the ISO path exists and is correct (use absolute path).
  * Test mounting manually first:

    ```bash
    sudo mount -o loop,ro /home/joy/Downloads/OracleLinux-R8-U10-x86_64-dvd.iso /mnt/localrepo
    ```

  * If still fails, try using `losetup`:

    ```bash
    sudo losetup -f --show /home/joy/Downloads/OracleLinux-...iso
    # suppose it prints /dev/loop0
    sudo mount -o ro /dev/loop0 /mnt/localrepo
    ```

### 2. `yum repolist` doesn’t show packages / `No more mirrors` or 404 errors

* Check the exact URL from the client:

  ```bash
  curl -I http://<Server-01-IP>/localrepo/BaseOS/
  curl -I http://<Server-01-IP>/localrepo/AppStream/
  ```

* Confirm `repodata` exists under those directories on server:

  ```bash
  ls -l /var/www/html/localrepo/BaseOS/repodata
  ```
  
* If the ISO structure differs (some ISOs have `Packages/` at root but `repodata` under separate subdirs), adapt your baseurl to the correct subpath or use the root `/localrepo/`.

### 3. Apache returns 403 Forbidden

* Check `/var/log/httpd/error_log`.
* Ensure permissions:

  ```bash
  sudo chmod -R a+rX /var/www/html/localrepo
  sudo chown -R root:root /var/www/html/localrepo
  ```

* If SELinux is enabled, restore correct type:

  ```bash
  sudo chcon -R -t httpd_sys_content_t /var/www/html/localrepo
  ```

### 4. Bind mount disappears after reboot

* Ensure both fstab entries exist and use full paths, and the ISO line is above the bind line.
* Example order in `/etc/fstab`:

```bash
/home/joy/Downloads/OracleLinux-R8-U10-x86_64-dvd.iso  /mnt/localrepo  iso9660  loop,ro  0  0
/mnt/localrepo  /var/www/html/localrepo  none  bind  0  0
```

* Then:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

---

## Optional: create a proper local mirror (if you want to update packages later)

If you want a syncable repo (not just DVD contents) so Server-01 can pull updates from internet repos:

* Install `yum-utils` and `createrepo` on Server-01:

```bash
sudo yum install -y yum-utils createrepo
```

* Use `reposync` to download packages to `/var/www/html/localrepo/` then `createrepo` to generate metadata:

```bash
sudo reposync -p /var/www/html/localrepo --repoid=ol8_baseos_latest
sudo createrepo /var/www/html/localrepo
```

(You will need network access and valid repo ids — this is optional and for a different use-case.)

---

## Final verification checklist

On Server-01:

* `mount | grep localrepo` → shows both ISO and bind mount
* `curl -I http://localhost/localrepo/` → returns HTTP 200
* `systemctl status httpd` → running

On web-01:

* `ping <Server-01-IP>` → reachable
* `yum repolist` → shows local repos
* `yum install -y httpd` → installs from local repo

---

If you want, I can:

* produce a single shell script that automates all Server-01 steps (mount, fstab, bind, install httpd, create apache conf),
* or walk you through the exact commands using the exact ISO path on your machine (paste your ISO path and server IP and I’ll generate the exact commands). Which do you prefer?
