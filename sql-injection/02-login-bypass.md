# Lab 02 — SQL Injection Vulnerability Allowing Login Bypass

**Difficulty:** Apprentice  
**Category:** SQL Injection  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

The login form passes the username directly into a SQL query. The goal is to log in as the administrator without knowing the password.

---

## The vulnerability

The login query looks something like this:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'password'
```

If we can manipulate the username field, we can comment out the password check entirely.

---

## What I did

In the username field I entered:

```
administrator'--
```

Left the password field blank and submitted.

This turns the query into:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

The `--` comments out everything after the username check — the password condition never runs. The DB returns the administrator user and the app logs me straight in.

---

## Key takeaway

This is one of the most classic SQLi attacks. The `'--` pattern breaks out of the string and comments out the rest of the query. Any login form that doesn't use parameterised queries is vulnerable to this.
