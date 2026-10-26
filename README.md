# Frozen Yoghurt Ltd — Cyber Attack Investigation & Incident Response

A hands-on SOC case study: investigating a compromised server from recovered network and host evidence, reconstructing the attack, and writing up the incident report, root cause and risk-mitigation strategy.

> **Scenario:** On **8 March 2022**, Frozen Yoghurt Ltd's central server (email, public website, internal documents and company databases) was compromised. Files were accessed, copied and deleted. A packet capture, firewall logs, IDS logs and server logs were recovered. The job: work out **what** happened, **how**, and **when** — then recommend how to stop it happening again.

Prepared by **TechCapacity**. Full write-ups on Medium:
- Part One — Investigation & Analysis: [Medium article](https://medium.com/@prospereromonsele/cyber-attack-investigation-analysis-and-incident-response-bd40320a6778)
- Part Two — Incident Report & Risk Assessment: [Medium article](https://medium.com/@prospereromonsele/cyber-attack-investigation-analysis-and-incident-response-824afb961f18)

---

## TL;DR

| | |
|---|---|
| **Attacker** | `192.168.56.1` |
| **Victim** | `192.168.56.102` (central server) |
| **Initial access** | Telnet (TCP/23) brute force against user `frozenyoghurt` — password `frozen` |
| **Other activity** | Web vulnerability / directory scanning on TCP/80 (many HTTP 404s); failed SSH brute force on TCP/22 (locked out) |
| **Data accessed** | Card transactions, employee details, email password-recovery codes, company credit-card info |
| **Impact** | Data exfiltration; `frozenyoghurtshop` copied and deleted; `index.html` deleted (website disrupted) |
| **Root cause** | Legacy cleartext service (Telnet) exposed + weak/default credentials + no lockout or timely alerting |

## Attack flow

```
 192.168.56.1 (attacker)                                   192.168.56.102 (server)
        │
        ├─ 14:29:25  TCP/80  Web scanning ─ repeated GET requests, many HTTP 404 ──► web server
        │
        ├─ TCP/23  Telnet brute force ─ user "frozenyoghurt", many passwords ─────► telnetd
        │
        ├─ 14:35:08  Successful Telnet login (password "frozen") ──────────────────► shell
        │             ls  →  cat sensitive files  →  copy / delete files
        ├─ 14:38:38  Logout
        │
        └─ TCP/22  SSH brute force ─ failed, max auth attempts exceeded ──────────► sshd
```

## Repository contents

| Path | What's inside |
|---|---|
| [`analysis/01-network-reconnaissance.md`](analysis/01-network-reconnaissance.md) | Findings 1–6: identifying attacker/victim from conversation stats, first contact, web scanning |
| [`analysis/02-telnet-brute-force.md`](analysis/02-telnet-brute-force.md) | Findings 7–11: Telnet brute force, correlation across PCAP / Suricata / auth / kernel logs, successful login |
| [`analysis/03-ssh-brute-force.md`](analysis/03-ssh-brute-force.md) | Finding 12: failed SSH brute force and lockout |
| [`analysis/04-post-exploitation.md`](analysis/04-post-exploitation.md) | What the attacker did once inside — files read, copied and deleted |
| [`report/incident-report.md`](report/incident-report.md) | Formal incident report: executive summary, impact, response evaluation, root cause |
| [`report/risk-assessment.md`](report/risk-assessment.md) | Risk identification, mitigation plan, monitoring and compliance |
| [`timeline-and-iocs.md`](timeline-and-iocs.md) | Consolidated timeline and indicators of compromise |
| [`mitre-attack-mapping.md`](mitre-attack-mapping.md) | Attacker behaviour mapped to MITRE ATT&CK |
| [`evidence/`](evidence/) | Where to get the lab evidence files |

## Evidence analysed

- **Network capture (PCAP)** — packet-level view of attacker ↔ server traffic
- **Firewall logs (NGFW)** — IPs, ports, protocols, allowed/denied activity
- **IDS logs (Suricata)** — signatures, behavioural alerts
- **Server logs** — authentication (`auth.log`), kernel and system events, file activity

Download link: see [`evidence/README.md`](evidence/README.md).

## Tools used

- **Wireshark** — Statistics → Conversations, Follow TCP Stream, display filters
- **Suricata** alert logs
- Linux **auth** and **kernel** logs

## Skills demonstrated

Network forensics · Log correlation across multiple sources · Brute-force detection · Attack timeline reconstruction · Incident reporting · Root cause analysis · Risk assessment · MITRE ATT&CK mapping

---

*Frozen Yoghurt Ltd is a fictional company used for a training lab. All IPs are private lab addresses.*
