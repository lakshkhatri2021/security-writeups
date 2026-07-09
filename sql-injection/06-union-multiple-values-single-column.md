# Lab 06 — SQL Injection UNION Attack, Retrieving Multiple Values in a Single Column

**Difficulty:** Practitioner  
**Category:** SQL Injection — UNION Attacks  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

Sometimes the query only returns one usable string column. This lab covers how to get multiple pieces of data (username and password) through a single column.

---

## The vulnerability

Same injectable category filter, but this time only one column accepts string data. You can't do `UNION SELECT username,password` because one of those would go into a non-string column and error.

---

## What I did

Used string concatenation to combine username and password into one value with a separator:

```
' UNION SELECT NULL,username||'~'||password FROM users--
```

The `||` operator concatenates strings in PostgreSQL. The `~` is just a separator so I can split the result.

The app returned something like:

```
administrator~s3cur3p4ssw0rd
```

Split on `~`, logged in as administrator.

---

## Key takeaway

When you're limited to one column, concatenation is how you get multiple values out. The separator character just needs to be something that won't appear in the actual data — `~` or `:` works fine. Different databases use different concatenation syntax: `||` in PostgreSQL/Oracle, `+` in MSSQL, `CONCAT()` in MySQL.
