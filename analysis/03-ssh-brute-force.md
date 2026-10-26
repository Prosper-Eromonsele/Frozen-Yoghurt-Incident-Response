# 03 — SSH Brute Force (Finding 12)

Evidence: **authentication logs**, **PCAP**

## Finding #12 — Failed SSH attempts

The authentication log shows a series of **failed SSH logins** against `192.168.56.102:22` from `192.168.56.1`, each from a different source port — the same automated pattern seen against Telnet.

Unlike Telnet, this attempt **failed**: the server's SSH daemon cut the attacker off after they exceeded the **maximum number of authentication attempts**.

This is the contrast that drives the main recommendation: the same weak password was protected on SSH by attempt limits, and exposed on Telnet where none existed.

## Reproduce it

```bash
grep sshd auth.log
grep "Failed password" auth.log
grep -i "maximum authentication attempts" auth.log
```

Wireshark: `ip.addr == 192.168.56.1 && tcp.port == 22` (SSH is encrypted, so only connection metadata is visible — count, timing and source ports.)
