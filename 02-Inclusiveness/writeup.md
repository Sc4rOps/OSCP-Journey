# Inclusiveness

| Field | Details |
|-------|---------|
| **Platform** | OffSec Proving Grounds Play |
| **OS** | Linux |
| **Difficulty** | Easy (Community: Intermediate) |
| **Date** | 2026-04-08 |
| **Status** | Completed |

---

## Attack Path

```
Anonymous FTP (writable) → User-Agent bypass → LFI → PHP shell via /var/ftp/pub/ → www-data → PATH hijack → root
```

---

## What happened

Nmap showed FTP, SSH and a web server. FTP allowed anonymous login and the `pub` folder was writable — filed that away.

The web server was just an Apache default page. Gobuster found `robots.txt` but it blocked me with "You are not a search engine." Switched the User-Agent to Googlebot and it revealed `/secret_information/` directory.

That page talked about DNS zone transfer attacks. Spent some time on that rabbit hole before realizing the actual attack was the `?lang=` parameter in `index.php` — it was vulnerable to LFI.

Since FTP `pub` maps to `/var/ftp/pub/` on the server, I uploaded a PHP webshell there via FTP, then included it through the LFI to get command execution. Reverse shell came back as `www-data`.

---

## Privilege Escalation

Found `/home/tom/rootshell` — a SUID binary with its C source code sitting right next to it. The code called `whoami` without a full path and gave a shell if the result was "tom".

PATH hijacking:

```bash
echo "printf 'tom'" > /tmp/whoami
chmod +x /tmp/whoami
export PATH=/tmp:$PATH
/home/tom/rootshell
```

Root.

---

## What I learned

- User-Agent checks are easy to bypass — always worth trying Googlebot if access is blocked
- LFI + writable FTP directory is a classic combo
- When a SUID binary calls commands without full path, check if you can hijack PATH
- `strings` on any binary you don't understand — it shows you what commands it calls

---

## References

- [HackTricks - LFI](https://book.hacktricks.xyz/pentesting-web/file-inclusion)
- [PayloadsAllTheThings - File Inclusion](https://swisskyrepo.github.io/PayloadsAllTheThings/File%20Inclusion/)
