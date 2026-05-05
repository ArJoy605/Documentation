# EVE-NG Native Console Setup on Ubuntu (Firefox + GNOME Terminal)

## 📌 Overview

This guide explains how to configure **EVE-NG Native Console** on Ubuntu so that clicking a device opens a terminal (gnome-terminal) automatically via `telnet://`.

---

## 🧰 Requirements

* Ubuntu (host OS)
* EVE-NG running (e.g., inside VMware)
* Firefox browser (recommended)
* `gnome-terminal` (preinstalled)
* `telnet` package

---

## 🔧 Step 1: Install Required Packages

```bash
sudo apt update
sudo apt install telnet xdg-utils
```

---

## 🔧 Step 2: Create Telnet Handler

Create a custom handler so Ubuntu knows how to open `telnet://` links.

```bash
nano ~/.local/share/applications/telnet.desktop
```

Paste the following:

```ini
[Desktop Entry]
Name=Telnet Handler
Exec=gnome-terminal -- bash -c "telnet %h %p"
Type=Application
NoDisplay=true
MimeType=x-scheme-handler/telnet;
```

Save and exit:

* `CTRL + O` → Enter
* `CTRL + X`

---

## 🔧 Step 3: Register the Handler

```bash
update-desktop-database ~/.local/share/applications/
xdg-mime default telnet.desktop x-scheme-handler/telnet
```

---

## 🔧 Step 4: Configure Firefox

1. Open Firefox
2. Go to:

   ```
   about:config
   ```
3. Search:

   ```
   network.protocol-handler.expose.telnet
   ```
4. Set it to:

   ```
   false
   ```

Restart Firefox completely.

---

## 🔧 Step 5: Test Native Console

1. Open EVE-NG in Firefox:

   ```
   https://<EVE-IP>
   ```
2. Start a device
3. Click:

   ```
   Console → Native
   ```

✅ Expected result:

* A **gnome-terminal window opens**
* Connects to device via telnet

---

## 🧪 Manual Test (Optional)

```bash
xdg-open telnet://192.168.70.70:32772
```

If terminal opens → setup is working correctly.

---

## 🔍 Troubleshooting

### ❌ Firefox shows “No apps available”

* Ensure `MimeType=x-scheme-handler/telnet;` is present
* Run:

  ```bash
  update-desktop-database ~/.local/share/applications/
  ```
* Restart Firefox

---

### ❌ Nothing happens on click

* Confirm telnet works manually:

  ```bash
  telnet <EVE-IP> <PORT>
  ```
* If works → issue is browser handler

---

### ❌ Chrome not working

* Chrome does **NOT support telnet://**
* Use:

  * Firefox (for Native console) ✅
  * HTML5 console (in Chrome) ✅

---

## 🔓 Exiting the Console

### Telnet session:

```
Ctrl + ]
quit
```

### Inside device:

```
exit
```

or

```
Ctrl + D
```

---

## 🧠 Notes

* Native console uses **telnet (unencrypted)** → safe in local lab
* For public environments → use VPN or secure access
* HTML5 console is a good fallback if native fails

---

## ✅ Summary

| Component              | Status |
| ---------------------- | ------ |
| telnet installed       | ✔      |
| handler created        | ✔      |
| Firefox configured     | ✔      |
| native console working | ✔      |

---

## 🎯 Result

Clicking **Native Console in EVE-NG** now:

* Opens GNOME Terminal
* Automatically connects to the device
* No external tools required

---

## 🚀 Optional Improvements

* Use **Tilix / Terminator** for tabbed consoles
* Customize terminal titles per device
* Use SSH instead of telnet where possible

---

**End of Document**
