# Lab 12 — Blind SQL Injection with Time Delays and Information Retrieval

**Difficulty:** Practitioner  
**Category:** Blind SQL Injection — Time-Based  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

No errors, no visible differences in responses — the app is completely silent. The only feedback channel left is time. If the query delays, the condition was true.

---

## The vulnerability

The TrackingId cookie is injectable into a PostgreSQL query that runs synchronously. If we inject a sleep function, the HTTP response delays by exactly that amount — giving us a true/false signal via timing.

---

## What I did

**Step 1 — confirm time-based injection:**

```
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(2)+ELSE+pg_sleep(0)+END--
```

Response took ~2 seconds — injection confirmed. (`%3B` is URL-encoded `;`)

**Step 2 — confirm admin exists:**

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(2)+ELSE+pg_sleep(0)+END+FROM+users--
```

2 second delay — administrator user exists.

**Step 3 — find password length:**

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(2)+ELSE+pg_sleep(0)+END+FROM+users--
```

Incremented the number until response was instant — password is 20 characters.

**Step 4 — extract password with Intruder:**

Set up Cluster Bomb attack with two payload positions — character position (1-20) and character value (a-z, 0-9). Set resource pool to 1 concurrent request. Sorted results by Response Received — rows showing ~2000ms were the hits.

**Problems I ran into:**

- Lab kept timing out because Community Burp throttles Intruder to ~1 req/sec. With `pg_sleep(10)` and 720 requests that's over 2 hours
- Fixed by switching to `pg_sleep(2)` — much faster
- Was intercepting `GET /my-account` instead of `GET /` — `/my-account` redirects to login when session expires, giving 302s for everything and making all results look identical. Fixed by intercepting the homepage request

---

## Key takeaway

Time-based blind SQLi is the most reliable technique when everything else fails because timing can't be suppressed by the app. The main challenge in practice is speed — keep sleep values small (2-3 seconds) and use Intruder carefully. Always intercept the right endpoint and check status codes in results — if everything is 302, the lab session has expired.
