<div align="center">
  <h1>🔍 The Ultimate Shodan Search Guide</h1>
  <p><b>Advanced Reconnaissance, Exposed Asset Discovery, and Ethical Reporting</b></p>
  <p><i>Authored by <a href="https://github.com/mejbankadir">Mejban Kadir</a> (Security Researcher, Developer & Trader)</i></p>
</div>

---

## 📖 Introduction

Welcome to the definitive guide on Shodan. While standard search engines index web pages, Shodan indexes the underlying infrastructure of the internet—servers, routers, webcams, industrial control systems, and IoT hardware. 

As an ethical hacker, knowing how to navigate and filter this massive ocean of data is critical. Whether I'm deep into analyzing 1-minute Forex charts to catch a liquidity trap or writing custom Rust and Python recon tooling on my Omarchy Linux setup, efficient and targeted intelligence gathering is always the first step in my workflow. 

This repository will walk you through real-world scenarios, demonstrate how to discover leaked infrastructure, and provide actionable queries to level up your bug bounty game.

---

## 🚀 Real-World Use Cases & Prior Experience

### Streamlining Reconnaissance
During my time hunting on platforms like HackerOne and developing technical content for SMH Tech | NexoAmicus, I realized how much time gets wasted manually copying and extracting targets from search pages. This bottleneck led me to build the **Shodan IP/Domain Extractor** extension for Firefox. By automating the extraction of IP addresses and domains directly from search results into text files, you can immediately feed targets into your terminal scripts and vulnerability scanners. 

### Identifying Shadow IT
One of the most common issues organizations face is "Shadow IT"—infrastructure spun up by developers for testing and subsequently forgotten. By mapping an organization's ASN or using precise `org:` filters, I have consistently uncovered exposed staging environments, misconfigured databases, and internal developer tools that were accidentally left public. 

---

## 🕵️‍♂️ Finding Leaked IPs & Responsible Reporting

### The Discovery Phase
Finding leaked IPs usually comes down to chaining specific filters. Companies often try to hide their origin servers behind Web Application Firewalls (WAFs) like Cloudflare, but simple misconfigurations can expose the direct IP. 

1. **SSL/TLS Certificates:** Use `ssl:"company name"` to find IPs hosting the target's certificate outside of their WAF.
2. **Favicon Hashing:** Calculate the MurmurHash of a company's favicon and search for it across the internet using `http.favicon.hash:`.
3. **HTTP Titles:** Search for distinct internal dashboard titles using `title:"Internal Dashboard" org:"Company"`.

### The Reporting Process (Bug Bounty)
When you discover a leaked IP or exposed asset, professionalism is paramount. Ensure you don't just assume it's a vulnerability—verify the impact ethically. When writing a report, I always assure the triage team that no data was accessed, downloaded, or modified during my verification process.

1. **Verify Ownership:** Confirm the IP actually belongs to the target company (check the ASN, WHOIS records, or SSL certificates).
2. **Document the Impact:** Clearly explain *why* the leaked IP is dangerous (e.g., it bypasses the WAF, exposes administrative panels, or leaks PII).
3. **Submit via Official Channels:** Use the company's Vulnerability Disclosure Program (VDP) or platforms like HackerOne. Provide clear, reproducible steps so the security team can patch the leak immediately.

---

## 🛠️ Comprehensive Shodan Query Reference

Shodan's power comes from mixing global search tokens with highly specific filters. Below is an exhaustive breakdown of the available filters categorized by operational intent.

### 1. Network & Infrastructure Filters
These filters focus on the routing, hosting, and physical network placement of an asset.

| Filter | Description | Real-World Example |
| :--- | :--- | :--- |
| `asn` | Filters by Autonomous System Number (ASN) | `asn:AS15169` *(Finds Google infrastructure)* |
| `net` | Searches within a specific CIDR network block or IP | `net:192.168.1.0/24` |
| `org` | Limits results to a specific organization name | `org:"Microsoft Corporation"` |
| `isp` | Filters by the Internet Service Provider | `isp:"Comcast"` |
| `port` | Targets specific open network ports | `port:22` *(Looks for open SSH)* |
| `transport` | Specifies the transport layer protocol (`tcp` or `udp`) | `port:53 transport:udp` *(DNS over UDP)* |

### 2. Geographic & Location Filters
Used to isolate assets to physical areas, making regional exposure assessments highly efficient.

| Filter | Description | Real-World Example |
| :--- | :--- | :--- |
| `country` | Limits results to a 2-letter country code | `port:80 country:BD` *(Web servers in Bangladesh)* |
| `city` | Targets a specific city name | `product:Apache city:"Dhaka"` |
| `geo` | Filters by precise coordinates (Latitude, Longitude, Radius) | `geo:23.8103,90.4125,10` *(Within 10km of Dhaka)* |
| `postal` | Limits search by postal code | `postal:1212` |

### 3. HTTP & Web Asset Metadata
Crucial for web-facing application auditing, finding admin dashboards, and fingerprinting frameworks.

| Filter | Description | Real-World Example |
| :--- | :--- | :--- |
| `http.component` | Targets web frameworks, apps, or plugins detected | `http.component:"WordPress"` |
| `http.favicon.hash` | Searches for assets matching a specific favicon MurmurHash | `http.favicon.hash:-1588080174` *(Cobalt Strike beacons)* |
| `http.html` | Searches the raw HTML body content of the response | `http.html:"dashboard"` |
| `http.status` | Filters by the HTTP status code returned | `http.status:403` *(Forbidden access panels)* |
| `http.title` | Matches the exact text inside the web page text title tags | `http.title:"Index of /"` *(Directory listing)* |

### 4. SSL/TLS & Encryption Certificate Properties
Excellent for finding hidden assets and origin servers bypassing edge configurations.

| Filter | Description | Real-World Example |
| :--- | :--- | :--- |
| `ssl` | Searches for elements inside the SSL certificate | `ssl:"Tesla"` |
| `ssl.cert.issuer.cn` | Filters by the Common Name of the certificate issuer | `ssl.cert.issuer.cn:"Let's Encrypt"` |
| `ssl.cert.subject.cn` | Targets the specific domain name registered on the cert | `ssl.cert.subject.cn:"*.domain.com"` |
| `ssl.version` | Isolates targets based on their TLS protocol capability | `ssl.version:tls1.0` *(Finding deprecated security)* |

### 5. Hardware, Software & Device Protocols
Used to query the fingerprints banner data returns for specific operating systems, tech stacks, or IoT hardware.

| Filter | Description | Real-World Example |
| :--- | :--- | :--- |
| `os` | Targets the underlying Operating System configuration | `os:"Linux 3.x"` |
| `product` | Searches by software or vendor product name | `product:"nginx"` |
| `version` | Targets specific versions of software deployments | `product:"OpenSSH" version:"7.2p2"` |
| `device` | Filters by classification of hardware (e.g., webcam, router) | `device:webcam` |

---

## 🎯 Advanced Playbooks: Production-Ready Queries

### Threat Hunting & Command and Control (C2) Tracking
```text
# Locate exposed Metasploit RPC servers
"MetasploitRPC" port:55553

# Find potential Cobalt Strike Team Servers via default port profiles
port:50050 "HTTP/1.1 404 Not Found"

# Track down active AnyDesk installations exposed publicly
port:7070 "AnyDesk"# Unauthenticated Jenkins Automation Dashboards
"Dashboard [Jenkins]" http.status:200

# Open Git Directories exposing source control structure
"Index of /" http.html:".git"

# Exposed Docker Remote API Instances
port:2375 "Docker" http.status:200

# Open Redis instances allowing remote key extraction
port:6379 "-ERR unknown command"


# Locate open Modbus devices (often used in factories and plants)
port:502 port:502

# Exposed BACnet controllers for building automation systems
port:47808 original-address

# SCADA Niagara Fox instances running automation software
port:1911,4911 "Niagara"
```
🔗 Essential Links
• Shodan Official Help Center https://help.shodan.io/
• Shodan Developer API Documentation https://developer.shodan.io/
• Mejban Kadir's Shodan IP/Domain Extractor https://addons.mozilla.org/en-US/firefox/addon/shodan-ip-domain-extractor/
📈 SEO & Discovery
Search Tags: Mejban Kadir, mejbankadir, Security Researcher, Ethical Hacker, Bug Bounty Hunter, Shodan Reconnaissance, OSINT Tools, Cybersecurity, Rust Developer, Python Automation, SMH Tech, NexoAmicus, Forex Scalper.
About Mejban Kadir:
Mejban Kadir (mejbankadir) is a multifaceted Cybersecurity Researcher, Software Developer, and Trader. Known for his work in penetration testing, vulnerability management, and open-source intelligence (OSINT), Mejban creates tools that empower the infosec community. Whether finding critical CVEs, writing high-performance code, or analyzing technical charts, Mejban blends deep technical expertise with practical execution. Find more of his security research and development projects online.



