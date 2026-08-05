# Pickle Rick – TryHackMe Write-up

## Objective

Complete the Pickle Rick challenge by finding the three secret ingredients through web enumeration, command execution, and privilege escalation.

---

# Methodology

## 1. Initial Enumeration

I started by scanning the target machine to identify open ports and running services.

### Findings

* HTTP service
* SSH service

Since the challenge focuses on web exploitation, I continued investigating the HTTP service.

---

## 2. Web Enumeration

### View Source

Inspecting the page source revealed a username.

**Discovered Username**

```
R1ckRul3s
```

---

### robots.txt

Checking the `robots.txt` file revealed a password.

This demonstrated that sensitive information should never be stored inside publicly accessible files.

---

### Directory Enumeration

I used Gobuster to discover hidden directories and files.

Example command:

```bash
gobuster dir -u http://<TARGET_IP>/ \
-w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
-x php
```

Important discovery:

```
/login.php
```

Adding the `.php` extension was necessary to discover the login page.

---

## 3. Authentication

Using the discovered username and password, I successfully logged into the web portal.

The application provided a command execution interface (Web Shell).

---

## 4. Command Execution

Basic enumeration commands were used to understand the environment.

Examples:

```bash
pwd
whoami
ls
```

Attempting to read files with:

```bash
cat
```

was blocked by the application.

Instead, alternative commands were used to access file contents.

---

## 5. Finding the Ingredients

### Ingredient 1

Located inside the web directory.

---

### Ingredient 2

Located in Rick's home directory.

Because the filename contained spaces, quotation marks were required when referencing it.

Example:

```bash
"/home/rick/second ingredients"
```

---

## 6. Privilege Escalation

To check sudo permissions:

```bash
sudo -l
```

Output:

```
(ALL) NOPASSWD: ALL
```

This indicated that the current user could execute any command as root without entering a password.

Using sudo, I accessed the root directory and obtained the final ingredient.

Example:

```bash
sudo ls /root
```

---

# Skills Learned

* Web enumeration
* Inspecting HTML source code
* Enumerating robots.txt
* Directory brute-forcing with Gobuster
* Importance of file extensions during enumeration
* Linux command execution
* Working around restricted commands
* Understanding Linux file permissions
* Checking sudo privileges with `sudo -l`
* Basic privilege escalation using misconfigured sudo permissions

---

# Security Issues Identified

* Credentials exposed in publicly accessible resources.
* Sensitive functionality accessible through a web shell.
* Overly permissive sudo configuration (`NOPASSWD: ALL`).
* Poor privilege separation.

---

# Key Takeaways

* Always enumerate thoroughly before attempting exploitation.
* Hidden files and directories often reveal sensitive information.
* When a command is blocked, alternative Linux commands may achieve the same objective.
* Misconfigured sudo permissions are a common privilege escalation vector.
* Enumeration is often the most important phase of a penetration test.

---

# Conclusion

This room introduced the basic penetration testing workflow:

1. Enumeration
2. Information Gathering
3. Authentication
4. Command Execution
5. Privilege Escalation
6. Flag Collection

It provided hands-on practice with common web application enumeration techniques and basic Linux privilege escalation concepts.

