# Local-Network-for-Open-Ports
# Task 1: Network Port Scanning with Nmap

## Objective
Perform network reconnaissance on a local network to discover active hosts and open ports, understand service exposure, and identify potential security risks.

---

## Tools Used
| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | 7.95 | Network scanning & port discovery |
| Kali Linux | Latest | Operating system used |
| Wireshark | Optional | Packet capture & analysis |

---

## Environment
- **OS:** Kali Linux
- **Network Interface:** eth0
- **Local IP:** 192.168.0.168/24
- **Target Range Scanned:** 192.168.1.0/24

---

## Steps Performed

### 1. Identify Local Network
```bash
ip a
```
Identified local IP as `192.168.0.168` on `eth0`. Scanned the `192.168.1.0/24` subnet.

### 2. Basic TCP SYN Scan
```bash
sudo nmap -sS 192.168.1.0/24
```

### 3. Detailed Scan with Service Version Detection
```bash
sudo nmap -sS -sV 192.168.1.0/24 -oN scan_results.txt
```

### 4. HTML Report Export
```bash
sudo nmap -sS -sV 192.168.1.0/24 -oX scan_results.xml
xsltproc scan_results.xml -o scan_results.html
```

---

## Scan Results Summary

**Hosts Discovered: 4 out of 256 IPs**

| IP Address | Device Type | Open Ports |
|------------|------------|-----------|
| 192.168.1.1 | Router/Gateway | 21, 53, 80, 443, 8000 |
| 192.168.1.33 | Server/Pi | 22, 53, 1900 |
| 192.168.1.34 | Apache Tomcat Server | 8009 |
| 192.168.1.43 | Unknown | None (all closed) |

---

## Security Risks Identified

| Severity | Host:Port | Service | Risk |
|----------|-----------|---------|------|
| 🔴 CRITICAL | 192.168.1.34:8009 | AJP13 | Ghostcat (CVE-2020-1938) — unauthenticated file read / RCE |
| 🟠 HIGH | 192.168.1.1:21 | FTP | Plaintext credentials, easily intercepted |
| 🟠 HIGH | 192.168.1.1:23 | Telnet | Unencrypted legacy protocol |
| 🟠 HIGH | 192.168.1.1:445 | SMB | EternalBlue (MS17-010) exploit risk |
| 🟠 HIGH | 192.168.1.33:1900 | UPnP | Unauthorized port forwarding / SSRF attacks |
| 🟡 MEDIUM | 192.168.1.1:53 | DNS | DNS amplification attack risk |
| 🟡 MEDIUM | 192.168.1.1:80 | HTTP | Unencrypted web admin panel |
| 🟡 MEDIUM | 192.168.1.33:22 | SSH | Brute force risk |
| 🟡 MEDIUM | 192.168.1.1:8000 | HTTP-alt | Possibly unsecured admin interface |

---

## Recommendations

1. **Disable AJP (port 8009)** immediately — patch Ghostcat CVE-2020-1938 on Tomcat
2. **Replace FTP** with SFTP or FTPS to encrypt credentials
3. **Disable Telnet** entirely — use SSH only
4. **Disable UPnP** on router if not explicitly required
5. **Restrict SMB (445)** via firewall; ensure EternalBlue patch is applied
6. **Enable SSH key auth** on 192.168.1.33; disable password-based login
7. **Force HTTPS** — redirect port 80 to 443 for all admin panels
8. **Restrict DNS** to internal queries only to prevent amplification abuse

---

## Key Learnings

- **TCP SYN Scan (-sS):** Sends a SYN packet and waits for SYN-ACK (open) or RST (closed). Never completes the 3-way handshake — making it stealthy and fast. Requires root/admin.
- **Filtered vs Open:** Filtered ports are blocked by a firewall; open ports are actively listening.
- **Port-to-Service Mapping:** Common ports (21=FTP, 22=SSH, 53=DNS, 80=HTTP, 443=HTTPS, 445=SMB) reveal what services are running on a device.
- **Real Vulnerabilities Found:** CVE-2020-1938 (Ghostcat) on port 8009 is a real critical vulnerability — this shows why port scanning matters.
- **Defense Mindset:** Attackers use the exact same tools to find entry points — understanding offense is key to building better defenses.

---

## Interview Q&A

**Q: What is an open port?**
A port actively accepting connections; a service is bound and listening on it.

**Q: How does Nmap perform a TCP SYN scan?**
Sends a SYN packet → receives SYN-ACK (open) or RST (closed) → sends RST to abort. Never completes the handshake. Fast and stealthy; requires root.

**Q: What risks are associated with open ports?**
Each open port is a potential attack surface — exploitable services, brute force, unauthorized access.

**Q: TCP vs UDP scanning?**
TCP is connection-oriented (reliable, detectable); UDP is connectionless (harder to scan, requires different techniques like `-sU`).

**Q: How can open ports be secured?**
Disable unused services, use firewalls, patch software, use encrypted alternatives (SFTP over FTP, SSH over Telnet).

**Q: What is a firewall's role?**
Filters incoming/outgoing traffic based on rules — can block specific ports or IP ranges.

**Q: Why do attackers port scan?**
To discover running services, identify software versions, and find known vulnerabilities to exploit.

**Q: How does Wireshark complement Nmap?**
Captures actual packets at the network level — you can see the SYN/SYN-ACK/RST exchange visually and analyze raw traffic.

---

## Files in This Repository

```
network-port-scanning-task/
├── README.md               ← This file
├── scan_results.txt        ← Raw Nmap output with risk analysis
├── scan_results.html       ← Formatted HTML report
└── screenshots/            ← Terminal screenshots (add your own)
```

---

## Disclaimer
This scan was performed **only on my own local network** for educational purposes as part of a cybersecurity internship task. Unauthorized port scanning of networks you do not own is illegal.
