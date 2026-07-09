# Lab 07 — SQL Injection Attack, Querying the Database Type and Version on MySQL and Microsoft

**Difficulty:** Practitioner  
**Category:** SQL Injection — Examining the Database  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

Different databases have different syntax. Before going deep on an attack, it helps to know exactly which DB you're dealing with. This lab is about fingerprinting the database.

---

## The vulnerability

The category filter is injectable. We can use it to query built-in system variables that reveal the DB type and version.

---

## What I did

First confirmed the column count was 2 using ORDER BY, then used the version query specific to MySQL/MSSQL:

```
' UNION SELECT @@version,NULL--
```

On MySQL/MSSQL, `@@version` is a global variable that returns the database version string. The response showed the full version info confirming the DB type.

Note: on PostgreSQL it's `SELECT version()`, on Oracle it's `SELECT banner FROM v$version`.

---

## Key takeaway

Knowing the DB type matters because syntax varies across databases — comment styles, string concatenation, sleep functions, version queries all differ. Fingerprinting early saves you from using the wrong syntax and getting errors that make you think the injection isn't working.
