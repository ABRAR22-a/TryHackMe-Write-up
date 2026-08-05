# RootMe – TryHackMe Write-up

## Objective

The objective of this room is to gain initial access to the target web server through a vulnerable file upload feature and then escalate privileges to obtain root access.

---

# Methodology

## 1. Network Enumeration

I began by identifying the available services on the target using Nmap.

Example command:

```bash
nmap -sC -sV <TARGET_IP>
```

### Findings

* HTTP service
* SSH service

Since the challenge focused on web exploitation, I continued investigating the web application.

---

# 2. Web Enumeration

## View Source

Inspecting the page source revealed the username used for authentication.

**Username**

```text
R1ckRul3s
```

---

## robots.txt

Browsing to:

```text
http://<TARGET_IP>/robots.txt
```

revealed the password.

**Password**

```text
Wubbalubbadubdub
```

---

## Directory Enumeration

I used Gobuster to discover hidden files and directories.

```bash
gobuster dir -u http://<TARGET_IP>/ \
-w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt \
-x php
```

### Discovery

The scan revealed:

```text
panel.php
```

which provided access to the file upload functionality after authenticating with the discovered credentials.

---

# 3. Authentication

Using the credentials obtained during enumeration:

**Username**

```text
R1ckRul3s
```

**Password**

```text
Wubbalubbadubdub
```

I successfully logged in and reached the upload panel.

---

# 4. File Upload Bypass

Initially, I attempted to upload a Python reverse shell.

The upload was accepted, but the server displayed the Python source code instead of executing it. This indicated that Python files were not interpreted by the web server.

After further testing, I identified a PHP-compatible extension that was accepted by the upload filter while still being executed by the server.

I uploaded a PHP reverse shell using the `.phtml` extension.

After starting a Netcat listener on my attacking machine, I executed the uploaded file and successfully obtained a reverse shell.

---

# 5. Initial Access

After triggering the uploaded payload, I received a shell as the web server user.

Verification:

```bash
whoami
```

This confirmed successful code execution on the target system.

The first flag (`user.txt`) was then retrieved.

---

# 6. Privilege Escalation

To identify privilege escalation opportunities, I searched for SUID binaries.

```bash
find / -perm -4000 -type f 2>/dev/null
```

Among the results was:

```text
/usr/bin/python2.7
```

Python 2.7 was configured with the SUID bit, allowing execution with elevated privileges.

Using Python, I spawned a root shell.

Afterward, I verified the privilege escalation:

```bash
whoami
```

Output:

```text
root
```

Finally, I accessed the root user's directory and retrieved the second flag (`root.txt`).

---

# Skills Learned

* Network enumeration with Nmap
* Web directory enumeration using Gobuster
* HTML source code inspection
* Information disclosure through robots.txt
* Authentication using discovered credentials
* File upload vulnerability analysis
* Understanding the difference between accepted uploads and executable server-side files
* Upload filter bypass using an alternative PHP extension
* Reverse shell execution
* Linux privilege escalation
* SUID enumeration
* Exploiting SUID Python binaries
* Root shell verification

---

# Security Issues Identified

* Credentials exposed through publicly accessible resources.
* Sensitive administrative functionality accessible through hidden pages.
* Insecure file upload validation.
* Server execution of uploaded scripts.
* Dangerous SUID configuration on Python.
* Improper privilege separation.

---

# Key Takeaways

* Successful exploitation begins with thorough enumeration.
* Accepted file uploads are not necessarily executable; understanding how the web server processes file extensions is essential.
* Source code inspection and `robots.txt` can expose sensitive information.
* Misconfigured SUID binaries can provide a straightforward path to privilege escalation.
* Always verify elevated privileges after exploitation before proceeding to post-exploitation activities.

---

# Conclusion

This room demonstrated a complete web penetration testing workflow:

1. Network Enumeration
2. Web Enumeration
3. Information Disclosure
4. Authentication
5. File Upload Exploitation
6. Reverse Shell
7. Privilege Escalation
8. Flag Retrieval

The challenge provided practical experience with web exploitation, file upload bypass techniques, and Linux privilege escalation using SUID binaries.

