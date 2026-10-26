# Risk Assessment Strategy — Mitigating Future Incidents

A five-step strategy built from the weaknesses exploited in the Frozen Yoghurt Ltd breach.

## 1. Risk identification

| ID | Risk | Evidence from the incident |
|---|---|---|
| R1 | Insecure legacy protocols | Telnet (TCP/23) open; credentials and data sent in cleartext |
| R2 | Weak authentication | Password guessed by brute force; no lockout on Telnet |
| R3 | Exposure of sensitive data | Card, employee and recovery-code files readable by one user and exfiltrated |
| R4 | Insufficient monitoring and response | IDS and firewall recorded the attack but did not stop it in time |
| R5 | Inadequate segmentation and access control | One compromised account reached email, web, documents and databases |

## 2. Risk evaluation

Each risk is rated on a **likelihood × impact** matrix (1–5 each). See the risk matrix in the [Part Two article](https://medium.com/@prospereromonsele/cyber-attack-investigation-analysis-and-incident-response-824afb961f18).

## 3. Risk mitigation plan

| Risk | Controls |
|---|---|
| **R1 — Legacy protocols** | Disable Telnet, FTP and rlogin; enforce SSH; restrict management services to an admin network |
| **R2 — Weak authentication** | Password complexity and expiry; MFA; account lockout; rate limiting on login services |
| **R3 — Data exfiltration** | Encryption at rest and in transit; role-based access control; Data Loss Prevention (DLP) |
| **R4 — Monitoring** | SIEM correlation across firewall, IDS and host logs; real-time alerts for failed-login bursts, sensitive-file access and large outbound transfers |
| **R5 — Segmentation** | Network segmentation and VLANs; least privilege; separate web, mail and database roles to limit lateral movement |

## 4. Risk monitoring and review

- **Monthly** vulnerability scans
- **Quarterly** penetration tests
- Regular **incident response drills** (tabletop and technical)
- Re-assess the risk register after every significant change or incident

## 5. Compliance

Align controls with **GDPR**, **PCI DSS** and the **NIST Cybersecurity Framework**, and audit systems and policies regularly against them.

## Conclusion

The plan targets each weakness the attacker used — an insecure protocol, a weak password, exposed sensitive data, slow detection and flat access — and turns them into monitored, layered controls, leaving Frozen Yoghurt Ltd far more resilient to the same attack and its variants.
