# Nagios Core Setup

## **Step 1: Install Nagios Core on `nms-server`**

In this example there will be three hosts on virtualbox or vmware.

### Prerequisites

- nms-server.joy.local [192.168.133.100]
- nms-host01.joy.local [192.168.133.101]
- nms-host02.joy.local [192.168.133.102]
- mail01.joy.local [192.168.133.103] --> This is a postfix relay server.

### **Step 1: Enable EPEL Repo on Oracle Linux 8**

On your `nms-server`:

```bash
sudo dnf install -y oracle-epel-release-el8
```

This provides the `ol8_developer_EPEL` repo which contains Nagios Core + plugins.

---

### **Step 2: Install Nagios and Plugins**

```bash
sudo dnf install -y nagios nagios-plugins-all nagios-plugins-nrpe nagios-plugins-disk nagios-plugins-load nagios-plugins-ping nagios-plugins-ssh nagios-plugins-http
```

⚠️ Even though `nagios-plugins-nrpe` is included, we don’t have to use NRPE (we'll rely on ping, SNMP).

---

### **Step 3: Start Services**

```bash
sudo systemctl enable nagios httpd
sudo systemctl start nagios httpd
```

---

### **Step 4: Configure Web Access**

Nagios web UI runs on Apache. Create a web user:

```bash
sudo htpasswd -c /etc/nagios/passwd nagiosadmin
```

Set password (example: `nagios123`).

Now go to:
👉 `http://192.168.133.100/nagios`
Login with `nagiosadmin / nagios123`.

---

### **Step 5: Add Hosts & Services**

Config files are in:

```text
/etc/nagios/objects/hosts.cfg
/etc/nagios/nagios.cfg
```

***Create and Edit `/etc/nagios/objects/hosts.cfg`.***

```bash

###############################################################################
# Host Definitions
###############################################################################

# Define first host: nms-host01
define host {
    use             linux-server     ; Template to inherit defaults from
    host_name       nms-host01       ; Short name (used internally in Nagios)
    alias           Oracle Linux Host 01 ; Friendly display name in UI
    address         192.168.133.101  ; IP address of the host
}

# Define second host: nms-host02
define host {
    use             linux-server
    host_name       nms-host02
    alias           Oracle Linux Host 02
    address         192.168.133.102
}
```

***Also include the path to `/etc/nagios/nagios.cfg`***

```bash
cfg_file=/usr/local/nagios/etc/objects/hosts.cfg
```

***Now Add basic hosts check on `/etc/nagios/objects/services.cfg`***

```bash
define service {
    use                     generic-service
    host_name               nms-host01
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

define service {
    use                     generic-service
    host_name               nms-host02
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

define service {
    use                     generic-service
    host_name               nms-host01
    service_description     SSH Port
    check_command           check_tcp!22
}

define service {
    use                     generic-service
    host_name               nms-host02
    service_description     SSH Port
    check_command           check_tcp!22
}
```

after adding **ping and TCP checks** as services.
configure the nagios cfg file with nagios with the following command:

```bash
sudo nagios -v /etc/nagios/nagios.cfg
```

Finally, restart:

```bash
sudo systemctl restart nagios
```

---


To be continued.....