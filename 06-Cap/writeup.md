# Cap

| Field | Details |
|-------|---------|
| **Platform** | HackTheBox |
| **OS** | Linux (Ubuntu) |
| **Difficulty** | Easy |
| **Date** | 2026-04-22 |
| **Status** | Completed |

---

## Overview

The name is a hint. This machine is about Linux capabilities — specifically what happens when a binary gets `cap_setuid` assigned to it carelessly. Getting there wasn't complicated either: a security dashboard with an IDOR vulnerability leaked a packet capture, and that capture had FTP credentials sitting in plaintext. Same credentials worked on SSH. From there, one `getcap` command and it was over.

---

## Attack Path

```
nmap → anonymous FTP fail → HTTP dashboard → IDOR (/data/1 → /data/0) → pcap download → FTP creds in cleartext → SSH → getcap → python3.8 cap_setuid → root
```

---

## Enumeration

### Open Ports

| Port | Service | Version |
|------|---------|---------|
| 21 | FTP | vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Gunicorn — Security Dashboard |

### What stood out

- FTP was open but anonymous login failed immediately
- The web dashboard had distinct URL paths: `/ip`, `/netstat`, `/data/1`
- The `/data/` path with a number in it was suspicious — that number is an ID

---

## Foothold

### IDOR on the dashboard

The dashboard at port 80 was a security monitoring tool showing network stats. Navigating around I noticed the URL pattern `/data/1` — an ID in the path is always worth testing.

Changed `1` to `0`:

```
http://10.129.37.105/data/0
```

Different data showed up — packet counts that weren't zeros. There was a download button. Downloaded the file — it was a `.pcap`.

### Credentials in the packet capture

Opened it in Wireshark and filtered by FTP. The capture showed a full FTP session in cleartext:

```
USER nathan
PASS Buck3tH4TF0RM3!
230 Login successful.
```

Tried the same credentials on SSH since FTP home directory was read-only:

```bash
ssh nathan@10.129.37.105
```

Logged in as nathan. Grabbed `user.txt` from the home directory.

---

## Privilege Escalation

### Linux Capabilities — cap_setuid on python3.8

`sudo -l` gave nothing. Moved to capabilities:

```bash
getcap -r / 2>/dev/null
```

Output:
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

`cap_setuid` means python3.8 is allowed to call `setuid()` — normally a root-only syscall. That's all you need:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

The process sets its own UID to 0, then spawns bash — which inherits root's UID. Root shell.

---

## Flags

- `user.txt` — `/home/nathan/user.txt`
- `root.txt` — `/root/root.txt`

---

## What I learned

- IDOR is simple but easy to miss — any number in a URL path is worth changing
- Packet captures leak cleartext protocols. FTP sends credentials in plaintext, always
- `getcap -r / 2>/dev/null` is now a fixed step in my Linux privesc checklist, right after `sudo -l`
- `cap_setuid` on any scripting language (python, perl, ruby) = instant root. The binary doesn't matter, the capability does
- The machine name was the hint the whole time

---

## References

- [HackTricks - Linux Capabilities](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
- [GTFOBins - Python Capabilities](https://gtfobins.github.io/gtfobins/python/#capabilities)
