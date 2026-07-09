# SQL Injection — PortSwigger Web Security Academy

Writeups for the full SQL injection learning path on PortSwigger. Completed as part of building toward AppSec knowledge alongside Security+ prep.

**Tools used:** Burp Suite Community Edition, browser proxy

---

## Labs

| # | Lab | Difficulty | Technique |
|---|-----|-----------|-----------|
| 01 | [WHERE clause hidden data](./01-where-clause-hidden-data.md) | Apprentice | Basic SQLi |
| 02 | [Login bypass](./02-login-bypass.md) | Apprentice | Basic SQLi |
| 03 | [UNION — number of columns](./03-union-number-of-columns.md) | Practitioner | UNION attack |
| 04 | [UNION — finding text column](./04-union-finding-text-column.md) | Practitioner | UNION attack |
| 05 | [UNION — retrieving data](./05-union-retrieving-data.md) | Practitioner | UNION attack |
| 06 | [UNION — multiple values in single column](./06-union-multiple-values-single-column.md) | Practitioner | UNION attack |
| 07 | [Querying database version](./07-querying-database-version.md) | Practitioner | DB enumeration |
| 08 | [Listing database contents](./08-listing-database-contents.md) | Practitioner | DB enumeration |
| 09 | [Blind SQLi — conditional responses](./09-blind-sqli-conditional-responses.md) | Practitioner | Blind SQLi |
| 10 | [Blind SQLi — conditional errors](./10-blind-sqli-conditional-errors.md) | Practitioner | Blind SQLi |
| 11 | [Visible error-based SQLi](./11-visible-error-based-sqli.md) | Practitioner | Error-based |
| 12 | [Time-based blind SQLi](./12-time-delay-information-retrieval.md) | Practitioner | Time-based |
| 13-14 | [OAST techniques](./13-14-oast-techniques.md) | Practitioner | OAST (Pro only) |
| 15 | [Filter bypass via XML encoding](./15-filter-bypass-xml-encoding.md) | Practitioner | WAF bypass |

---

## What I learned

- SQLi exists anywhere user input touches a SQL query — URL params, cookies, JSON, XML
- UNION attacks require matching column count and compatible data types
- Blind SQLi relies on indirect signals: response differences, errors, or timing
- Error-based is faster than time-based when verbose errors are enabled
- Time-based is the last resort when everything else is silent
- OAST is the most powerful technique but requires Burp Pro
- WAFs can be bypassed by encoding payloads in formats the parser handles before SQL sees them
- Prevention: parameterised queries / prepared statements — never string concatenation
