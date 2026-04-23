# Penetration Test Report — Inferno 1.1

## 1. Executive Summary

An assessment was performed against the target system **Inferno 1.1** to evaluate exposure across web services, authentication mechanisms, and local privilege boundaries.

The system was fully compromised through:

* Weak web authentication bypassed via brute force
* Exploitation of a vulnerable web-based IDE (Codiad)
* Credential extraction from encoded application artifacts
* Misconfigured sudo permissions allowing privilege escalation

Final impact: **complete root-level system compromise**

---

## 2. Scope

**Target System:**

* Inferno 1.1 (192.168.56.125)

**In-Scope Components:**

* Web application stack
* Authentication interfaces
* Remote administration tooling (Codiad)
* Local Linux system environment

---

## 3. Methodology

The following methodology was applied:

1. Network enumeration and service discovery
2. Web application discovery and endpoint enumeration
3. Authentication testing and brute force attack
4. Exploitation of known vulnerable web application (Codiad)
5. Post-exploitation file system analysis
6. Credential extraction and SSH access
7. Privilege escalation via sudo misconfiguration

---

## 4. Technical Findings

---

## 4.1 Network Enumeration

A full TCP port scan with service detection was performed:

```bash id="p8m1qv"
nmap -T4 -sV -A -p- TARGET_IP
```

The scan identified a web service hosting an application with hidden or non-obvious endpoints.

---

## 5. Initial Access

---

### 5.1 Hidden Web Directory Discovery

During manual enumeration, a hidden application directory was identified:

```
/inferno
```

This endpoint contained a JavaScript-based login interface.

---

### 5.2 Authentication Brute Force

The login form was tested using Hydra with a known default username:

```
admin
```

A successful authentication was achieved with the password:

```
dante1
```

This granted access to a backend web interface running **Codiad**, a web-based IDE.

---

### 5.3 Codiad Exploitation (RCE)

The target application version was identified as **Codiad**, which is known to contain remote code execution vulnerabilities.

A public exploit (**CVE-2018-14009**) was used:

```bash id="x7k2ld"
49705.py
```

This resulted in a web-based shell, providing initial code execution capability on the server.

---

### 5.4 Credential Extraction from Application Artifacts

During post-exploitation enumeration, a file was identified:

```
.downloads.dat
```

This file contained encoded data in hexadecimal format.

After decoding (e.g., via CyberChef), embedded credentials were recovered:

```
dante:V1rg1l10h3lpm3
```

These credentials were used for SSH authentication, granting a stable system shell as user **dante**.

---

## 6. Privilege Escalation

---

### 6.1 Sudo Privilege Enumeration

Sudo permissions were checked:

```bash id="z2m8fa"
sudo -l
```

The user **dante** was allowed to execute `tee` as root.

This misconfiguration allowed modification of privileged system files.

---

### 6.2 Privilege Escalation via /etc/sudoers Modification

A malicious entry was appended to the sudoers file using `tee`:

```bash id="k4v9ps"
echo "dante ALL=(ALL) NOPASSWD:ALL" | sudo tee -a "/etc/sudoers"
```

After modifying the sudoers configuration, full root privileges were obtained:

```bash id="t1x6rq"
sudo su
```

Root access was confirmed successfully.

---

## 7. Flags (Proof of Compromise)

* User Flag:

```
77f6f3c544ec0811e2d1243e2e0d1835
```

* Root Flag:

```
77f6f3c544ec0811e2d1243e2e0d1835
```

---

## 8. Impact Assessment

The system was fully compromised due to:

* Weak authentication on exposed web interface
* Use of default credentials
* Vulnerable third-party application (Codiad) allowing RCE
* Poor secrets handling in application-generated files
* Dangerous sudo permissions allowing modification of privileged system configuration

This resulted in full compromise of system integrity and administrative control.

---

## 9. Recommendations

### 9.1 Authentication Security

* Eliminate default credentials from all services
* Enforce strong password policies
* Implement rate limiting and account lockout mechanisms

### 9.2 Web Application Security

* Remove or restrict vulnerable Codiad installation
* Apply security patches for known CVEs (e.g., CVE-2018-14009)
* Restrict access to administrative interfaces

### 9.3 Secrets Management

* Avoid storing credentials in plaintext or reversible encoded formats
* Encrypt sensitive application artifacts or disable insecure logging/export features

### 9.4 Privilege Escalation Prevention

* Audit sudo permissions and remove unnecessary access to `tee`
* Prevent direct or indirect modification of `/etc/sudoers`
* Enforce least privilege for all system users

---

## 10. Conclusion

Inferno 1.1 was fully compromised through a chain of weak authentication, vulnerable web application exploitation, and unsafe privilege delegation.

The attack highlights the compounded risk of:

* default credentials
* unpatched web applications
* overly permissive sudo configurations

Together, these issues enabled complete root-level system compromise.
