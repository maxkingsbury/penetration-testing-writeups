# Penetration Test Report — Shenron1

## 1. Executive Summary

A security assessment was performed against the target system **Shenron1** to evaluate its resilience against common web application and operating system-level attacks.

The assessment resulted in **full system compromise**, beginning with exploitation of a Joomla web application and culminating in **root-level access** via misconfigured sudo privileges.

The primary contributing factors to compromise were:

* Exposure of credentials in a hidden web-accessible file
* Insecure Joomla plugin upload functionality enabling remote code execution
* Storage of plaintext credentials in configuration files
* Weak privilege escalation controls allowing misuse of `apt` via sudo

**Final impact:** Complete compromise of system integrity, confidentiality, and availability.

---

## 2. Scope

**Target System:**

* Shenron1

**Services in scope:**

* Web application (Joomla CMS)
* SSH service
* Local operating system environment

No external systems were assessed.

---

## 3. Methodology

The following penetration testing methodology was used:

1. Network enumeration
2. Service identification and analysis
3. Web application assessment
4. Exploitation of identified vulnerabilities
5. Privilege escalation
6. Post-exploitation analysis

---

## 4. Technical Findings

### 4.1 Open Port and Service Enumeration

A full TCP port scan with service detection and default scripts was performed:

```bash
nmap -T4 -sV -A -p- TARGET_IP
```

This identified a web service running a Joomla CMS instance.

---

### 4.2 Credential Exposure in Hidden File

During manual enumeration of the web application, a hidden file was discovered containing administrative credentials:

```
admin:3iqtzi4RhkWANcu@$pa$$
```

**Risk:** High
**Impact:** Unauthorized administrative access to CMS

These credentials allowed direct authentication to the Joomla administrator panel.

---

### 4.3 Remote Code Execution via Joomla Plugin Upload

After authentication, a plugin upload feature was identified. The upload mechanism enforced structural requirements but failed to properly validate or sanitize uploaded content.

By bypassing these restrictions, a malicious plugin was uploaded, resulting in **remote code execution (RCE)** on the server.

**Vulnerability Class:**

* Insecure file upload leading to arbitrary code execution

**Risk:** Critical
**Impact:** Full server-level code execution under web service privileges

---

### 4.4 Shell Access and Environment Constraints

A reverse shell was initially attempted; however, the system’s netcat implementation did not support the `-e` flag.

A Python-based interactive shell was used instead, providing stable command execution for further enumeration.

---

### 4.5 Credential Discovery in Configuration Files

During post-exploitation enumeration, Joomla configuration files were found to contain plaintext credentials:

```
public $user = 'jenny';
public $password = 'Mypa$$wordi$notharD@123';
```

**Risk:** High
**Impact:** Credential reuse enabling lateral movement within system

---

## 5. Privilege Escalation

### 5.1 Access to Secondary User (SSH Key Injection)

Sudo privileges were enumerated:

```bash
sudo -l
```

Direct privilege escalation was not immediately available; however, SSH key-based authentication was leveraged to pivot to user **shenron**.

A key pair was generated and the public key was inserted into:

```
~/.ssh/authorized_keys
```

This enabled authenticated SSH access as the target user.

---

### 5.2 Privilege Escalation via Misconfigured APT Sudo Rule

Post-authentication enumeration revealed that the user **shenron** had permission to execute `apt` as root.

This was exploited using the `APT::Update::Pre-Invoke` option to execute a system shell during package update execution:

```bash
sudo /usr/bin/apt update -o APT::Update::Pre-Invoke::=/bin/sh
```

This resulted in immediate **root-level shell access**.

**Vulnerability Class:**

* Unsafe sudo configuration allowing command injection via package manager hooks

**Risk:** Critical
**Impact:** Full root compromise

---

## 6. Impact Assessment

Successful exploitation resulted in:

* Full administrative control of the Joomla CMS
* Execution of arbitrary system commands as web user
* Access to sensitive configuration data containing credentials
* Lateral movement between system users
* Complete root compromise of the operating system

This constitutes a **total system compromise**.

---

## 7. Flags (Proof of Compromise)

* User Flag:

```
098bf43cc909e1f89bb4c910bd31e1d4
```

* Root Flag:

```
aa087b2d466cd593622798c8e972bffb
```

---

## 8. Recommendations

### 8.1 Web Application Security

* Disable or restrict Joomla plugin upload functionality
* Implement strict file type validation and server-side content inspection
* Enforce least privilege for CMS administrative roles

### 8.2 Credential Management

* Remove all plaintext credentials from configuration files
* Implement secure secret storage mechanisms (e.g., vault solutions)
* Rotate all exposed credentials immediately

### 8.3 System Hardening

* Review and restrict sudo permissions, particularly for package managers such as `apt`
* Disable or tightly control `APT::Update::Pre-Invoke` functionality
* Implement least privilege access controls for all users

### 8.4 SSH Security

* Enforce key management policies
* Monitor and restrict unauthorized modifications to `authorized_keys`

---

## 9. Conclusion

The target system **Shenron1** was fully compromised due to a combination of insecure web application configuration, exposed credentials, and misconfigured privilege escalation pathways.

The attack chain demonstrates how low-complexity initial vulnerabilities can escalate into full system compromise when combined with weak operational security and privilege mismanagement.
