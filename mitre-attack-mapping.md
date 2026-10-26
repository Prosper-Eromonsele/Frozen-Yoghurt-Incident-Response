# MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence in this incident |
|---|---|---|---|
| Reconnaissance | Active Scanning: Vulnerability Scanning | [T1595.002](https://attack.mitre.org/techniques/T1595/002/) | Automated HTTP probing of the web server on TCP/80 |
| Reconnaissance | Active Scanning: Wordlist Scanning | [T1595.003](https://attack.mitre.org/techniques/T1595/003/) | Many requests for non-existent paths returning HTTP 404 |
| Credential Access | Brute Force: Password Guessing | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) | Repeated Telnet logins for `frozenyoghurt`; repeated SSH logins |
| Initial Access / Persistence | Valid Accounts | [T1078](https://attack.mitre.org/techniques/T1078/) | Login with the guessed credentials `frozenyoghurt` / `frozen` |
| Discovery | File and Directory Discovery | [T1083](https://attack.mitre.org/techniques/T1083/) | `ls` after login |
| Collection | Data from Local System | [T1005](https://attack.mitre.org/techniques/T1005/) | `cat` on card, employee and credit-card files |
| Credential Access | Unsecured Credentials: Credentials In Files | [T1552.001](https://attack.mitre.org/techniques/T1552/001/) | `email-password-recovery-code.csv` read |
| Exfiltration | Exfiltration Over C2 Channel | [T1041](https://attack.mitre.org/techniques/T1041/) | File contents returned over the interactive Telnet session; outbound traffic in PCAP |
| Impact | Data Destruction | [T1485](https://attack.mitre.org/techniques/T1485/) | `frozenyoghurtshop` and `index.html` deleted |

## Detection opportunities

| Technique | Detection |
|---|---|
| T1595 | Alert on one source generating a high rate of HTTP 404s |
| T1110.001 | Alert on N failed logins for one account or from one source within M minutes |
| T1078 | Alert on a successful login immediately following a burst of failures for the same account |
| T1552.001 / T1005 | File-integrity / audit monitoring on directories holding payment and credential data |
| T1485 | Alert on deletion of web root files such as `index.html` |
