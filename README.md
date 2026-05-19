# 🛡️ Linux Server Hardening

## 📋 About the Project
This project documents the hardening process of a newly deployed AWS cloud-based Linux server Ubuntu. The goal is to drastically reduce the server's attack surface, protecting it against automated attacks, brute force, and malicious network scanning.

## 🛠️ Technologies and Tools Used
* **Operating System:** Linux Ubuntu
* **Identity and Access Management (IAM):** User creation and privilege delegation via `sudo`
* **Cryptography:** SSH Public Key Authentication (RSA 4096-bit)
* **Network Security:** UFW (Uncomplicated Firewall)
* **Patch Automation:** `unattended-upgrades`

## 🚀 Implementation Steps (Security Controls)

### 1. Critical Updates Automation (Patch Management)
OOutdated servers are the main gateway for intrusions. I performed a manual update and configured security patch automation to ensure the system applies critical fixes without manual intervention.
* **Manual Update:** `sudo apt update && sudo apt upgrade -y`
* **Installed package:** `apt install unattended-upgrades -y`
* **Configuration:** `dpkg-reconfigure --priority=low unattended-upgrades`

### 2. User and Privilege Management (IAM)
Using the `root` user is a severe security flaw. I created a standard non-privileged user and granted explicit administrative rights via the sudo group, applying the Principle of Least Privilege.
* Created a standard non-privileged user (`adduser`).
* Added the new user to the `sudo` group (`usermod -aG sudo`). Every critical command now requires explicit authentication.

### 3. Cryptographic Key-Based Authentication
Password-based authentication was completely replaced by cryptographic keys (RSA 4096-bit), preventing brute force and dictionary attacks.
* Key pair generation: `ssh-keygen -t rsa -b 4096`
* Uploaded the public key to the server via SCP protocol (`ssh-copy-id -i ~/.ssh/id_rsa.pub secadmin@<server_ip>`).

### 4. SSH Service Hardening (`sshd_config`)
The SSH service is the number one target for malicious scripts. I edited the `/etc/ssh/sshd_config` file, applying the following controls:
* **Changed Default Port:** Moved from port 22 to a high custom port (e.g., 717) to evade automated scanners.
* **Disabled Root Login:** `PermitRootLogin no` (Blocks direct login attempts to the administrator account).
* **Disabled Passwords:** `PasswordAuthentication no` (Forces the exclusive use of the cryptographic keys created in step 3).
* **IP Restriction:** Configured to accept only IPv4 traffic (`AddressFamily inet`).
  
  ⚠️ Troubleshooting & Modern Ubuntu Quirks
On newer Ubuntu versions, simply changing the sshd_config file and restarting the service might not change the port due to Systemd sockets and Cloud-Init overrides. To enforce the custom port and disable passwords, the following steps were taken:

1. Disable Systemd SSH Socket:

`sudo systemctl disable --now ssh.socket
sudo systemctl daemon-reload`

2. Check for Cloud-Init Overrides:
If port 22 is still active, edit the cloud overrides file (if it exists): `sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf`

Action: Ensure `PasswordAuthentication no` is set and remove any `Port 22` entries.

3. Restart the SSH Service: `sudo systemctl restart ssh`

### 5. Firewall Configuration and Server Cloaking (UFW & ICMP Block)
To protect the server's perimeter, I installed and configured UFW (Uncomplicated Firewall) with a default-deny policy.
* **Default Deny:** All incoming traffic is blocked by default, except explicitly allowed ports.
* **Restricted Access:** Allowed only the custom SSH port (`ufw allow 717`) and necessary service ports (e.g., port 80 for HTTP).
* **Ping (ICMP) Blocking:** To cloak the server from external network scans, I edited the `/etc/ufw/before.rules` file to drop "ping" packets.
* Action: Under the `# ok icmp codes for INPUT` section, I changed the `ACCEPT` rules to `DROP` for `echo-request`.

### 6. Cloud Provider Perimeter Security (AWS Security Groups)
Even with the OS firewall active, the cloud perimeter must be secured. In the AWS Management Console:

Navigated to the instance's Security Group.

Deleted the default rule allowing SSH on port 22 (`0.0.0.0/0`).

Added a new Custom TCP rule allowing incoming traffic on port `717`.

## 📈 Results
Following the implementation, the server became resilient to common network scanners (e.g., Nmap), immune to SSH password cracking (Brute Force/Dictionary attacks), and now remediates security flaws automatically. The server no longer responds to ICMP ping requests, reducing its visibility to automated botnets
