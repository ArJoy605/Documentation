# Cisco IOS-XE gNMI/MDT Setup with mTLS + NACM

**Device:** ios-xe-lab (Cisco Catalyst 9000v, IOS-XE 17.10)  
**Monitoring Server:** prometheus.prosmox.local  
**Purpose:** Secure gNMI telemetry collection with read-only enforcement via NACM

---

## Architecture Overview

```
Prometheus Server (10.100.100.210)
├── gnmic service  →  gNMI Subscribe (MDT)  →  Cisco IOS-XE (10.100.100.19:9339)
│     └── mTLS (client.crt) + username/password (gnmi_client)
├── Prometheus  ←  scrapes  ←  gnmic :9804/metrics
└── Grafana  ←  queries  ←  Prometheus :9000

Security Layers:
1. mTLS   - client cert signed by rootCA, encrypts transport
2. Password Auth  - gnxi secure-password-auth, identifies user for NACM
3. NACM   - gnmi_client (priv 1) → read + subscribe only, writes denied
```

---

## Step 1: Certificate Generation

### 1.1 Create Root CA

```bash
openssl genrsa -out rootCA.key 2048
openssl req -subj /C=/ST=/L=/O=/CN=rootCA \
  -x509 -new -nodes \
  -key rootCA.key \
  -sha256 \
  -out rootCA.pem
```

### 1.2 Create Device Certificate (installed on the switch)

The device cert CN must match the `tls-server-name` used by gnmic clients.

```bash
openssl genrsa -out device.key 2048

openssl req -subj /C=/ST=/L=/O=/CN=gnmi-crt \
  -new -key device.key \
  -out device.csr

# Sign with SAN for switch IP and DNS name
openssl x509 -req \
  -in device.csr \
  -CA rootCA.pem \
  -CAkey rootCA.key \
  -CAcreateserial \
  -out device.crt \
  -days 1095 \
  -sha256 \
  -extfile <(printf "subjectAltName=DNS:cisco-device,IP:10.100.100.19")
```

### 1.3 Export Device Cert to PKCS12 (required for IOS-XE import)

```bash
openssl pkcs12 -export \
  -out mycert.pfx \
  -inkey device.key \
  -in device.crt \
  -certfile rootCA.pem \
  -aes128
# Export Password: cisco  (used during IOS-XE import)
```

### 1.4 Create Client Certificate (CN must match local username on switch)

```bash
openssl genrsa -out client.key 2048

# CN=gnmi_client must match the username configured on the switch
openssl req -subj /C=/ST=/L=/O=/CN=gnmi_client \
  -new -key client.key \
  -out client.csr

openssl x509 -req \
  -in client.csr \
  -CA rootCA.pem \
  -CAkey rootCA.key \
  -CAcreateserial \
  -out client.crt \
  -sha256
```

### Certificate Files Summary

| File | Purpose | Location |
|------|---------|----------|
| `rootCA.pem` | CA certificate | Switch trustpoint + gnmic |
| `rootCA.key` | CA private key | Secure storage only |
| `device.crt` | Switch TLS cert | Switch (via PKCS12) |
| `device.key` | Switch TLS key | Switch (via PKCS12) |
| `mycert.pfx` | PKCS12 bundle for IOS-XE | Copied to switch |
| `client.crt` | gnmic client cert | `/etc/gnmic/certificates/` |
| `client.key` | gnmic client key | `/etc/gnmic/certificates/` |

---

## Step 2: IOS-XE Certificate Import

### 2.1 Copy PKCS12 to Switch

```bash
# From the Linux server, copy the PFX and CA cert to the switch
scp mycert.pfx admin@10.100.100.19:flash:
scp rootCA.pem admin@10.100.100.19:flash:
```

### 2.2 Import CA Certificate (rootCA) into Trustpoint

```
ios-xe-lab# configure terminal

ios-xe-lab(config)# crypto pki trustpoint GNMI-TP
ios-xe-lab(ca-trustpoint)#  enrollment pkcs12
ios-xe-lab(ca-trustpoint)#  revocation-check none
ios-xe-lab(ca-trustpoint)#  exit

ios-xe-lab(config)# crypto pki import GNMI-TP pkcs12 flash:mycert.pfx password cisco
```

### 2.3 Verify Certificate Import

```
ios-xe-lab# show crypto pki certificates GNMI-TP
ios-xe-lab# show crypto pki trustpoints
```

---

## Step 3: Configure AAA

```
ios-xe-lab(config)# aaa new-model
ios-xe-lab(config)# aaa authentication login default local
ios-xe-lab(config)# aaa authorization exec default local
```

---

## Step 4: Configure gNXI (gNMI Server)

```
ios-xe-lab(config)# gnxi
ios-xe-lab(config)# gnxi secure-server
ios-xe-lab(config)# gnxi server
ios-xe-lab(config)# gnxi secure-trustpoint GNMI-TP
ios-xe-lab(config)# gnxi secure-client-auth
ios-xe-lab(config)# gnxi secure-password-auth
ios-xe-lab(config)# netconf-yang
```

**What each command does:**

| Command | Purpose |
|---------|---------|
| `gnxi secure-server` | Enables TLS gNMI on port 9339 |
| `gnxi server` | Enables insecure gNMI on port 50052 |
| `gnxi secure-trustpoint GNMI-TP` | Sets the device cert trustpoint |
| `gnxi secure-client-auth` | Requires client to present a certificate (mTLS) |
| `gnxi secure-password-auth` | Requires username+password in gRPC metadata |
| `netconf-yang` | Enables NETCONF (required for NACM) |

> **Important:** Both `secure-client-auth` and `secure-password-auth` must be enabled together. The client cert handles TLS transport authentication; the password handles user identity for NACM enforcement.

### Verify gNXI State

```
ios-xe-lab# show gnxi state detail
```

Expected output:
```
Secure server: Enabled
Secure server port: 9339
Secure client authentication: Enabled
Secure password authentication: Enabled
Secure trustpoint: GNMI-TP
```

---

## Step 5: Configure Local Users

```
! Admin user - full privilege, used for device management and NACM administration
ios-xe-lab(config)# username joy privilege 15 password 0 Asdf@1234
ios-xe-lab(config)# username manzil privilege 15 password 0 Asdf@1234

! Monitoring user - privilege 1, NACM will enforce read-only
! No explicit privilege keyword = defaults to 1
ios-xe-lab(config)# username gnmi_client secret 9 <hashed-password>
```

> **Key Point:** `gnmi_client` must NOT be privilege 15. Privilege 15 users are automatically in the `PRIV15` NACM group which has a hardcoded permit-all rule that overrides any custom NACM deny rules.

---

## Step 6: Apply NACM Policy

NACM can only be configured via NETCONF or gNMI — there is no CLI command.  
Use the `joy` admin user (privilege 15) to apply this, since the policy will restrict `gnmi_client`.

```bash
gnmic -a 10.100.100.19:9339 \
  --tls-ca rootCA.pem \
  --tls-cert client.crt \
  --tls-key client.key \
  --tls-server-name "cisco-device" \
  --username joy \
  --password Asdf@1234 \
  set \
  --update-path 'rfc7951:/ietf-netconf-acm:nacm' \
  --update-value '{
    "ietf-netconf-acm:enable-nacm": true,
    "ietf-netconf-acm:read-default": "permit",
    "ietf-netconf-acm:write-default": "deny",
    "ietf-netconf-acm:exec-default": "deny",
    "ietf-netconf-acm:groups": {
      "group": [
        {
          "name": "MONITORING",
          "user-name": ["gnmi_client"]
        }
      ]
    },
    "ietf-netconf-acm:rule-list": [
      {
        "name": "monitoring-rules",
        "group": ["MONITORING"],
        "rule": [
          {
            "name": "allow-read",
            "module-name": "*",
            "access-operations": "read",
            "action": "permit"
          },
          {
            "name": "permit-subscriptions",
            "module-name": "*",
            "access-operations": "exec",
            "action": "permit"
          },
          {
            "name": "deny-writes",
            "module-name": "*",
            "access-operations": "create update delete",
            "action": "deny"
          }
        ]
      }
    ]
  }'
```

### NACM Rule Explanation

| Rule | Operations | Action | Effect |
|------|-----------|--------|--------|
| `allow-read` | read | permit | gNMI GET and MDT data reads work |
| `permit-subscriptions` | exec | permit | gNMI Subscribe / MDT streaming works |
| `deny-writes` | create, update, delete | deny | Cannot modify any configuration |
| `exec-default: deny` | exec (catch-all) | deny | Any exec not explicitly permitted is blocked |
| `write-default: deny` | write (catch-all) | deny | Safety net for all writes |

### Revert NACM (if needed)

Use `joy` or any other admin user that is NOT in the MONITORING group:

```bash
# Disable NACM entirely
gnmic -a 10.100.100.19:9339 \
  --tls-ca rootCA.pem \
  --tls-cert client.crt \
  --tls-key client.key \
  --tls-server-name "cisco-device" \
  --username joy \
  --password Asdf@1234 \
  set \
  --delete 'rfc7951:/ietf-netconf-acm:nacm'

# Or reset to IOS-XE defaults via CLI on the device
ios-xe-lab# request platform software yang-management nacm reset-config
```

---

## Step 7: Test NACM Enforcement

### Test 1: Write should be DENIED for gnmi_client

```bash
gnmic -a 10.100.100.19:9339 \
  --tls-ca rootCA.pem \
  --tls-cert client.crt \
  --tls-key client.key \
  --tls-server-name "cisco-device" \
  --username gnmi_client \
  --password <password> \
  set \
  --update-path 'rfc7951:/Cisco-IOS-XE-native:native/interface/GigabitEthernet[name=1/0/8]' \
  --update-value '{"Cisco-IOS-XE-native:mtu": 1600}'
```

Expected: `access denied`

### Test 2: Read should be PERMITTED for gnmi_client

```bash
gnmic -a 10.100.100.19:9339 \
  --tls-ca rootCA.pem \
  --tls-cert client.crt \
  --tls-key client.key \
  --tls-server-name "cisco-device" \
  --username gnmi_client \
  --password <password> \
  subscribe \
  --path 'rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics' \
  --mode once
```

Expected: Returns interface statistics data.

### Test 3: Write should SUCCEED for joy (admin)

```bash
gnmic -a 10.100.100.19:9339 \
  --tls-ca rootCA.pem \
  --tls-cert client.crt \
  --tls-key client.key \
  --tls-server-name "cisco-device" \
  --username joy \
  --password Asdf@1234 \
  set \
  --update-path 'rfc7951:/Cisco-IOS-XE-native:native/interface/GigabitEthernet[name=1/0/8]' \
  --update-value '{"Cisco-IOS-XE-native:mtu": 1600}'
```

Expected: Success (joy is priv 15, not in MONITORING group, NACM does not restrict).

---

## Step 8: gnmic Service Configuration

### 8.1 Configuration File `/etc/gnmic/gnmic.yaml`

```yaml
targets:
  ios-xe-lab:
    address: 10.100.100.19:9339
    timeout: 60s
    username: gnmi_client
    password: <gnmi_client_password>
    tls-ca: /etc/gnmic/certificates/rootCA.pem
    tls-cert: /etc/gnmic/certificates/client.crt
    tls-key: /etc/gnmic/certificates/client.key
    tls-server-name: cisco-device
    skip-verify: false
    subscriptions:
      - ifstats
      - cpustats
      - memstats
      - tempstats
      - sysstats
    outputs:
      - prom-out

subscriptions:
  ifstats:
    paths:
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/mtu
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/speed
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/admin-status
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/oper-status
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/if-index
      - rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/interface/interface-type
    mode: stream
    stream-mode: sample
    sample-interval: 10s

  cpustats:
    paths:
      - rfc7951:/Cisco-IOS-XE-process-cpu-oper:cpu-usage/cpu-utilization/five-seconds
      - rfc7951:/Cisco-IOS-XE-process-cpu-oper:cpu-usage/cpu-utilization/one-minute
      - rfc7951:/Cisco-IOS-XE-process-cpu-oper:cpu-usage/cpu-utilization/five-minutes
    mode: stream
    stream-mode: sample
    sample-interval: 10s

  memstats:
    paths:
      - rfc7951:/Cisco-IOS-XE-memory-oper:memory-statistics/memory-statistic/total-memory
      - rfc7951:/Cisco-IOS-XE-memory-oper:memory-statistics/memory-statistic/used-memory
      - rfc7951:/Cisco-IOS-XE-memory-oper:memory-statistics/memory-statistic/free-memory
      - rfc7951:/Cisco-IOS-XE-memory-oper:memory-statistics/memory-statistic/highest-usage
      - rfc7951:/Cisco-IOS-XE-memory-oper:memory-statistics/memory-statistic/lowest-usage
    mode: stream
    stream-mode: sample
    sample-interval: 10s

  tempstats:
    paths:
      - rfc7951:/Cisco-IOS-XE-platform-oper:components/component/state/temp
    mode: stream
    stream-mode: sample
    sample-interval: 10s

  sysstats:
    paths:
      - openconfig:/system/state/boot-time
    mode: stream
    stream-mode: sample
    sample-interval: 10s

processors:
  drop-discontinuity:
    event-delete:
      value-names:
        - '.*discontinuity-time$'

outputs:
  prom-out:
    type: prometheus
    listen: :9804
    path: /metrics
    metric-prefix: gnmic
    append-subscription-name: false
    strings-as-labels: true
    expiration: 60s
    event-processors:
      - drop-discontinuity
```

### 8.2 Systemd Service `/etc/systemd/system/gnmic.service`

```ini
[Unit]
Description=gNMIc Telemetry Collector
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=gnmic
Group=gnmic
ExecStart=/usr/local/bin/gnmic --config /etc/gnmic/gnmic.yaml subscribe
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 8.3 Certificate Permissions

The gnmic service runs as the `gnmic` user — certificates must be readable by it:

```bash
chown -R gnmic:gnmic /etc/gnmic/
chmod 640 /etc/gnmic/certificates/*
chmod 750 /etc/gnmic/certificates/
```

### 8.4 Enable and Start Service

```bash
systemctl daemon-reload
systemctl enable gnmic
systemctl start gnmic
systemctl status gnmic
```

---

## Step 9: Prometheus Integration

### 9.1 Add gnmic Scrape Target to `/etc/prometheus/prometheus.yml`

```yaml
scrape_configs:
  - job_name: 'gnmic'
    static_configs:
      - targets: ['localhost:9804']
    scrape_interval: 15s
```

### 9.2 Reload Prometheus

```bash
curl -X POST http://localhost:9000/-/reload
```

### 9.3 Verify Data Flow

```bash
# Check gnmic metrics endpoint
curl http://localhost:9804/metrics | head -30

# Check Prometheus target health
curl -s "http://localhost:9000/api/v1/targets" | python3 -m json.tool | grep -E "gnmic|health"

# Query a metric directly
curl -s "http://localhost:9000/api/v1/query?query=gnmic_rfc7951_Cisco_IOS_XE_process_cpu_oper_cpu_usage_cpu_utilization_five_seconds"
```

---

## Troubleshooting Reference

### Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `CONFD_ERRCODE_INCONSISTENT_VALUE` | Value type wrong or interface constraint | Check value is integer not string; check interface mode |
| `unexpected prefix [Cisco-IOS-XE-native]` | Wrong path format for encoding | Use `rfc7951:/Cisco-IOS-XE-native:...` prefix |
| `Cannot find the username` | `secure-password-auth` enabled but no `--username` | Add `--username` flag |
| `Cannot find the password` | Username provided but no `--password` | Add `--password` flag |
| `authentication handshake failed: EOF` | `secure-client-auth` enabled but no client cert | Add `--tls-cert` and `--tls-key` |
| `access denied` | NACM write-deny rule blocking operation | Use admin user (joy/manzil) for writes |
| `nacm group` invalid CLI command | NACM cannot be configured via CLI | Use gNMI/NETCONF to configure NACM |
| NACM rules not enforced | User is privilege 15 (PRIV15 bypasses NACM) | Lower user to priv 1 |
| gnmic service metrics empty | Certificate permission denied for gnmic user | `chown -R gnmic:gnmic /etc/gnmic/` |

### Key gNMI Path Format for Cisco IOS-XE Native Models

```
# Correct format for Cisco Native YANG models:
rfc7951:/Cisco-IOS-XE-native:native/...

# Correct format for Cisco Operational models:
rfc7951:/Cisco-IOS-XE-interfaces-oper:interfaces/...

# Correct format for OpenConfig models:
openconfig:/system/state/...

# JSON value body - namespace prefix required on leaf:
{"Cisco-IOS-XE-native:mtu": 9000}   ← integer, not string
```

### MTU Configuration Notes

- Per-port MTU requires IOS-XE 17.1.1 or later
- Per-port MTU range: 1500–9198 bytes (varies by platform)
- Catalyst 9000v virtual switch: max MTU may be lower than physical hardware
- MTU can only be set on routed (Layer 3) ports via the native YANG `mtu` leaf

### NACM Privilege Level Interaction

```
Privilege 15  →  PRIV15 group  →  hardcoded permit-all  →  NACM rules IGNORED
Privilege 1   →  PRIV01 group  →  no default rules      →  custom NACM rules APPLY
```

Always use a non-privilege-15 user for monitoring accounts where NACM write-deny is needed.

---

## Final IOS-XE Running Config (Relevant Sections)

```
aaa new-model
aaa authentication login default local
aaa authorization exec default local

username manzil privilege 15 password 0 Asdf@1234
username joy privilege 15 password 0 Asdf@1234
username hasan password 0 Asdf@1234
username gnmi_client secret 9 <hashed>

crypto pki trustpoint GNMI-TP
 enrollment pkcs12
 revocation-check none
 rsakeypair GNMI-TP

gnxi
gnxi secure-client-auth
gnxi secure-password-auth
gnxi secure-trustpoint GNMI-TP
gnxi secure-server
gnxi server
netconf-yang
```
