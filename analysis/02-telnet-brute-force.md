# 02 — Telnet Brute Force & Initial Access (Findings 7–11)

Evidence: **PCAP**, **Suricata IDS logs**, **authentication logs**, **kernel logs**

With the web scan going nowhere, the attacker turns to the **Telnet** service on **TCP/23** — a legacy remote-login protocol that sends usernames, passwords and every command in **cleartext**.

---

## Finding #7 — Telnet connection

A TCP three-way handshake from `192.168.56.1` to `192.168.56.102:23` opens the Telnet phase.

## Finding #8 — Guessing credentials

**Follow → TCP Stream** on the Telnet connections shows the login prompt and the attacker's repeated, unsuccessful attempts to guess a username and password. Because Telnet is unencrypted, every guess is readable in the capture.

## Finding #9 — Brute force confirmed across four log sources

The streams show a high number of login attempts against the username **`frozenyoghurt`** (written `frozen-yoghurt` in some attempts) with a different password each time, coming from **multiple source ports** at machine-speed intervals — the signature of an automated brute-force tool.

The same activity is recorded independently in:

| Source | What it shows |
|---|---|
| PCAP (TCP streams) | Each username/password pair sent in cleartext |
| Suricata IDS | Alerts on repeated Telnet login attempts from `192.168.56.1` |
| Authentication log | Run of failed logins for `frozenyoghurt` |
| Kernel log | Matching connection events on port 23 |

Seeing the same event in four independent sources removes any doubt that `192.168.56.1` is the attacker.

**Credentials obtained:** username **`frozenyoghurt`**, password **`frozen`** — a weak password derived from the username.

## Finding #10 — Successful login

The authentication log records a **successful Telnet login at 14:35:08**.

## Finding #11 — Session and logout

The attacker stays logged in, runs commands on the server (see [04](04-post-exploitation.md)), then logs out at **14:38:38**.

---

## Reproduce it

**Wireshark**

| Goal | Display filter |
|---|---|
| All Telnet traffic | `telnet` or `tcp.port == 23` |
| Telnet connections from the attacker | `ip.src == 192.168.56.1 && tcp.dstport == 23 && tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| Cleartext data sent by the attacker | `ip.src == 192.168.56.1 && telnet.data` |

Then **Follow → TCP Stream** on any connection to read the login exchange.

**Host logs**

```bash
grep -i telnet auth.log
grep -i "authentication failure" auth.log
grep -i "frozenyoghurt" auth.log
```

## Why it worked

- Telnet was enabled and reachable — there is no reason to run it on a modern server.
- The account password was trivially guessable.
- No account lockout or rate limiting on Telnet logins.
- IDS alerts fired, but nothing blocked the source or alerted an analyst in time.
