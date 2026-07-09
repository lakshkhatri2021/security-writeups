# Lab 15 — SQL Injection with Filter Bypass via XML Encoding

**Difficulty:** Practitioner  
**Category:** SQL Injection in Different Contexts  
**Platform:** PortSwigger Web Security Academy

---

## What the lab is

SQLi isn't limited to URL parameters. This lab has a stock check feature that sends data as XML. A WAF blocks standard SQL keywords — the goal is to bypass it using XML encoding.

---

## The vulnerability

The stock check sends a POST request with an XML body:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
  <productId>1</productId>
  <storeId>1</storeId>
</stockCheck>
```

The `storeId` value is injectable. However a WAF sits in front and blocks requests containing SQL keywords like `UNION` and `SELECT`.

---

## What I did

**Step 1 — find the injection point:**

Intercepted the stock check request in Burp (turned on intercept, clicked "Check stock", sent to Repeater).

Tested if input was evaluated by trying `1+1` in storeId — app returned stock for store 2, confirming the input goes into the query.

**Step 2 — WAF blocks normal payload:**

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

Got `403 - Attack detected`. WAF is filtering SQL keywords.

**Step 3 — bypass with XML encoding:**

Installed the **Hackvertor** extension from the BApp Store. Used it to wrap the payload in hex encoding tags:

```xml
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>
```

Hackvertor converts everything inside the tags to XML hex entities before sending. The WAF sees encoded characters, not SQL keywords. The XML parser on the server decodes them back before they hit the database.

**Step 4 — read the response:**

Got back `administrator~[password]`. Logged in as administrator.

---

## Why only one column

When testing column count, returning two columns gave 0 units (an error). So the query only returns one column — used `||'~'||` concatenation to squeeze both username and password into it.

---

## Key takeaway

SQLi exists anywhere user input touches a SQL query — URL params, cookies, JSON bodies, XML bodies. WAFs that filter raw keywords can be bypassed by encoding the payload in a format the input parser handles. XML encoding works here because the XML parser decodes entities server-side before the SQL interpreter ever sees them — the WAF only sees the encoded version.
