# Penetration Test Report — Momentum

## 1. Executive Summary

An assessment was conducted against the target system as part of a controlled security exercise. The objective was to identify exploitable vulnerabilities leading to unauthorized access and privilege escalation.

The system was fully compromised through:

* Client-side cookie manipulation and credential recovery
* SSH access via recovered credentials
* Local privilege escalation via exposed Redis interface

The final outcome was **complete root-level system compromise**.

---

## 2. Scope

**Target System:**

* Momentum

**In-Scope Components:**

* Web application (PHP/JavaScript frontend)
* Authentication mechanism (cookie/session handling)
* SSH service
* Local services and processes

---

## 3. Methodology

The following methodology was used during the engagement:

1. Web application enumeration
2. Static and client-side code analysis
3. Session handling and cookie inspection
4. Credential extraction and SSH access
5. Process enumeration
6. Privilege escalation via internal services

---

## 4. Technical Findings

---

## 4.1 Web Application Enumeration

During manual enumeration of the application, the following resources were identified:

* `/js/main.js`
* `opus-details.php`

These files were reviewed for client-side logic and potential secrets handling.

A cookie was observed being set by the application, indicating possible client-side authentication or session encoding logic.

<img width="769" height="112" alt="image" src="https://github.com/user-attachments/assets/64b4f62c-35f0-4d20-9809-023e2c9aaae1" />

---

## 5. Initial Access

### 5.1 Cookie Decryption and Credential Recovery

The application was found to use a client-side encoded cookie mechanism.

Using Node.js-based analysis and decoding of the cookie structure, the following credentials were recovered:

```id="v8k2lm"
password: auxerre-alienum##
```

Based on contextual analysis of the application and naming conventions, the likely username was inferred as:

```id="q3n7zp"
auxerre
```

---

### 5.2 SSH Access

The recovered credentials were used to authenticate via SSH:

```bash id="x9m2aa"
ssh auxerre@TARGET_IP
```

Access was successfully obtained to the system as a low-privileged user.

---

## 6. Privilege Escalation

---

### 6.1 Process Enumeration

Standard enumeration tools (including linPEAS) did not immediately reveal privilege escalation vectors.

Manual process inspection was performed:

```bash id="r2k9fd"
ps aux
```

This revealed an unusual process not typically expected in standard configurations:

* `redis-cli` exposed in a local context

---

### 6.2 Redis Service Discovery

Upon executing `redis-cli`, a local Redis interface was accessed.

This indicated that Redis was running on the local system and accessible from the compromised user context.

Further exploration of Redis keyspace was performed.

---

### 6.3 Sensitive Data Exposure in Redis

Using Redis enumeration commands:

```bash id="t6z1pq"
KEYS *
```

A sensitive key was identified:

```
rootpass
```

The value was retrieved using:

```bash id="k7v4bn"
GET rootpass
```

This returned the root password.

<img width="247" height="88" alt="image" src="https://github.com/user-attachments/assets/059636e8-8458-47f2-b40a-e2d101c7980c" />


---

### 6.4 Root Access

The retrieved credentials were used to escalate privileges to root, resulting in full administrative access to the system.

---

## 7. Flags (Proof of Compromise)

* User Flag:

```id="f1q8st"
84157165c30ad34d18945b647ec7f647
```

* Root Flag:

```id="l9v3re"
658ff660fdac0b079ea78238e5996e40
```

---

## 8. Impact Assessment

The system was fully compromised through a combination of:

* Insecure client-side session handling
* Weak credential derivation from application logic
* Lack of proper secrets management in internal services
* Exposed Redis instance containing sensitive authentication data

This resulted in full compromise of system integrity, including root-level access.

---

## 9. Recommendations

### 9.1 Web Application Security

* Do not rely on client-side cookie encoding for authentication
* Implement secure session management using server-side validation
* Avoid storing sensitive logic or credentials in frontend code

### 9.2 SSH Security

* Enforce strong authentication policies
* Disable password-based authentication where possible
* Monitor for unusual login patterns

### 9.3 Redis Hardening

* Restrict Redis to localhost with authentication enabled
* Avoid storing plaintext secrets (e.g., root passwords) in key-value stores
* Implement access control lists (ACLs) for Redis operations
* Disable or restrict `KEYS *` in production environments

### 9.4 System Hardening

* Audit running processes for unintended services
* Remove unnecessary internal services from production environments
* Implement secrets management (vault-based solutions)

---

## 10. Conclusion

The Momentum system was fully compromised due to insecure client-side authentication design combined with exposed internal services lacking proper access control.

The attack demonstrates how weak session handling and improperly secured backend services (such as Redis) can lead to complete system compromise when combined.
