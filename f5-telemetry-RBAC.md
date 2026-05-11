### Step 1: Change `svc_nms` Role to `no-access`

**Method:** `PUT`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/tm/auth/user/svc_nms
```
**Auth:** Basic Auth → `admin` / `<admin-password>`

**Body → raw → JSON:**
```json
{
  "partitionAccess": [
    {
      "name": "all-partitions",
      "role": "no-access"
    }
  ],
  "shell": "none"
}
```

Click **Send** → expect `200 OK`.

---

### Step 2: Create the Resource Group

**Method:** `POST`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/authz/resource-groups
```
**Auth:** Basic Auth → `admin` / `<admin-password>`

**Body → raw → JSON:**
```json
{
  "name": "telemetry_resource_group",
  "resources": [
    {
      "resourceMask": "/mgmt/shared/telemetry/pullconsumer/metrics",
      "restMethod": "GET"
    }
  ]
}
```

Click **Send** → **copy the `selfLink` from the response.**

---

### Step 3: Create the Role

**Method:** `POST`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/authz/roles
```
**Auth:** Basic Auth → `admin` / `<admin-password>`

**Body → raw → JSON:**
```json
{
  "name": "telemetry_readonly",
  "userReferences": [
    {
      "link": "https://localhost/mgmt/shared/authz/users/svc_nms"
    }
  ],
  "resourceGroupReferences": [
    {
      "link": "<selfLink copied from Step 2>"
    }
  ]
}
```

Click **Send** → expect `200 OK`. ✅

---

### Step 4: Test Access

**Method:** `GET`
**URL:**
```
https://<big-ip-mgmt-ip>/mgmt/shared/telemetry/pullconsumer/metrics
```
**Auth:** Basic Auth → `svc_nms` / `<password>`

→ Should return `200 OK` with metrics. ✅

Try any other endpoint like `/mgmt/tm/ltm` with `svc_nms` → should return `401` or `403`. ✅
