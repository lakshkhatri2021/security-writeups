# TryHackMe: Pre Security Path

Beginner path covering the fundamentals that every later security topic builds on. This is a summary of the concepts I took from it, grouped by area.

## Networking

- **Models:** The OSI model has 7 layers and is mostly used for troubleshooting and talking about where a problem sits. The TCP/IP model has 4 layers (link, internet, transport, application) and is what the internet actually runs on.
- **Addressing:** A MAC address identifies a network card on the local network. An IP address identifies a device across networks. Switches forward by MAC address, routers forward by IP address.
- **TCP vs UDP:** TCP sets up a connection with a three-way handshake (SYN, SYN-ACK, ACK) and guarantees delivery and order. UDP skips all of that, so it is faster but gives no delivery guarantee (used for DNS lookups, streaming, gaming).
- **DNS:** Translates a domain name into an IP address. Your device checks its cache, then asks a recursive resolver, which walks the root, TLD and authoritative servers until it gets an answer.

## Ports

A port is a number (0 to 65,535) that tells a machine which service a packet is meant for. A service "listens" on a port, and every open port is a possible way in, which is why attackers scan ports first and defenders close the ones they don't need.

| Port | Service | Notes |
|---|---|---|
| 21 | FTP | Sends credentials in plaintext |
| 22 | SSH | Encrypted remote shell |
| 25 | SMTP | Outgoing email |
| 53 | DNS | Usually UDP, TCP for large replies |
| 80 | HTTP | Unencrypted web |
| 443 | HTTPS | HTTP over TLS |
| 3389 | RDP | Windows remote desktop, a common brute-force target |

## How the web works

1. You type a URL and the browser asks DNS for the server's IP.
2. The browser opens a TCP connection to the server (and does a TLS handshake if it's HTTPS).
3. It sends an HTTP request, made of a method (GET, POST, PUT, DELETE), a path, headers and sometimes a body.
4. The server replies with a status code, headers and the content.

Status codes worth knowing: 200 (OK), 301/302 (redirect), 401 (not authenticated), 403 (authenticated but not allowed), 404 (not found), 500 (server error). Cookies let the server recognise you between requests, which is why stealing a session cookie can mean taking over an account.

## Linux and Windows fundamentals

- **Linux:** Everything hangs off one root directory (`/`). Permissions are read, write and execute for the owner, the group and everyone else, changed with `chmod` and `chown`. Useful starting commands are `ls`, `cd`, `cat`, `grep`, `find` and `sudo`.
- **Windows:** Uses the NTFS file system with access control lists per file. User accounts are managed locally or centrally through Active Directory. Settings live in the Registry, and User Account Control (UAC) asks before an action runs with admin rights.
- **Operating systems in general:** The OS manages processes, memory, storage and permissions between users and programs, so misconfigured permissions and outdated systems are among the most common ways in.

## Cloud computing

Instead of owning hardware, you rent it. The three service models differ in how much you manage yourself.

| Model | You manage | Example |
|---|---|---|
| IaaS | OS, apps, data | AWS EC2 |
| PaaS | Apps and data | Heroku, Elastic Beanstalk |
| SaaS | Just your data and access | Gmail, Microsoft 365 |

Under the shared responsibility model, the provider secures the physical infrastructure and you secure what you put on it (configuration, access control, data). Most real cloud breaches come from the customer's side, such as public storage buckets.

## Cryptography

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared secret key | Public key and private key pair |
| Speed | Fast | Slow |
| Examples | AES | RSA, ECC |
| Used for | Encrypting bulk data | Key exchange, digital signatures |
| Weak point | Sharing the key safely | Performance |

HTTPS combines both: asymmetric cryptography is used to agree on a key safely, then symmetric encryption protects the actual traffic. Hashing is different from encryption because it's one-way. It's used to check integrity and to store passwords.

## The CIA triad

| Principle | Meaning | Example attack | Example control |
|---|---|---|---|
| Confidentiality | Only authorised people can read the data | Data breach | Encryption, access control |
| Integrity | Data hasn't been changed without authorisation | Tampering with a transaction | Hashing, digital signatures |
| Availability | Systems and data are usable when needed | DDoS, ransomware | Backups, redundancy |

## Attacks and defences

Offensive teams (red) find weaknesses by attacking, and defensive teams (blue) detect and respond to them. Defence in depth means layering controls (firewall, patching, least privilege, monitoring, backups) so that one failure doesn't mean total compromise.
