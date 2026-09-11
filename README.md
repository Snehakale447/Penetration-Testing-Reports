# Penetration Testing Portfolio

This repository contains my hands-on cybersecurity reports and walkthroughs for various target machines. It tracks my practical experience in vulnerability assessment, exploitation, and privilege escalation.

---
## Target Machine: Shenron-1

### Project Setup
* **Target Machine:** Shenron-1 (A vulnerable Linux VM designed for practice)
* **Target IP:** `192.168.56.105`
* **My Setup:** Kali Linux
* **Full Technical Report:** [Download PDF Report](./SHENRON1%20Network%20Pentration%20Testing%20Report\(1\).pdf)
* **Core Tools Used:** Nmap, Dirb, Python, and native Linux command-line utilities.

---

### How I Compromised the System (Step-by-Step)

1. **Finding the Target (Scanning)**
   I started with an Nmap network scan and found two open doors: Port 22 (SSH) and Port 80 (HTTP web server). Next, I used a tool called dirb to look for hidden folders on the website. It found three interesting paths: 
   * `/joomla`
   * `/test`
   * `/joomla/administrator`

2. **Getting Inside (Initial Access)**
   I looked at the website's source code and noticed the developer had left the admin username and password right inside the HTML comments:
   * **Username:** `admin`
   * **Password:** `3igtzi4RhkWANcu@pass$$`
   
   I used these credentials to log directly into the Joomla admin dashboard. Once inside, I edited one of the website's design templates (`index.php`) and pasted a PHP reverse shell script into it. When I visited that page in my browser, the server ran my script and handed me my first low-privilege command line as the `www-data` user.

3. **Moving Across the System (Lateral Movement)**
   While looking around the web server files, I checked the main Joomla configuration file (`configuration.php`) and found a plaintext database password. 
   
   Because the basic web shell couldn't handle password prompts, I used a quick Python trick (`pty.spawn`) to upgrade it into a real interactive terminal. I then tried that database password against a local user named **jenny**, and it worked, letting me log into her account.

4. **Climbing Higher (Privilege Escalation to User: shenron)**
   I checked Jenny's account privileges by running `sudo -l`. I discovered she was allowed to use the system copy command (`/usr/bin/cp`) as another user named **shenron** without needing a password. 
   
   I used this copy privilege to duplicate a protected file (`/var/opt/passwd.txt`) over to a temporary folder where I could read it. Inside, I found the plaintext password for **shenron** (`YouKNowMyPaSsWoRdToStRoNgDeAr`) and logged into that account.

5. **Taking Total Control (Root Takeover)**
   Once I was logged in as `shenron`, I checked my privileges again. The system allowed this account to run the Ubuntu package manager (`/usr/bin/apt`) as the absolute administrator (root) without a password. 
   
   I exploited a well-known feature in `apt` that lets you run a custom command right before it checks for updates. By forcing it to run a system shell (`/bin/sh`), the application dropped me straight into a permanent **root shell**. From there, I went into the administrator's private folder and grabbed the final text flag (`root.txt`) to complete the challenge.
