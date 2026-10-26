# Timeline & Indicators of Compromise

## Timeline — 8 March 2022

| Time | Source | Event |
|---|---|---|
| 14:29:25 | PCAP | First contact: TCP handshake from `192.168.56.1` to `192.168.56.102:80` |
| 14:29:25 onward | PCAP | Automated web scanning — repeated HTTP GETs from many source ports, many HTTP 404 responses |
| After web scan | PCAP, Suricata, auth, kernel | Telnet brute force on TCP/23 against user `frozenyoghurt` |
| **14:35:08** | auth log | **Successful Telnet login** — `frozenyoghurt` / `frozen` |
| 14:35:08 – 14:38:38 | PCAP, server logs | `ls`; `cat` on four sensitive files; `frozenyoghurtshop` copied; `frozenyoghurtshop` and `index.html` deleted |
| **14:38:38** | auth log | Attacker logs out |
| — | auth log | SSH brute force on TCP/22 — fails, max authentication attempts exceeded |

## Indicators of compromise

**Network**

| Type | Value | Notes |
|---|---|---|
| Attacker IP | `192.168.56.1` | Source of all scanning, brute force and the interactive session |
| Victim IP | `192.168.56.102` | Central server |
| Port | TCP/80 | Web vulnerability / directory scanning |
| Port | TCP/23 | Telnet brute force → initial access |
| Port | TCP/22 | SSH brute force (failed) |
| Behaviour | Bursts of HTTP 404 from one source | Directory brute forcing |
| Behaviour | Many Telnet/SSH connections, rotating source ports, machine-speed timing | Automated password guessing |

**Host**

| Type | Value |
|---|---|
| Compromised account | `frozenyoghurt` (weak password) |
| Files accessed | `electronic-card-transactions-january-2022-csv-tables.csv`, `employee-details.txt`, `email-password-recovery-code.csv`, `credit-card-info.txt` |
| File copied | `frozenyoghurtshop` |
| Files deleted | `frozenyoghurtshop`, `index.html` |
| Commands | `ls`, `cat` |

## Immediate actions for these IOCs

- Block `192.168.56.1` at the firewall and review all other activity from it
- Reset the `frozenyoghurt` password and audit that account's privileges
- Invalidate all codes in `email-password-recovery-code.csv` and reset affected mailboxes
- Cancel and reissue the cards in `credit-card-info.txt`; notify the card issuer
- Restore `index.html` and `frozenyoghurtshop` from known-good backup
