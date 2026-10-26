# 01 — Network Analysis & Web Reconnaissance (Findings 1–6)

Evidence: **PCAP** · Tool: **Wireshark**

In incident response, *findings* answer three questions: **what** happened, **how** it happened, and **when**. The investigation starts with the network capture to work out who was talking to whom.

---

## Finding #1 — Conversation statistics

**Wireshark → Statistics → Conversations** lists every host pair in the capture.

Two hosts dominate the traffic:

| Label | IP | Role (provisional) |
|---|---|---|
| IP address A | `192.168.56.1` | Attacker |
| IP address B | `192.168.56.102` | Victim (central server) |

Between them they account for **7,165 packets** and roughly **4 MB** of data — far more than any other conversation.

## Finding #2 — Who initiated contact

The TCP tab of the Conversations window shows `192.168.56.1` repeatedly opening connections **to** `192.168.56.102` on three ports:

| Port | Service |
|---|---|
| 22 | SSH |
| 23 | Telnet |
| 80 | HTTP |

Because `192.168.56.1` is the initiator on all three, it is treated as the **attacker** and `192.168.56.102` as the **victim**. This is a working hypothesis at this point, confirmed by the host logs in later findings.

## Finding #3 — First contact

The first packet from the attacker to the server is at **14:29:25 on 8 March 2022**: a TCP three-way handshake (SYN → SYN/ACK → ACK) to the web server on **port 80**.

## Finding #4 — HTTP GET request

**Follow → TCP Stream** on that first connection shows that, straight after the handshake, the attacker sends an **HTTP GET** request for a resource on the company website.

## Finding #5 — The pattern repeats

The handshake-then-GET pattern repeats many times, each time from a **different source port** on the attacker's machine. **Statistics → Conversations** filtered to TCP/80 shows the volume of short-lived connections.

## Finding #6 — Automated web vulnerability scanning

The number of connections, their speed and the paths requested point to an **automated web vulnerability / directory scanner** probing for vulnerabilities and hidden directories. Tools that behave this way include Nikto, OWASP ZAP, Burp Suite, Acunetix and Nessus.

Many requests come back **HTTP 404 Not Found** — the scanner is guessing paths that don't exist. The web probing does not appear to give the attacker a way in, and they move on to Telnet (see [02](02-telnet-brute-force.md)).

---

## Reproduce it in Wireshark

| Goal | Display filter |
|---|---|
| All traffic between the two hosts | `ip.addr == 192.168.56.1 && ip.addr == 192.168.56.102` |
| New connections from the attacker (SYNs) | `ip.src == 192.168.56.1 && tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| Attacker's HTTP requests | `ip.src == 192.168.56.1 && http.request` |
| 404 responses (scanner misses) | `http.response.code == 404` |

Useful menus: **Statistics → Conversations**, **Statistics → HTTP → Requests**, **Statistics → Endpoints**, **Follow → TCP Stream**.
