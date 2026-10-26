# Cyber Incident Report — Frozen Yoghurt Ltd

| | |
|---|---|
| **Incident date** | 8 March 2022 |
| **Prepared by** | TechCapacity |
| **Report date** | 15 February 2025 |
| **Incident type** | Data breach, unauthorized file manipulation, system compromise |
| **Exploited weakness** | Telnet (TCP/23) left open; brute force against weak user credentials |
| **Attacker** | `192.168.56.1` |
| **Affected host** | `192.168.56.102` — central server |

---

## 1. Executive summary

On 8 March 2022, Frozen Yoghurt Ltd's central server — hosting email, the public website, internal documents and all company databases — was compromised. The attacker gained access by brute-forcing a weak password over **Telnet (port 23)**, read sensitive customer, employee and financial files, copied the `frozenyoghurtshop` file, and deleted `frozenyoghurtshop` and `index.html`, disrupting the website.

Network and host logs recovered after the incident allowed the full sequence of events to be reconstructed. This report sets out what happened, evaluates the response, identifies the root cause and recommends controls to prevent recurrence.

## 2. Incident overview

**Affected systems** (all on the central server): email services · public website · internal documents · company databases

**Files affected**

| Action | Files |
|---|---|
| Copied | `frozenyoghurtshop` |
| Deleted | `frozenyoghurtshop`, `index.html` |
| Accessed | `email-password-recovery-code.csv`, `employee-details.txt`, `electronic-card-transactions-january-2022-csv-tables.csv`, `credit-card-info.txt` |

## 3. Recovered logs

| Log | Contribution to the investigation |
|---|---|
| Network capture (PCAP) | Packet-level record of attacker ↔ server traffic; cleartext Telnet session showing credentials, commands and data exfiltrated |
| Firewall logs (NGFW) | IPs, ports and protocols; identified the point of entry |
| IDS logs (Suricata) | Signature and behavioural alerts; identified attacker TTPs (scanning, brute force) |
| Server logs | Authentication, kernel and file-activity records; established the timeline and which files were touched |

## 4. Investigation and analysis

Full technical findings are in [`../analysis/`](../analysis/). In summary:

1. **Reconnaissance** — automated web scanning against TCP/80 from 14:29:25, producing many HTTP 404s.
2. **Initial access** — Telnet brute force against user `frozenyoghurt`; successful login at **14:35:08** with password `frozen`. Corroborated by PCAP, Suricata, auth and kernel logs.
3. **Discovery & collection** — `ls`, then `cat` on four sensitive files.
4. **Exfiltration** — file contents sent back over the cleartext Telnet session; significant outbound traffic visible in the PCAP.
5. **Impact** — `frozenyoghurtshop` copied and deleted; `index.html` deleted. Logout at **14:38:38**.
6. **Further attempts** — SSH brute force on TCP/22, blocked after maximum authentication attempts were exceeded.

**What each log showed**

- **PCAP:** substantial outbound traffic from the server to the attacker, consistent with exfiltration.
- **Firewall:** a run of failed connection attempts followed by successful connections from the attacker's IP.
- **IDS:** brute-force alerts, access to critical files and unauthorized data transfer.
- **Server logs:** the attacker's login, the commands run, and the copy/delete actions — deletion apparently intended to hinder recovery.

## 5. Impact evaluation

| Area | Impact |
|---|---|
| **Business operations** | Website and internal systems disrupted — lost revenue and customer confidence |
| **Data security** | Customer card data, employee PII and email recovery codes exposed — risk of fraud, identity theft and account takeover |
| **Legal / regulatory** | Potential liability under **GDPR** (personal data) and **PCI DSS** (cardholder data) |
| **Reputation** | Significant damage if exposure of customer card data becomes public |

## 6. Critical evaluation of the response

| Phase | What went well | What fell short |
|---|---|---|
| **Detection** | Suspicious file activity was eventually flagged | Telnet brute force on port 23 was not identified as the cause straight away; alerts did not prevent the compromise |
| **Containment** | Affected systems isolated, stopping further damage | Data had already left the network |
| **Communication** | Prompt coordination between IT and management | Notification of customers and regulators still outstanding |
| **Recovery** | Logs recovered; website restored from backup | Full scope of exfiltration and recovery of other files still under investigation |

## 7. Root cause analysis

1. **Insecure legacy service** — Telnet, an unencrypted protocol, was enabled and reachable.
2. **Weak credentials** — a password easily guessed from the username, on an account with access to sensitive data.
3. **No brute-force protection on Telnet** — no lockout or rate limiting (SSH, by contrast, did lock the attacker out).
4. **Detection without response** — IDS alerts fired but did not lead to blocking or analyst action in time.
5. **Weak access control** — a single user account could read payment, employee and credential files.

## 8. Recommendations

1. **Disable Telnet** and use SSH with key-based authentication only.
2. **Strengthen authentication** — password complexity, lockout after failed attempts, and MFA for access to critical systems.
3. **Regular vulnerability scanning** to find open ports, outdated services and misconfigurations before attackers do.
4. **Tune and act on detection** — IDS/SIEM alerts for repeated failed logins, unusual logins, sensitive-file access and large outbound transfers, with automated blocking where possible.
5. **Protect sensitive data** — encrypt it at rest, restrict it by role, and remove credential/recovery files from general-access locations.
6. **Security awareness training** for staff on passwords, phishing and social engineering.

The full risk-based mitigation strategy is in [`risk-assessment.md`](risk-assessment.md).
