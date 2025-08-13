# SNMPv3 User Creation and Security Levels

## **1️⃣ SNMPv3 Security Levels (How SNMPv3 Works — simple breakdown)**

SNMPv3 provides three main **security levels**, combining authentication (`auth`) and encryption (`priv`) options:

| Level            | Authentication                   | Encryption             | Example Use Case                                |
| ---------------- | -------------------------------- | ---------------------- | ----------------------------------------------- |
| **noAuthNoPriv** | ❌ No authentication              | ❌ No encryption        | Quick internal tests, not secure                |
| **authNoPriv**   | ✅ Auth (password-based, MD5/SHA) | ❌ No encryption        | Ensures sender is trusted, but data is readable |
| **authPriv**     | ✅ Auth (MD5/SHA)                 | ✅ Encryption (DES/AES) | Secure communications (auth + encryption)       |

* **Authentication algorithms**

  * `MD5` → older, less secure hashing algorithm.
  * `SHA` → stronger than MD5, preferred.

* **Encryption algorithms**

  * `DES` → older, weaker encryption.
  * `AES` → strong encryption, recommended.

**Best practice:** Use **authPriv** with **SHA** + **AES**.

---

## **2️⃣ Your Command Breakdown**

```bash
net-snmp-create-v3-user -ro -A "xyxASDFasldfk" -a SHA -X snmpnetkey -x AES monuser 
perl -i -pe 's/(rouser monuser)/$1 priv/' /etc/snmp/snmpd.conf
```

**Step-by-step:**

1. **Create SNMPv3 user**

   ```bash
   net-snmp-create-v3-user -ro -A "xyxASDFasldfk" -a SHA -X snmpnetkey -x AES monuser
   ```

   * `-ro` → Read-Only user (cannot modify SNMP data)
   * `-A "xyxASDFasldfk"` → Authentication password
   * `-a SHA` → Authentication algorithm = SHA
   * `-X snmpnetkey` → Encryption password
   * `-x AES` → Encryption algorithm = AES
   * `monuser` → Username

   This creates a **read-only** SNMPv3 user with:

   * **Auth:** SHA + password `"xyxASDFasldfk"`
   * **Encryption:** AES + password `"snmpnetkey"`

2. **Edit SNMP config to enforce `priv` level**

   ```bash
   perl -i -pe 's/(rouser monuser)/$1 priv/' /etc/snmp/snmpd.conf
   ```

   * Finds the line with `rouser monuser`
   * Adds `priv` at the end → `rouser monuser priv`
   * `priv` here means **authPriv** (authentication + encryption required).

---
