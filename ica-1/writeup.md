# Penetration Test Report — ICA-1

## 1. Executive Summary

A security assessment was conducted against the target system **ICA-1** to evaluate exposure across database services, authentication mechanisms, and privilege boundaries.

The system was fully compromised through:

* Exposure of database credentials in application configuration files
* Credential harvesting from database tables
* Weak SSH authentication combined with reused credentials
* Privilege escalation via SUID binary exploiting insecure PATH resolution

Final impact: **complete root-level system compromise**

---

## 2. Scope

**Target System:**

* ICA-1

**In-Scope Components:**

* Web application
* Database service (MySQL)
* SSH service
* Local Linux environment

---

## 3. Methodology

The following methodology was used:

1. Network enumeration and service discovery
2. Web application analysis
3. Credential extraction from configuration files
4. Database access and data enumeration
5. SSH credential brute force testing
6. Privilege escalation via SUID binary analysis
7. PATH hijacking exploitation

---

## 4. Technical Findings

---

## 4.1 Network Enumeration

A full TCP port scan with service detection was performed:

```bash id="m2v8xq"
nmap -T4 -sV -A -p- TARGET_IP
```

The scan identified:

* Web service exposing configuration files
* MySQL database service
* SSH service

---

## 5. Initial Access

---

### 5.1 Exposure of Database Credentials

During web application enumeration, a configuration file was discovered:

```id="p7k3lm"
databases.yml
```

This file contained plaintext database credentials.

---

### 5.2 Database Access and Credential Extraction

Using the discovered credentials, access to the MySQL database was obtained:

```bash id="t9q2sd"
mysql -u <user> -p
```

Within the **staff database**, records containing usernames and passwords for multiple staff members were identified.

---

### 5.3 Credential Processing and SSH Brute Force

The extracted passwords were encoded in Base64 and subsequently decoded.

These credentials were then used in a brute-force attack against SSH using Hydra.

The following users were successfully authenticated:

* travis
* dexter

---

## 6. User Access

---

### 6.1 Initial User — travis

Login as `travis` was successful; however, no immediately exploitable artifacts or misconfigurations were identified in this account.

---

### 6.2 Primary User — dexter

Login as `dexter` revealed further enumeration hints indicating the presence of misconfigured privileged binaries.

---

## 7. Privilege Escalation

---

### 7.1 SUID Binary Discovery

Enumeration of SUID binaries revealed a custom executable:

```id="v5k9np"
get_access
```

Execution of the binary initially appeared benign; however, static analysis revealed a critical insecure implementation.

---

### 7.2 Vulnerable Code Path

A sensitive system call was identified within the binary:

```id="x3r8qt"
cat /root/system.info
```

This indicated reliance on the system `cat` binary without specifying an absolute path, introducing a **PATH hijacking vulnerability**.

---

### 7.3 PATH Hijacking Exploitation

To exploit this misconfiguration:

1. A malicious `cat` binary was created in `/tmp`
2. The `PATH` environment variable was modified to prioritize `/tmp`

```bash id="k8m1zd"
export PATH=/tmp:$PATH
```

When `get_access` executed `cat`, it resolved to the malicious version, enabling execution of arbitrary commands as root.

---

### 7.4 Root Access

The exploit resulted in execution with elevated privileges, leading to full root shell access.

---

## 8. Flags (Proof of Compromise)

* User Flag:

```id="c1v9pa"
ICA{Secret_Project}
```

* Root Flag:

```id="r6x2me"
ICA{Next_Generation_Self_Renewable_Genetics}
```

---

## 9. Impact Assessment

The system was fully compromised due to:

* Exposure of database credentials in configuration files
* Unrestricted database access exposing user credentials
* Weak password handling and reuse across services
* Custom SUID binary vulnerable to PATH hijacking
* Lack of absolute path usage in privileged operations

This resulted in full system compromise with root-level access.

---

## 10. Recommendations

### 10.1 Configuration Security

* Remove plaintext credentials from configuration files
* Restrict access to sensitive configuration files

### 10.2 Database Security

* Enforce strict database access controls
* Hash and salt all stored credentials
* Avoid storing reusable plaintext passwords

### 10.3 Authentication Hardening

* Implement account lockout and rate limiting for SSH
* Prevent password reuse across services

### 10.4 Privilege Escalation Prevention

* Avoid use of SUID binaries for custom executables
* Use absolute paths for all system binaries in privileged programs
* Sanitize and control environment variables (especially PATH)
* Apply principle of least privilege

---

## 11. Conclusion

ICA-1 was fully compromised through a chain of configuration weaknesses, credential exposure, and insecure privileged binary design.

The final compromise was achieved through PATH hijacking of a SUID binary, demonstrating how unsafe coding practices in privileged contexts can directly lead to root-level access.
