# RedPath

---

## Machine List

| # | Machine | Platform | OS | Difficulty | Key Technique | Status |
|---|---------|----------|----|------------|---------------|--------|
| 01 | [Photographer](./01-Photographer/writeup.md) | PG Play | Linux | Easy | SMB null session → Koken CMS file upload bypass → SUID php7.2 | Completed |
| 02 | [Inclusiveness](./02-Inclusiveness/writeup.md) | PG Play | Linux | Easy | User-Agent bypass → LFI → FTP shell upload → PATH hijacking | Completed |

---

## Skills Demonstrated

| Category | Techniques |
|----------|-----------|
| **Web** | File upload bypass, CMS exploitation, LFI, User-Agent bypass |
| **Active Directory** | |
| **Windows PrivEsc** | |
| **Linux PrivEsc** | SUID binary abuse, PATH hijacking |
| **Network** | SMB null sessions, anonymous FTP |
| **CVEs** | CVE-2020-48706 (Koken CMS file upload) |

---

*All machines completed manually — no Metasploit on initial exploitation.*
