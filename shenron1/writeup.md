# Shenron1

## Overview

* Target: Shenron1
* Platform: VulnHub / CTF VM
* Focus: Web exploitation, credential discovery, lateral movement, sudo privilege escalation

---

## Enumeration

A full port scan with service detection was performed:

```bash
nmap -T4 -sV -A -p- TARGET_IP
```

The results identified a web server hosting a Joomla application, which became the primary attack surface.

---

## Initial Access

During manual enumeration of the web application, a hidden file was discovered containing credentials:

```
admin:PASSWORD
```

These credentials were valid for the Joomla administrator panel.

After logging in, the admin interface exposed a plugin upload feature. Although the upload mechanism enforced a specific structure, it was possible to bypass validation and upload a malicious plugin.

This resulted in remote code execution (RCE) on the target system.

---

## Shell Access

Following successful RCE, a reverse shell was attempted. However, the system’s netcat binary did not support the `-e` flag.

To bypass this limitation, a Python-based interactive shell was spawned instead, providing a more stable execution environment for further enumeration.

---

## Credential Discovery

While examining Joomla configuration files on the system, additional credentials were identified:

```
public $user = 'jenny';
public $password = 'PASSWORD';
```

These credentials were useful for further lateral movement within the system.

---

## Privilege Escalation (User → Shenron)

Sudo privileges were enumerated using:

```bash
sudo -l
```

Initial access to the `shenron` user was restricted. To pivot, SSH key authentication was used:

* Generated an SSH key pair with `ssh-keygen`
* Added the public key to `~/.ssh/authorized_keys` for the `shenron` user
* Gained SSH access as `shenron`

---

## Privilege Escalation (Shenron → Root)

Once logged in as `shenron`, sudo privileges were re-evaluated. The user was allowed to execute `apt` as root.

This misconfiguration was exploited using the `APT::Update::Pre-Invoke` option to execute a shell during the update process:

```bash
sudo /usr/bin/apt update -o APT::Update::Pre-Invoke::=/bin/sh
```

This resulted in an immediate root shell.

---

## Flags

User flag:

```
098bf43cc909e1f89bb4c910bd31e1d4
```

Root flag:

```
aa087b2d466cd593622798c8e972bffb
```

---

## Takeaways

* Joomla plugin upload features are a frequent RCE vector when improperly secured
* Configuration files often expose reusable credentials that enable lateral movement
* Python shells are a reliable fallback when netcat reverse shells are limited
* SSH key injection remains a strong method for privilege pivoting when write access is available
* Misconfigured sudo permissions on package managers such as `apt` can directly lead to root via pre-execution hooks
