# Security Writeups

Hands-on writeups from PortSwigger Web Security Academy, TryHackMe and my own recon tooling. I'm a third-year B.Tech CSE (Cybersecurity) student at VIT Vellore. Each writeup covers the technique, what I did, what went wrong along the way, and where relevant, how the issue would be fixed. Everything here was done on intentionally vulnerable labs, on systems I own, or on targets that explicitly allow scanning.

## PortSwigger: SQL injection

Fourteen writeups covering all 15 labs in the SQL injection series, from retrieving hidden data through to blind SQLi, out-of-band techniques and filter bypass. The folder has its own [README](sql-injection/README.md).

| # | Writeup | Technique |
|---|---|---|
| 01 | [WHERE clause hidden data](sql-injection/01-where-clause-hidden-data.md) | Retrieving hidden data |
| 02 | [Login bypass](sql-injection/02-login-bypass.md) | Subverting application logic |
| 03 | [UNION: number of columns](sql-injection/03-union-number-of-columns.md) | UNION attack |
| 04 | [UNION: finding a text column](sql-injection/04-union-finding-text-column.md) | UNION attack |
| 05 | [UNION: retrieving data](sql-injection/05-union-retrieving-data.md) | UNION attack |
| 06 | [UNION: multiple values in one column](sql-injection/06-union-multiple-values-single-column.md) | UNION attack |
| 07 | [Querying the database version](sql-injection/07-querying-database-version.md) | Database fingerprinting |
| 08 | [Listing database contents](sql-injection/08-listing-database-contents.md) | Schema enumeration |
| 09 | [Blind SQLi: conditional responses](sql-injection/09-blind-sqli-conditional-responses.md) | Blind SQLi |
| 10 | [Blind SQLi: conditional errors](sql-injection/10-blind-sqli-conditional-errors.md) | Blind SQLi |
| 11 | [Visible error-based SQLi](sql-injection/11-visible-error-based-sqli.md) | Error-based SQLi |
| 12 | [Time delay information retrieval](sql-injection/12-time-delay-information-retrieval.md) | Time-based blind SQLi |
| 13-14 | [OAST techniques](sql-injection/13-14-oast-techniques.md) | Out-of-band SQLi |
| 15 | [Filter bypass with XML encoding](sql-injection/15-filter-bypass-xml-encoding.md) | WAF and filter bypass |

## Recon tooling

| Writeup | What it covers |
|---|---|
| [Attack surface recon: scanme.nmap.org](internship/recon-tool.md) | I rewrote an existing recon script into [recon-tool](https://github.com/lakshkhatri2021/recon-tool), chaining subfinder, httpx, naabu, tlsx, dnstwist, checkdmarc, nuclei and ffuf. Part 1 is the results and recommended fixes (Terrapin CVE, weak SSH crypto, missing email authentication and security headers). Part 2 is running the pipeline from a free-tier AWS VPS and fixing disk and memory problems along the way. |

## TryHackMe

| Writeup | What it covers |
|---|---|
| [Pre Security path](THM/pre-security.md) | Summary of the fundamentals: networking, ports, how the web works, Linux and Windows, cloud, cryptography, the CIA triad |
