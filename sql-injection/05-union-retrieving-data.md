# Lab 05 — SQL Injection UNION Attack, Retrieving Data from Other Tables

**Difficulty:** Practitioner  
**Category:** SQL Injection — UNION Attacks  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

Now that we know the column count and which columns hold strings, we can actually pull data from a completely different table — in this case the users table.

---

## The vulnerability

The UNION operator lets you append a second query that reads from any table in the database. Since the app displays the results of the original query, it'll display our injected results too.

---

## What I did

The original query returns 2 columns. I used:

```
' UNION SELECT username,password FROM users--
```

This appends a query that reads from the users table and returns it alongside the normal product data. The app displayed usernames and passwords directly on the page.

Logged in as administrator with the retrieved password.

---

## Key takeaway

This is the payoff of the previous two labs — once you know column count and types, extracting data from other tables is just one query away. This is why SQL injection is so dangerous: one vulnerable parameter can expose your entire database.
