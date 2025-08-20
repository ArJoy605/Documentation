# Postfix mail transfer One VM user to another VM user without DNS server

## **0️⃣ Prerequisites**

1. **Assign static IPs**:

   * Sender VM: `192.168.100.103` #any ip from the same subnet
   * Receiver VM: `192.168.100.10` #another ip from the same subnet
   * Also you need to set the hostname using

    ```bash
        sudo hostnamectl set-hostname testserver.joy.local ##your choice
    ```

1. **Verify connectivity**

```bash
ping 192.168.100.10
ping 192.168.100.103
```

1. **Configure `/etc/hosts` on both VMs**

```ini
127.0.0.1   localhost
192.168.100.103 testserver.joy.local testserver         #your choice
192.168.100.10  testserver2.joy.local testserver2       #your choice
```

---

## **1️⃣ Install Postfix and mailx**

```bash
sudo dnf install postfix mailx -y
```

---

## **2️⃣ Configure transport maps (for hostname-based delivery)**

**On Sender VM (`testserver`)**

```bash
sudo vi /etc/postfix/transport
```

Add:

```bash
testserver2.joy.local    smtp:[192.168.100.10]
```

Build transport database:

```bash
sudo postmap /etc/postfix/transport
```

**We need to do the same on Receiver VM just the ip will change (`testserver2`)** (for replies):

```bash
testserver.joy.local     smtp:[192.168.100.103]
```

---

## **3️⃣ Open SMTP port in firewall on both VMs**

```bash
sudo firewall-cmd --permanent --add-service=smtp
sudo firewall-cmd --reload
```

---

## **4️⃣ Configure `/etc/postfix/main.cf`**

### **Sender VM (`testserver`)**

```ini
myhostname = testserver.joy.local
mydomain = joy.local
myorigin = $mydomain

# Accept mail for local users
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Use IPv4 and listen on all interfaces
inet_protocols = ipv4
inet_interfaces = all

# Allow LAN relay
mynetworks = 127.0.0.0/8, 192.168.100.0/24

# Use transport map for hostname to IP
transport_maps = hash:/etc/postfix/transport
```

### **Receiver VM (`testserver2`)**

```ini
myhostname = testserver2.joy.local
mydomain = joy.local
myorigin = $mydomain

mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

inet_protocols = ipv4
inet_interfaces = all

# Allow LAN relay for sending replies
mynetworks = 127.0.0.0/8, 192.168.100.0/24

# Use transport map for sending back by hostname
transport_maps = hash:/etc/postfix/transport
```

---

## **5️⃣ Start and enable Postfix on both VMs**

```bash
sudo systemctl enable --now postfix
sudo systemctl status postfix
```

---

## **6️⃣ Send mail [Example]**

You need to create users and test the mail transfer.

### **Option 1: Using hostname with transport map**

```bash
echo "Hello Roy" | mail -s "LAN test" roy@testserver2.joy.local
```

### **Option 2: Using direct IP (simplest)**

```bash
echo "Hello Roy" | mail -s "LAN test" "roy@[192.168.100.10]"
```

---

## **7️⃣ Check mail on receiver on another VM**

```bash
mail
```

* Delete messages with `d <number>` or `d *`
* Quit with `q`

---

## **8️⃣ Troubleshooting**

| Issue                                                | Check / Fix                                                        |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| Mail stuck in queue: `Host or domain name not found` | Use IP in brackets or configure transport map; check `/etc/hosts`. |
| Cannot connect to SMTP                               | `nc -vz 192.168.100.10 25`; ensure firewall allows port 25.        |
| Queue not delivering                                 | Clear old mails: `sudo postsuper -d ALL`; restart Postfix.         |
| User cannot read mail                                | Check `/var/spool/mail/<user>` permissions; use `mail` command.    |
| Sender cannot relay                                  | Check `mynetworks` includes LAN subnet.                            |

---

## **✅ Notes**

* Using **transport maps** allows typing hostnames but still bypasses DNS/MX.
* Direct IP (`[IP]`) always works, simplest for a lab.

---
