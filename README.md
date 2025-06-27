# 🛡️ Local Vulnerability Assessment using OpenVAS on Kali Linux VM

## 🎯 Objective
In this project, I performed a full vulnerability assessment on my local Kali Linux machine using **OpenVAS / Greenbone Vulnerability Management (GVM)**. I documented every step, issue faced, and how I resolved it.

---

## 🧰 Tools Used
- Kali Linux (VM on Windows)
- Greenbone Vulnerability Management (GVM)
- Firefox browser for accessing GSA web interface

---

## 🛠️ Step-by-Step Setup

### Step 1: Install GVM
```bash
sudo apt update && sudo apt install -y gvm
✅ Step 2: System Clean and Full Upgrade
bash
Copy code
sudo apt clean
sudo apt update
sudo apt full-upgrade -y
⚠️ I faced a GPG key error related to a missing public key.
🔧 I fixed it with:

bash
Copy code
sudo gpg --recv-keys 827C8569F2518CC677FECA1AED65462EC8D5E4C5
sudo apt update
✅ Step 3: PostgreSQL Version Mismatch
GVM expected PostgreSQL 17, but my system had version 16 by default.

🔍 Checked clusters:

bash
Copy code
pg_lsclusters
⚙️ I upgraded using:

bash
Copy code
sudo pg_upgradecluster 16 main
sudo systemctl restart postgresql
Now version 17 runs on port 5433, which is what libgvmd needs.

✅ Step 4: GVM Setup
bash
Copy code
sudo gvm-setup
⚠️ I faced feed download timeouts (SCAP/CERT) during this step.

🔄 Re-ran:

bash
Copy code
sudo greenbone-feed-sync --type scap
sudo greenbone-feed-sync --type cert
✅ Step 5: Check Configuration
bash
Copy code
sudo gvm-check-setup
🧪 This checks if all components (scanners, feeds, certificates, PostgreSQL, services) are set up properly.

✅ Step 6: Fixing "No Users" or "Login Failed" Errors
Initially, I wasn’t able to log in and user list showed empty.

🔄 So I created a new user and password:

bash
Copy code
sudo gvmd --create-user=admin
sudo gvmd --user=admin --new-password='gvm@123'
🧾 Verified user:

bash
Copy code
sudo runuser -u _gvm -- gvmd --get-users
If users still didn’t appear, I restarted and rebuilt:

bash
Copy code
sudo gvm-stop
sudo runuser -u _gvm -- gvmd --rebuild
sudo gvm-start
✅ Step 7: Start and Check All Services
bash
Copy code
sudo gvm-start
Verified:

bash
Copy code
sudo systemctl status gvmd
sudo systemctl status gsad
sudo systemctl status ospd-openvas
✅ Step 8: Access Web Interface
Opened Firefox and navigated to:

cpp
Copy code
https://127.0.0.1:9392
⚠️ Faced SSL Warning — I clicked Advanced > Accept the risk.

🔐 Login:

makefile
Copy code
Username: admin
Password: gvm@123
⚠️ Error: No Scan Configs Available
After logging in, I got this error:

❌ “No Scan Configs available” or “Failed to find config”

🔧 I fixed it by syncing the missing gvmd_data:

bash
Copy code
sudo greenbone-feed-sync --type gvmd_data
sudo runuser -u _gvm -- gvmd --rebuild
sudo gvm-stop && sudo gvm-start
🧪 Checked if scan configs appeared:

bash
Copy code
sudo runuser -u _gvm -- gvmd --get-configs
✅ Step 9: Create and Run a Vulnerability Scan
Navigate to: Scans > Tasks > New Task

Task Name: Localhost Scan

Target: 127.0.0.1

Scan Config: Full and fast

Click Save, then Start the task

💡 Learnings & Outcome
Understood end-to-end setup of GVM / OpenVAS on Kali

Troubleshot PostgreSQL version issues

Fixed feed-sync timeout and scan config problems

Learned to use gvmd, manage users, and check services

Successfully performed a vulnerability scan on localhost

📌 Final Tips
Some feed downloads may take 30–60+ minutes

Use sudo gvm-check-setup often to identify and fix issues

If services or users are missing, restart and rebuild using:

bash
Copy code
sudo gvm-stop
sudo runuser -u _gvm -- gvmd --rebuild
sudo gvm-start
📷 Optional: Add Screenshots to Repository
Login screen

Feed sync progress

Scan task created

Vulnerability scan results

This was a great learning experience and helped me understand the core of how vulnerability management systems work under the hood.








