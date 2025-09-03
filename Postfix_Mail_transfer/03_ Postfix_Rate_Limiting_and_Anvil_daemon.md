# **Postfix Rate Limiting and Deferred Mail Handling**

Postfix provides robust mechanisms to **limit mail submission rates**, **relay mail for trusted networks**, and **handle deferred mail** with controlled retries. These are managed by the **Anvil daemon** and Postfix queue system.

---

## **1. Rate Limiting with Anvil**

Postfix uses the **Anvil daemon** to track **SMTP client connections and message submission rates**. This prevents clients from overloading the server with too many messages in a short period.

### **1.1 Basic Configuration**

```bash
smtpd_client_message_rate_limit = 10
anvil_rate_time_unit = 60s
```

* `smtpd_client_message_rate_limit` → Maximum messages allowed per client within the specified time unit.
* `anvil_rate_time_unit` → Time window in seconds (e.g., 60s).

---

### **1.2 Default Behavior**

By default, Postfix exempts **trusted networks** (usually defined in `$mynetworks`) from rate limits:

```bash
smtpd_client_event_limit_exceptions = $mynetworks
```

* `$mynetworks` typically includes the loopback and LAN addresses.
* This means **only untrusted clients are limited** unless explicitly changed.

---

### **1.3 Enforcing Rate Limits for All Clients**

To apply limits to **all clients**, including trusted networks:

```bash
smtpd_client_event_limit_exceptions =
```

* An empty value removes all exemptions.
* Final configuration to limit all clients:

```bash
smtpd_client_message_rate_limit = 10
anvil_rate_time_unit = 60s
smtpd_client_event_limit_exceptions =
```

---

### **1.4 How Anvil Tracks Clients**

* **In-memory counters**: Tracks client IPs, number of messages, and connection events.
* **Temporary blocking**: If a client exceeds the limit, they receive a **4xx temporary SMTP error**:

`mailq` example output:

```bash
12E7420DB3E7     326 Thu Aug 21 13:02:11  root@lab.local
(host 192.168.9.25[192.168.9.25] said: 450 4.7.1 Error: too much mail from 192.168.9.28 (in reply to MAIL FROM command))
                                         root@mta-01.lab.local
```

* **Counter reset**: After `anvil_rate_time_unit` expires, the client can send mail again.
* **Logging**: Blocked clients and message rates are logged in `/var/log/maillog`:

```bash
grep "postfix/anvil" /var/log/maillog | grep "max connection rate"
```

**Example output:**

```bash
Aug 21 12:11:53 mta-01 postfix/anvil[3056]: statistics: max connection rate 27/60s for (smtp:192.168.9.28) at Aug 21 12:08:33
Aug 21 12:16:58 mta-01 postfix/anvil[34685]: statistics: max connection rate 17/60s for (smtp:192.168.9.28) at Aug 21 12:13:38
```

---

## **2. Postfix Relay Servers**

* Postfix **automatically relays mail from trusted networks** without needing explicit relay configuration.
* Once the relay server (e.g., `mail01`) accepts the mail, **delivery responsibility moves to the relay**, not the client.

---

## **3. Deferred Mail Handling**

When mail **cannot be delivered immediately**, Postfix places it in the **deferred queue**.

### **3.1 Reasons for Deferred Mail**

* Destination host unreachable (DNS, network issues)
* Recipient mailbox temporarily unavailable (full mailbox, greylisting)
* Temporary 4xx SMTP errors

---

### **3.2 Retry Configuration**

Postfix retries deferred mail according to:

```bash
minimal_backoff_time = 300s       # first retry interval (5 min)
maximal_backoff_time = 4000s      # maximum retry interval (~1h 6 min)
maximal_queue_lifetime = 5d       # maximum time to keep retrying
```

* **First retry:** after `minimal_backoff_time`.
* **Subsequent retries:** interval **increases gradually (smoothed exponential)** toward `maximal_backoff_time`.
* **After `maximal_queue_lifetime`:** mail is bounced back to the sender if still undeliverable.

---

### **3.3 Smoothed Exponential Example**

| Retry Attempt | Interval                    |
| ------------- | --------------------------- |
| 1             | 5 min (300 s)               |
| 2             | 8 min (480 s)               |
| 3             | 12 min (720 s)              |
| 4             | 20 min (1200 s)             |
| 5             | 40 min (2400 s)             |
| 6+            | 4000 s (\~1h 6 min, capped) |

> **Note:** This is gradual growth, not strict doubling. Early retries happen faster; later retries slow down and cap at `maximal_backoff_time`.

---

### **3.4 Checking Retry Parameters**

```bash
postconf minimal_backoff_time
postconf maximal_backoff_time
postconf maximal_queue_lifetime
```

* Displays **active retry settings**, either default or user-configured.

---

## **4. Anvil vs Deferred Queue**

| Feature            | Purpose                                         | Storage                              |
| ------------------ | ----------------------------------------------- | ------------------------------------ |
| **Anvil**          | Tracks message rates, enforces temporary blocks | **Memory (volatile)**                |
| **Deferred Queue** | Holds undeliverable mail, retries delivery      | Disk (`/var/spool/postfix/deferred`) |

* **Anvil counters** are **lost on Postfix restart**.
* **Deferred mail** persists until delivered or expired.

---
