# OSCP Journey — Proving Grounds Portfolio

> 30 machines. Manual exploitation. No Metasploit.
> Working toward OSCP certification — documenting every step.

---

## Progress

![Completed](https://img.shields.io/badge/Completed-20%2F30-brightgreen)
![In Progress](https://img.shields.io/badge/In%20Progress-1-yellow)
![Remaining](https://img.shields.io/badge/Remaining-9-lightgrey)

---

## Machine List

| # | Machine | Platform | OS | Difficulty | Key Technique | Status |
|---|---------|----------|----|------------|---------------|--------|
| 01 | [Twiggy](./01-Twiggy/writeup.md) | PG Practice | Linux | Easy | SaltStack API RCE (CVE-2020-11651) | ✓ |
| 02 | [Bratarina](./02-Bratarina/writeup.md) | PG Practice | Linux | Easy | OpenSMTPD RCE (CVE-2020-7247) | ✓ |
| 03 | [ClamAV](./03-ClamAV/writeup.md) | PG Practice | Linux | Easy | Sendmail + ClamAV milter bind shell | ✓ |
| 04 | [FunboxEasy](./04-FunboxEasy/writeup.md) | PG Practice | Linux | Easy | PHP file upload + sudo time privesc | ✓ |
| 05 | [Wheels](./05-Wheels/writeup.md) | PG Practice | Linux | Easy | XPath injection + SUID privesc | ✓ |
| 06 | [Exfiltrated](./06-Exfiltrated/writeup.md) | PG Practice | Linux | Easy | Subrion upload + exiftool CVE-2021-22204 | ✓ |
| 07 | [Hutch](./07-Hutch/writeup.md) | PG Practice | Windows AD | Intermediate | LDAP creds + WebDAV ASPX + GodPotato | ✓ |
| 08 | [Resourced](./08-Resourced/writeup.md) | PG Practice | Windows AD | Intermediate | ntds.dit + RBCD + Pass-the-Ticket | ✓ |
| 09 | [Access](./09-Access/writeup.md) | PG Practice | Windows AD | Intermediate | .htaccess bypass + Kerberoast + SeManageVolumePrivilege | ✓ |
| 10 | [Slort](./10-Slort/writeup.md) | PG Practice | Windows | Intermediate | RFI + writable scheduled task binary | ✓ |
| 11 | [Jacko](./11-Jacko/writeup.md) | PG Practice | Windows | Intermediate | H2 DB JNI exploit + GodPotato | ✓ |
| 12 | [Zino](./12-Zino/writeup.md) | PG Practice | Linux | Intermediate | SMB null + Booked Scheduler RCE + cronjob | ✓ |
| 13 | [Banzai](./13-Banzai/writeup.md) | PG Practice | Linux | Intermediate | FTP upload + MySQL UDF privesc | ✓ |
| 14 | [Nibbles](./14-Nibbles/writeup.md) | PG Practice | Linux | Intermediate | PostgreSQL default creds + COPY FROM PROGRAM | ✓ |
| 15 | [Pelican](./15-Pelican/writeup.md) | PG Practice | Linux | Intermediate | Exhibitor RCE + gcore memory dump | ✓ |
| 16 | [Billyboss](./16-Billyboss/writeup.md) | PG Practice | Windows | Intermediate | Sonatype Nexus CVE-2020-10199 + GodPotato | ✓ |
| 17 | [Heist](./17-Heist/writeup.md) | PG Practice | Windows AD | Hard | SSRF → NTLMv2 → gMSA → SeRestorePrivilege | ✓ |
| 18 | [Vault](./18-Vault/writeup.md) | PG Practice | Windows AD | Hard | SMB null → ntlm_theft → SeBackupPrivilege → SAM dump | ✓ |
| 19 | [Nagoya](./19-Nagoya/writeup.md) | PG Practice | Windows AD | Hard | Kerbrute → spray → GenericAll → Silver Ticket → MSSQL | ✓ |
| 20 | [Shenzi](./20-Shenzi/writeup.md) | PG Practice | Windows | Hard | SMB null → WordPress → AlwaysInstallElevated MSI | ✓ |
| 21 | [Squid](./21-Squid/writeup.md) | PG Practice | Windows | Hard | — | In Progress |
| 22 | [Craft2](./22-Craft2/writeup.md) | PG Practice | Windows AD | Hard | — | Pending |
| 23 | [Authby](./23-Authby/writeup.md) | PG Practice | Windows | Hard | — | Pending |
| 24 | [Hetemit](./24-Hetemit/writeup.md) | PG Practice | Linux | Hard | — | Pending |
| 25 | [Peppo](./25-Peppo/writeup.md) | PG Practice | Linux | Hard | — | Pending |
| 26 | [Nukem](./26-Nukem/writeup.md) | PG Practice | Linux | Hard | — | Pending |
| 27 | [Sorcerer](./27-Sorcerer/writeup.md) | PG Practice | Linux | Hard | — | Pending |
| 28 | [Pebbles](./28-Pebbles/writeup.md) | PG Practice | Linux | Intermediate | ZoneMinder SQLi | Lap 2 |
| 29 | [Algernon](./29-Algernon/writeup.md) | PG Practice | Windows | Intermediate | SmarterMail RCE | Lap 2 |
| 30 | [Kevin](./30-Kevin/writeup.md) | PG Practice | Windows | Easy | HP Power Manager | Lap 2 |

---

## Skills Demonstrated

| Category | Techniques |
|----------|-----------|
| **Web** | File upload bypass, RFI, SQLi, XPath injection, SSRF |
| **Active Directory** | AS-REP Roasting, Kerberoasting, Silver Ticket, RBCD, Pass-the-Ticket, ntlm_theft, gMSA abuse |
| **Windows PrivEsc** | GodPotato, AlwaysInstallElevated, SeBackupPrivilege, SeRestorePrivilege, SeManageVolumePrivilege, Scheduled Tasks |
| **Linux PrivEsc** | SUID, Cronjob abuse, MySQL UDF, memory dump |
| **Network** | SMB null sessions, FTP, WebDAV |
| **CVEs** | CVE-2020-11651, CVE-2020-7247, CVE-2020-10199, CVE-2021-22204 |

---

*All machines completed manually — no Metasploit on initial exploitation.*
