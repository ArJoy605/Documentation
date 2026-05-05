# EVE-NG Native Console Setup on Ubuntu (Fixed Telnet Handler)

![Platform](https://img.shields.io/badge/Platform-Ubuntu-E95420?logo=ubuntu\&logoColor=white)
![EVE-NG](https://img.shields.io/badge/EVE--NG-Community-blue)
![Browser](https://img.shields.io/badge/Browser-Firefox-FF7139?logo=firefox\&logoColor=white)
![Status](https://img.shields.io/badge/Status-Working-success)

---

## 📌 Overview

This guide shows how to properly configure **EVE-NG Native Console on Ubuntu** using Firefox and GNOME Terminal.

✅ Fixes:

* “No apps available” error
* `telnet>` prompt issue (wrong parsing)
* Proper direct connection to device console

---

## 🧰 Requirements

* Ubuntu (host OS)
* EVE-NG (VMware or bare-metal)
* Firefox browser
* gnome-terminal (default)
* telnet

---

## 🔧 Step 1: Install Required Packages

```bash
sudo apt update
sudo apt install telnet xdg-utils
```

---

## 🔧 Step 2: Create Telnet Handler Script (IMPORTANT FIX)

This solves the issue where telnet opens but **does not connect automatically**.

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/eve-telnet-handler
```

Paste:

```bash
#!/bin/bash

url="$1"
host=$(echo "$url" | sed -E 's#telnet://([^:]+):([0-9]+).*#\1#')
port=$(echo "$url" | sed -E 's#telnet://([^:]+):([0-9]+).*#\2#')

gnome-terminal -- bash -c "telnet $host $port; exec bash"
```

Make executable:

```bash
chmod +x ~/.local/bin/eve-telnet-handler
```

---

## 🔧 Step 3: Create Desktop Handler

```bash
nano ~/.local/share/applications/telnet.desktop
```

Paste:

```ini
[Desktop Entry]
Name=Telnet Handler
Exec=/home/YOUR_USERNAME/.local/bin/eve-telnet-handler %u
Type=Application
NoDisplay=true
MimeType=x-scheme-handler/telnet;
```

⚠️ Replace:

```
YOUR_USERNAME
```

---

## 🔧 Step 4: Register Handler

```bash
update-desktop-database ~/.local/share/applications/
xdg-mime default telnet.desktop x-scheme-handler/telnet
```

---

## 🔧 Step 5: Configure Firefox

1. Open Firefox
2. Go to:

   ```
   about:config
   ```
3. Search:

   ```
   network.protocol-handler.expose.telnet
   ```
4. Set:

   ```
   false
   ```

Restart Firefox.

---

## 🔧 Step 6: Test Native Console

### In EVE-NG:

* Start a node
* Click:

  ```
  Console → Native
  ```

### ✅ Expected Result:

* GNOME Terminal opens
* Automatically connects to device
* No `telnet>` prompt

---

## 🧪 Manual Test

```bash
xdg-open telnet://192.168.70.70:32772
```

---

## 📸 Screenshots

### 🔹 Before Fix (Issue)

![Error](screenshots/error-no-app.png)

### 🔹 Wrong Behavior (telnet>)

![Telnet Prompt](screenshots/telnet-prompt.png)

### 🔹 After Fix (Working)

![Working](screenshots/native-console-working.png)

---

## 🔍 Troubleshooting

### ❌ “No apps available”

* Ensure `.desktop` file starts with:

  ```
  [Desktop Entry]
  ```
* Run:

  ```bash
  update-desktop-database ~/.local/share/applications/
  ```

---

### ❌ “Desktop file didn’t specify Exec field”

* Fix `.desktop` syntax
* Ensure:

  ```
  Exec=/home/USER/.local/bin/eve-telnet-handler %u
  ```

---

### ❌ Telnet opens but doesn’t connect

* Use the **script method (Step 2)**
* This is the correct fix

---

### ❌ Chrome not working

* Chrome ❌ (no telnet support)
* Firefox ✅
* HTML5 console ✅

---

## 🔓 Exit Console

Telnet session:

```
Ctrl + ]
quit
```

Inside device:

```
exit
```

or

```
Ctrl + D
```

---

## 🧠 Notes

* Telnet is unencrypted → safe for local lab only
* Do not expose EVE-NG directly to internet
* Use VPN for remote access

---

## 📂 Suggested Repo Structure

```
eve-ng-native-console/
├── README.md
├── scripts/
│   └── eve-telnet-handler
└── screenshots/
    ├── error-no-app.png
    ├── telnet-prompt.png
    └── native-console-working.png
```

---

## 🚀 Optional Improvements

* Use **Tilix / Terminator** for tabs
* Auto-name terminal windows per device
* Replace telnet with `socat` or `netcat`

---

## 🎯 Result

Clicking **Native Console in EVE-NG** now:

* Opens GNOME Terminal
* Directly connects to device
* Works reliably on Ubuntu

---

## 📜 License

MIT
