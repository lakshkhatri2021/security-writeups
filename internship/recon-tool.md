# Attack Surface Recon: scanme.nmap.org

## Overview

I took an existing Python recon script and rewrote it into [recon-tool](https://github.com/lakshkhatri2021/recon-tool). It chains subfinder, httpx, naabu, tlsx, dnstwist, checkdmarc, nuclei and ffuf to map the external exposure of a domain.

This writeup has two parts. Part 1 covers running it against `scanme.nmap.org`, the public test host maintained by the Nmap project. Part 2 covers running the same pipeline from a free-tier AWS VPS and the resource problems I hit.

## Scope and authorisation

scanme.nmap.org is run by the Nmap project, and its page authorises scanning with Nmap or other port scanners and asks users not to hammer it. The port scanning, DNS and email stages here fall inside that. The nuclei and ffuf stages go beyond plain port scanning, so I treat this as a one-off. Web-layer testing like this should be done against systems I own, such as an intentionally vulnerable app on my own VPS.

## Changes I made to the original script

- Removed a hardcoded local path.
- Added proper error handling per stage.
- Fixed a shell-injection issue in how subdomain lists were piped into httpx and nuclei.
- Made naabu scan discovered subdomains as well as the root domain (previously it only scanned the root).
- Added ffuf as an 8th stage for directory and file brute-forcing.

The full report is `scanme.nmap.org_report.txt` in the recon-tool repo.

# Part 1: Results

## 1. Subdomain enumeration (subfinder)

0 subdomains found. This was expected: `scanme.nmap.org` is a single standalone test host, not a company with sprawling infrastructure, so passive sources (Certificate Transparency logs, DNS aggregators and so on) have nothing indexed under it.

## 2. Live host probing (httpx)

`http://scanme.nmap.org` confirmed live on HTTP.

## 3. Port scanning (naabu)

Two ports open:
- **22 (SSH)**
- **80 (HTTP)**

Port 443 (HTTPS) is not open, which the next step confirmed.

## 4. TLS inspection (tlsx)

No TLS data returned. Since port 443 isn't open, there's no certificate to inspect, which means any traffic to this host over HTTP is unencrypted.

## 5. Typosquatting detection (dnstwist)

Generated hundreds of lookalike domain permutations (homoglyphs, bitsquats, insertions, hyphenations and so on). A handful are registered and resolve, for example:
- `scanme.nmapd.org`, parked via cashparking.com
- `scanme.nmaps.org`, parked via parkingcrew.net
- `scanme.nmal.org`, `scanme.nmsp.org` and several `afternic.com`-parked domains

These are domain-parking services, not active phishing infrastructure. But this is exactly the category of finding that matters for a real company: if any of these were serving content that mimicked the original site, it would be a strong phishing indicator.

## 6. Email security (checkdmarc)

- **SPF:** not configured, no SPF record exists.
- **DMARC:** not configured.
- **DKIM:** not checked directly by this tool, but with no DMARC there is no enforcement policy either way.
- **MTA-STS and BIMI:** also not configured.

Email claiming to be from `@scanme.nmap.org` (or `@nmap.org`) has no authentication backing it. In a real organisation this directly enables email spoofing and business email compromise.

## 7. Vulnerability scanning (nuclei)

The first run timed out after 5 minutes against the single live host, which is expected because nuclei runs 10,000+ templates. A later full run completed and found:

- **CVE-2023-48795 (Terrapin attack):** medium severity, affects the SSH protocol implementation on this host because of an outdated OpenSSH version.
- **Weak SSH cryptography:** weak MAC algorithms, CBC-mode ciphers and a weak key exchange (Diffie-Hellman, Logjam-style) are all supported by the server's SSH config.
- **Outdated software:** `Apache/2.4.7 (Ubuntu)` and `OpenSSH_6.6.1p1`. Both are old, which explains the CVE and the weak default algorithm lists.
- **Missing security headers:** CSP, X-Frame-Options, Strict-Transport-Security and others are all absent, which leaves the site more exposed to clickjacking and XSS-style attacks.

## 8. Directory and file brute-forcing (ffuf)

ffuf tries common file and directory names against each live host (`/admin`, `/.env`, `/.git`, `/backup.zip` and so on). On this target it flagged `images`, `.svn`, `.htaccess` and `.htpasswd` as existing. Checking each one manually returned **403 Forbidden**, meaning the files exist but Apache correctly blocks direct access.

The distinction matters: ffuf reports that something exists (anything other than a 404), not that it's readable. A 403 means "found but protected", not "exposed".

## Key takeaways

- A quick first pass can understate exposure. The early run made the host look nearly empty (one live service over plaintext HTTP, SSH open, no TLS, no email authentication). The full run with nuclei and ffuf found a documented CVE, weak SSH crypto, outdated software and missing security headers.
- The most serious finding was Terrapin (CVE-2023-48795) on an outdated OpenSSH, alongside weak MACs, CBC ciphers and a weak key exchange.
- No SPF or DMARC is the most actionable finding for a real target. It's a five-minute DNS change with a major effect on phishing and spoofing risk.
- HTTP-only with no TLS means anything transmitted is plaintext.
- Typosquat monitoring matters even for small targets. Parked lookalike domains exist for almost any domain name, and watching which ones go live over time is a useful early warning for phishing campaigns.

## Recommended fixes

1. **Upgrade OpenSSH** to a current release (9.6 or later fixes Terrapin) and update Apache from 2.4.7.
2. **Harden SSH crypto:** disable CBC ciphers, weak MAC algorithms and the weak Diffie-Hellman key exchange groups.
3. **Add security headers:** Content-Security-Policy, X-Frame-Options and Strict-Transport-Security.
4. **Enable HTTPS** on port 443 and redirect HTTP to it.
5. **Publish SPF and DMARC records** so spoofed email from the domain is rejected.
6. **Monitor typosquat domains** and act if any start serving content that mimics the real site.

---

# Part 2: Running the pipeline from a foreign VPS

## Context

As part of OPSEC fundamentals for the internship (IP masking, system masking, standalone foreign infrastructure), the goal was to take the recon script and run it from a server whose IP isn't linked to my home network. The AWS account is still tied to my billing details, so this hides my network location, not my identity.

The plan was a free-tier AWS EC2 instance (Ubuntu, t2/t3.micro, London region) with the full toolchain installed and the script run natively on the box, so every outbound request genuinely originates from that server's IP.

## Problem 1: Disk space

Installing five Go-based tools with `go install` compiles each one from source, which downloads a full dependency tree and writes substantial temporary build files. On an 8GB root volume this ran the disk to 92% capacity, and a build silently failed mid-compile.

A separate `/tmp` partition (`tmpfs`, capped independently of the root disk) hit its own limit shortly after, throwing a more specific "disk quota exceeded" instead of the generic "no space left on device". The cause was the same, but the reported symptom was different.

**Fix:** cleared the Go build cache (`go clean -cache`) as a stopgap, then properly resized the EBS volume from 8GB to 20GB in the AWS console, followed by `growpart` and `resize2fs` to extend the actual filesystem to match.

## Problem 2: Memory

With disk sorted, the heaviest tool (nuclei) still failed to compile, this time with `signal: killed`, which is the kernel's out-of-memory killer terminating the process. The instance only has about 900MB of RAM, the free-tier minimum.

**Fix:** added a swap file as backup memory. The first attempt (512MB) wasn't enough, so I increased it to 2GB once disk space allowed.

## The pivot

Rather than keep fighting a build process that was too heavy for the hardware, I switched strategy and downloaded prebuilt release binaries directly from each project's GitHub releases instead of compiling from source. This avoids both failure modes at once: with no compiler running, there's nothing to run out of memory or disk mid-build.

## A runtime gotcha

Even after installation succeeded, nuclei's full default template set (10,000+ checks) against a target with hundreds of subdomains was too heavy for this hardware, with a single CPU and about 900MB of RAM. Scoping the scan to `-severity critical,high` cut the workload dramatically and made it manageable.

I also found that swap doesn't survive an instance reboot unless it's reactivated manually (`sudo swapon`), so it's worth checking after any reboot mid-task.

## Takeaway

Free-tier hardware constraints aren't a footnote, they actively shape what's realistic to run somewhere. Working out which resource was the bottleneck (disk, memory or compute) before reaching for a fix mattered more than any individual command. The same error class ("ran out of room") had two completely different root causes depending on where in the pipeline it showed up.
