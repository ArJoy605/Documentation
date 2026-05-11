## Removing Old Declaration & Reconfiguring with `svc_nms` using Postman

---

### Step 1: Reset / Remove the Old Declaration

**Method:** `POST`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/telemetry/declare
```

**Auth tab:**
- Type: `Basic Auth`
- Username: `svc_nms`
- Password: `<password>`

**Body tab:**
- Select `raw` → `JSON`
- Paste:
```json
{
  "class": "Telemetry"
}
```

Click **Send** — you should get a `200 OK` with the response confirming the empty declaration. This wipes the old poller.

---

### Step 2: Post the New Declaration

**Method:** `POST`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/telemetry/declare
```

**Auth tab:**
- Type: `Basic Auth`
- Username: `svc_nms`
- Password: `<password>`

**Body tab:**
- Select `raw` → `JSON`
- Paste:
```json
{
  "class": "Telemetry",
  "My_Poller": {
    "class": "Telemetry_System_Poller",
    "interval": 0
  },
  "My_System": {
    "class": "Telemetry_System",
    "enable": "true",
    "systemPoller": [
      "My_Poller"
    ]
  },
  "metrics": {
    "class": "Telemetry_Pull_Consumer",
    "type": "Prometheus",
    "systemPoller": "My_Poller"
  }
}
```

Click **Send** — expect `200 OK` with the full declaration echoed back.

---

### Step 3: Verify the Declaration was Applied

**Method:** `GET`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/telemetry/declare
```

**Auth tab:**
- Type: `Basic Auth`
- Username: `svc_nms`
- Password: `<password>`

No Body needed. Click **Send** — you should see your `My_Poller`, `My_System`, and `metrics` objects returned.

---

### Step 4: Verify Metrics are Being Exposed

**Method:** `GET`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/telemetry/pullconsumer/metrics
```

**Auth tab:**
- Type: `Basic Auth`
- Username: `svc_nms`
- Password: `<password>`

Click **Send** — you should see a large Prometheus-formatted text response with BIG-IP metrics.

---

### Step 5: Update Prometheus Config

Once Postman confirms metrics are working, go to your Prometheus host and update `/etc/prometheus/prometheus.yml` with the new credentials:

```yaml
basic_auth:
  username: 'svc_nms'
  password: '<svc_nms_password>'
```

Then restart:
```bash
sudo systemctl restart prometheus.service
```

---

### Quick Reference Summary

| Step | Method | Endpoint |
|---|---|---|
| Reset old config | POST | `/mgmt/shared/telemetry/declare` — body: `{"class":"Telemetry"}` |
| Apply new config | POST | `/mgmt/shared/telemetry/declare` — body: full declaration |
| Verify declaration | GET | `/mgmt/shared/telemetry/declare` |
| Verify metrics | GET | `/mgmt/shared/telemetry/pullconsumer/metrics` |

All requests use **Basic Auth** with `svc_nms`. Make sure `svc_nms` has the **Administrator** role on BIG-IP, otherwise you'll get a `401` or `403`.
