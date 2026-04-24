# Penetration Test Report — Momentum 2

## 1. Executive Summary

A security assessment was conducted against the target system **Momentum 2** to evaluate exposure across web application functionality, authentication mechanisms, and privilege boundaries.

The system was fully compromised through:

* Insecure authentication logic relying on manipulable cookies
* Arbitrary file upload leading to remote code execution
* Exposure of plaintext credentials in user directories
* Command injection in a privileged Python script executed via sudo

Final impact: **complete root-level system compromise**

---

## 2. Scope

**Target System:**

* Momentum 2 (Target IP)

**In-Scope Components:**

* Web application (HTTP service)
* Authentication/session handling
* SSH service
* Local Linux environment

---

## 3. Methodology

The following methodology was applied:

1. Network enumeration and service discovery
2. Web directory and endpoint enumeration
3. Authentication and session analysis
4. Exploitation of file upload functionality
5. Post-exploitation enumeration
6. Credential harvesting and lateral movement
7. Privilege escalation via command injection

---

## 4. Technical Findings

---

## 4.1 Network Enumeration

A service and version scan was performed:

```bash id="k8x2fd"
nmap -sV -A <target-ip>
```

### Findings:

* Port 80: HTTP (web application)
* Port 22: SSH

The presence of a web application indicated a likely initial attack vector.

---

## 5. Initial Access

---

### 5.1 Web Enumeration

Directory brute forcing was conducted using Gobuster:

```bash id="q3m9zp"
gobuster dir -u http://<target-ip>/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,log,php,html,bak,zip
```

### Discovered Endpoints:

* `/dashboard.html`
* `/ajax.php`
* `/ajax.php.bak`
* `/owls`
* `/manual`
* Supporting asset directories (`/js`, `/css`, `/img`)

The endpoint `/dashboard.html` exposed a file upload feature.

---

### 5.2 Authentication Bypass via Cookie Manipulation

Analysis of the backup file `/ajax.php.bak` revealed:

* Administrative functionality depended on a client-controlled cookie
* The cookie required an additional uppercase character suffix
* A POST parameter (`secure=val1d`) was required for successful requests

Using Burp Suite:

* Intercepted the upload request
* Added `secure=val1d` parameter
* Modified the admin cookie accordingly

The correct cookie suffix was identified via brute force using Burp Intruder.

---

### 5.3 Remote Code Execution via File Upload

After successful authentication bypass:

* A PHP reverse shell payload was uploaded via `/dashboard.html`
* The file was placed in the `/owls` directory

A listener was configured:

```bash id="z7n4ya"
nc -lvnp <attacker-port>
```

Upon accessing the uploaded payload, a shell was obtained.

**Access Level:** `www-data`

---

## 6. Privilege Escalation

---

### 6.1 Horizontal Escalation (www-data → athena)

During enumeration of user directories:

```id="r1t8vs"
/home/athena/password-reminder.txt
```

This file contained plaintext credentials for user **athena**.

The credentials were used to switch users:

```bash id="m5k2xa"
su athena
```

---

### 6.2 Vertical Escalation (athena → root)

Sudo privileges were enumerated:

```bash id="y2p9dl"
sudo -l
```

The user **athena** was permitted to execute the following script as root:

```id="v8q3kn"
/home/team-tasks/cookie-gen.py
```

---

### 6.3 Command Injection in cookie-gen.py

The script was vulnerable to command injection due to unsafe handling of user input.

This vulnerability was exploited by injecting a reverse shell payload during execution.

Listener setup:

```bash id="w6f3zc"
nc -lvnp <attacker-port>
```

Execution:

```bash id="u3x8br"
sudo /home/team-tasks/cookie-gen.py
```

A root shell was successfully obtained.

---

## 7. Impact Assessment

The system was fully compromised due to:

* Insecure client-side authentication relying on manipulable cookies
* Improper access control on administrative functionality
* Arbitrary file upload enabling remote code execution
* Storage of plaintext credentials in user directories
* Privileged script vulnerable to command injection

This resulted in full compromise of system confidentiality, integrity, and availability.

---

## 8. Recommendations

### 8.1 Authentication & Session Management

* Do not rely on client-side cookies for authorization decisions
* Implement server-side session validation
* Enforce multi-factor authentication for administrative functions

### 8.2 Web Application Security

* Restrict file upload functionality and validate file types strictly
* Remove or secure backup files such as `.bak`
* Sanitize all user inputs to prevent injection vulnerabilities

### 8.3 Credential Management

* Never store plaintext credentials in user-accessible files
* Enforce strong password policies
* Apply strict file permissions on sensitive data

### 8.4 Privilege Escalation Mitigation

* Audit and restrict sudo permissions
* Validate and sanitize all inputs in privileged scripts
* Avoid executing user-influenced data in root-level contexts

---

## 9. Conclusion

Momentum 2 was fully compromised through a chained attack involving weak authentication controls, insecure file upload functionality, exposed credentials, and a command injection vulnerability in a privileged script.

The attack demonstrates how multiple moderate vulnerabilities can combine into a critical system compromise when proper security controls are not enforced.
