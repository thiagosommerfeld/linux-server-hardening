# linux-server-hardening

# 🛡️ Linux Server Hardening & Security Baseline

## 📋 About the Project
This project documents the hardening process of a newly deployed cloud-based Linux server (Debian/Ubuntu). The goal is to drastically reduce the server's attack surface, protecting it against automated attacks, brute force, and malicious network scanning.

## 🛠️ Technologies and Tools Used
* **Operating System:** Linux (Debian 10/Ubuntu)
* **Identity and Access Management (IAM):** User creation and privilege delegation via `sudo`
* **Cryptography:** SSH Public Key Authentication (RSA 4096-bit)
* **Network Security:** UFW (Uncomplicated Firewall)
* **Patch Automation:** `unattended-upgrades`

## 🚀 Implementation Steps (Security Controls)

### 1. Critical Updates Automation (Patch Management)
Outdated servers are the main gateway for intrusions. I configured security patch automation to ensure the system applies critical fixes without manual intervention.
* **Installed package:** `unattended-upgrades`
* **Configuration:** `dpkg-reconfigure --priority=low unattended-upgrades`

### 2. User and Privilege Management (IAM)
Using the `root` user is a severe security flaw.
* Created a standard non-privileged user (`adduser`).
* Added the new user to the `sudo` group (`usermod -aG sudo`), applying the Principle of Least Privilege. Every critical command now requires explicit authentication.

### 3. Cryptographic Key-Based Authentication
Password-based authentication was completely replaced by cryptographic keys (RSA 4096-bit), preventing brute force and dictionary attacks.
* Key pair generation: `ssh-keygen -b 4096`
* Uploaded the public key to the server via SCP protocol (`~/.ssh/authorized_keys`).

### 4. SSH Service Hardening (`sshd_config`)
The SSH service is the number one target for malicious scripts. I edited the `/etc/ssh/sshd_config` file, applying the following controls:
* **Changed Default Port:** Moved from port 22 to a high custom port (e.g., 717) to evade automated scanners.
* **Disabled Root Login:** `PermitRootLogin no` (Blocks direct login attempts to the administrator account).
* **Disabled Passwords:** `PasswordAuthentication no` (Forces the exclusive use of the cryptographic keys created in step 3).
* **IP Restriction:** Configured to accept only IPv4 traffic (`AddressFamily inet`).

### 5. Firewall Configuration and Server Cloaking (UFW & ICMP Block)
To protect the server's perimeter, I installed and configured **UFW**.
* **Default Deny:** All incoming traffic is blocked by default, except explicitly allowed ports.
* **Restricted Access:** Allowed only the custom SSH port (`ufw allow 717`) and necessary service ports (e.g., port 80 for HTTP).
* **Ping (ICMP) Blocking:** To cloak the server from external network scans, I edited the `/etc/ufw/before.rules` file to drop "ping" packets.

## 📈 Results
Following the implementation, the server became resilient to common network scanners (e.g., Nmap), immune to SSH password cracking (Brute Force/Dictionary attacks), and now remediates security flaws automatically.
