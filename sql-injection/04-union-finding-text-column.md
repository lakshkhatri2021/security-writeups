# Lab 04 — SQL Injection UNION Attack, Finding a Column Containing Text

**Difficulty:** Practitioner  
**Category:** SQL Injection — UNION Attacks  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

Once you know the column count, you need to figure out which columns can hold string data — because that's where you'll extract text like passwords.

---

## The vulnerability

Not every column in a query is a string type. If you try to extract a string value into an integer column, the DB throws a type error. You need to probe each column individually.

---

## What I did

The lab gives you a specific string to find (something like `'abc123'`). I replaced each NULL one at a time with that string:

```
' UNION SELECT 'abc123',NULL--    ← error, column 1 isn't string
' UNION SELECT NULL,'abc123'--    ← works, column 2 is string
```

When the app reflected the string back in the response, that column is usable for text extraction.

---

## Key takeaway

You can only pull string data through string-compatible columns. This step tells you exactly which column to use when you move on to extracting usernames and passwords in the next labs.
