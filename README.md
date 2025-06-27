# 🛡️ Local Vulnerability Assessment using OpenVAS on Kali Linux (VM)

## 🎯 Objective

Perform a full vulnerability assessment on your local Kali Linux machine using **Greenbone/OpenVAS (GVM)** and document all encountered issues, resolutions, and outcomes.

---

## 🧰 Tools Used

* Kali Linux (Virtual Machine on Windows)
* OpenVAS / Greenbone Vulnerability Management (GVM)
* Firefox browser

---

## 🛠️ Installation & Setup Steps

### Step 1: Install GVM and Dependencies

```bash
sudo apt update && sudo apt install -y gvm
```

### Step 2: Clean, Update, and Upgrade

```bash
sudo apt clean
sudo apt update
sudo apt full-upgrade -y
```

> ⚠️ Encountered Issue: GPG error with missing key `827C8569F2518CC677FECA1AED65462EC8D5E4C5`.
>
> ✅ Fix: Add missing key and update again:

```bash
sudo gpg --recv-keys 827C8569F2518CC677FECA1AED65462EC8D5E4C5
sudo apt update
```

### Step 3: Install PostgreSQL 17

```bash
sudo apt install postgresql-17
```

> ⚠️ GVM expected PostgreSQL 17, but system default was 16.
>
> ✅ Fix: Use `pg_upgradecluster` to upgrade:

```bash
sudo pg_upgradecluster 16 main
sudo systemctl restart postgresql
```

### Step 4: Setup GVM

```bash
sudo gvm-setup
```

> ⚠️ Feed download timeout issue with SCAP and CERT.
>
> ✅ Fix: Rerun until completed:

```bash
sudo greenbone-feed-sync --type scap
sudo greenbone-feed-sync --type cert
sudo greenbone-feed-sync --type gvmd_data
```

> 🔁 Sometimes encountered: `/var/lib/gvm/feed-update.lock is locked by another process`
>
> ✅ Wait and retry or remove lock:

```bash
sudo rm /var/lib/gvm/feed-update.lock
```

### Step 5: Check Setup Status

```bash
sudo gvm-check-setup
```

> ✅ Ensures all services, users, feeds, and permissions are set.

---

## 🧑‍💻 User Creation & Fixes

### Create Admin User & Set Password

```bash
sudo gvmd --create-user=admin
sudo gvmd --user=admin --new-password='gvm@123'
```

### Check Existing Users

```bash
sudo runuser -u _gvm -- gvmd --get-users
```

> ⚠️ Issue: `--get-users` returned nothing initially.
>
> ✅ Fix: Restart services and rebuild data:

```bash
sudo gvm-stop
sudo gvm-start
sudo runuser -u _gvm -- gvmd --rebuild
```

---

## 🔌 Starting GVM Services

```bash
sudo gvm-start
```

> ✅ Verify:

```bash
sudo systemctl status gvmd
sudo systemctl status gsad
sudo systemctl status ospd-openvas
```

Open Firefox and go to:

```
https://127.0.0.1:9392
```

> ⚠️ SSL Warning → Click "Advanced" > Accept risk.

Login with:

```
Username: admin
Password: gvm@123
```

---

## ⚠️ Scan Config Issue and Fix

> ⚠️ Error: `No Scan Configs available` or `Failed to find config`

### Fix:

```bash
sudo greenbone-feed-sync --type gvmd_data
sudo runuser -u _gvm -- gvmd --rebuild
sudo gvm-stop && sudo gvm-start
```

Check again:

```bash
sudo runuser -u _gvm -- gvmd --get-configs
```

Once you see scan configs like `Full and fast`, `Base`, `Host Discovery`, you're ready.

---

## ✅ Scan Creation Steps

1. Navigate to **Scans > Tasks > New Task**
2. Enter:

   * **Name:** Localhost Scan
   * **Target:** 127.0.0.1 (Create new target if not existing)
   * **Scan Config:** Full and fast
3. Click **Save** and then **Start** scan

> ⚠️ I initially couldn't see the green running bar or scan configs — had to wait for all feeds to finish syncing and show under Scan Configs tab.

> ✅ Once scan started, results were visible in green with report links.

---

## 🧪 Task-Level Errors (Ongoing Work)

* Encountered feed rebuild error: `A feed sync is already running. Failed to rebuild NVT cache.`

  * Still investigating if it’s a background sync delay or needs manual termination.

---

## 📝 Outcome & Learnings

* I now understand how to set up, troubleshoot, and perform vulnerability assessments with OpenVAS.
* Fixed PostgreSQL version conflicts, feed sync issues, user creation errors.
* Still monitoring the feed rebuild error during concurrent syncs.

---

## 📌 Final Note

This project helped me gain deep understanding of how vulnerability scanning works on Linux systems. It took multiple trials, error fixes, and restarts to get things right — all of which added to my learning.

---

**Author:** Latha Kola
**Status:** 🛠️ *Working on Task Feed Sync Bug*
**Last Updated:** June 27, 2025



