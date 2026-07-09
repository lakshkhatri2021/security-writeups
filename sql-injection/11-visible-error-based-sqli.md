# Lab 11 — Visible Error-Based SQL Injection

**Difficulty:** Practitioner  
**Category:** Error-Based SQL Injection  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

The database is misconfigured and shows verbose error messages. We can exploit this to leak data directly through the error output — much faster than blind techniques.

---

## The vulnerability

The TrackingId cookie is injectable into a PostgreSQL query. When a type error occurs, PostgreSQL includes the actual value in the error message — so we force a type error on the data we want.

---

## What I did

**Initial issue:** my TrackingId was too long and the payload got truncated. Fixed by shortening the ID to just `x'`:

```
TrackingId=x' AND CAST((SELECT password FROM users WHERE username='administrator') AS int)--
```

This tries to cast the password string as an integer. PostgreSQL can't do that, so it throws:

```
ERROR: invalid input syntax for type integer: "gh09nmm6c1xckkpec7qn"
```

The password is right there in the error message.

**Issue I hit:** initially used `WHERE username='administrator'` but the payload got cut off due to character limits. Switched to `LIMIT 1` instead:

```
TrackingId=x' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

Also had to use `1=CAST(...)` rather than just `CAST(...)` because AND requires a boolean expression on both sides.

Logged in as administrator with the leaked password.

---

## Mistakes made

- Kept the full original TrackingId making the payload too long — shortened to `x` fixed it
- Used just `CAST(...)` without the `1=` comparison which broke the AND logic
- `--` comment wasn't working until I checked the exact syntax required

---

## Key takeaway

Verbose errors are a massive misconfiguration. Forcing a type mismatch on the data you want is a clean one-request technique that bypasses the need for 720 Intruder requests. Always check error messages carefully — they often contain more than you expect.
