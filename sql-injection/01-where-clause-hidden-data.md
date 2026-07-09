# Lab 01 — SQL Injection Vulnerability in WHERE Clause Allowing Retrieval of Hidden Data

**Difficulty:** Apprentice  
**Category:** SQL Injection  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

The application has a product category filter that uses a SQL query to display items. The query only returns products marked as released. The goal is to retrieve unreleased products by manipulating the WHERE clause.

---

## The vulnerability

The category parameter is dropped directly into a SQL query with no sanitisation:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

By injecting into the category value, you can modify the logic of the query entirely.

---

## What I did

The category filter was passed in the URL like this:

```
/filter?category=Gifts
```

I modified it to:

```
/filter?category=Gifts'+OR+1=1--
```

This changes the query to:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

Since `1=1` is always true, the WHERE clause returns everything. The `--` comments out the `AND released = 1` check, so unreleased products show up too.

---

## Key takeaway

The `--` comment is essential — without it the query would break due to the leftover syntax. The injection point was the URL parameter, which is one of the most common places to start testing.
