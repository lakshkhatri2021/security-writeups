# Attack Surface Recon — scanme.nmap.org

## Overview

Ran my recon tool (https://github.com/lakshkhatri2021/recon-tool) against
`scanme.nmap.org`, the official public test host maintained by the Nmap
project for people to practice scanning on. The tool chains subfinder,
httpx, naabu, tlsx, dnstwist, checkdmarc, and nuclei to map external exposure
for a domain.

## Results

### 1. Subdomain enumeration (subfinder)
0 subdomains found. Expected — `scanme.nmap.org` is a single standalone test
host, not a company with sprawling infrastructure, so passive sources
(Certificate Transparency logs, DNS aggregators etc) have nothing indexed
under it.

### 2. Live host probing (httpx)
`http://scanme.nmap.org` confirmed live on HTTP.

### 3. Port scanning (naabu)
Two ports open:
- **22 (SSH)**
- **80 (HTTP)**

Port 443 (HTTPS) is not open — confirmed in the next step.

### 4. TLS inspection (tlsx)
No TLS data returned. Since port 443 isn't open, there's no certificate to
inspect. This means any traffic to this host over HTTP is unencrypted.

### 5. Typosquatting detection (dnstwist)
Generated hundreds of lookalike domain permutations (homoglyphs, bitsquats,
insertions, hyphenations etc). A handful are actually registered and resolve,
e.g.:
- `scanme.nmapd.org` → parked via cashparking.com
- `scanme.nmaps.org` → parked via parkingcrew.net
- `scanme.nmal.org`, `scanme.nmsp.org`, several `afternic.com`-parked domains

These are domain-parking services, not active phishing infrastructure, but
this is exactly the category of finding that matters for a real company —
if any of these were actively serving content mimicking the original site,
it would be a strong phishing indicator.

### 6. Email security (checkdmarc)
- **SPF**: not configured — no SPF record exists.
- **DMARC**: not configured.
- **DKIM**: not checked directly by this tool, but absence of DMARC means
  no enforcement policy exists either way.
- **MTA-STS / BIMI**: also not configured.

This means email claiming to be from `@scanme.nmap.org` (or `@nmap.org`)
has no authentication framework backing it — in a real org, this is a
direct enabler for email spoofing / business email compromise.

### 7. Vulnerability scanning (nuclei)
Timed out after 5 minutes against the single live host (nuclei runs 10,000+
templates, so this is expected on a full run — for production use this stage
would need a longer timeout or a reduced template set).

## Key takeaways

- This host has a genuinely minimal external footprint — one live service
  over plaintext HTTP, SSH open, no TLS, no email auth records.
- For a real target, the most actionable findings from this run alone would
  be: (1) no SPF/DMARC — fix immediately, it's a five-minute DNS change with
  major impact on phishing risk, and (2) HTTP-only with no TLS — anything
  transmitted is plaintext.
- The dnstwist results show why typosquat monitoring matters even for small
  targets — parked lookalike domains exist for almost any domain name, and
  monitoring which ones become "live" over time is a useful early-warning
  signal for phishing campaigns.

## Tool notes / improvements made

- Rewrote the original script to remove a hardcoded local path, add proper
  error handling per stage, fix a shell-injection issue in how subdomain
  lists were piped into httpx/nuclei, and make naabu scan discovered
  subdomains in addition to the root domain (previously it only scanned the
  root domain).
- Full report: `scanme.nmap.org_report.txt` in the recon-tool repo.
