# 🔄 Project Auto-Backup & Sync System

This project is configured to **automatically back up** to GitHub daily and supports **manual synchronization** for flexible workflow management.

## 📦 Overview

- **Daily automatic backup**: All local changes are committed and pushed to GitHub every day at **11:59 PM**. Note that the time is in local time. In my case, it is in `Colombo/Asia` time. 
- **Manual sync options**: Easily push or pull changes on demand via simple terminal commands.
- **Secure authentication**: Uses GitHub Personal Access Token (PAT) for safe, passwordless Git operations.

Designed for developers and sysadmins who want reliable, low-effort version control on headless servers.

---

## ⚙️ Setup on Ubuntu Server

### 1. **Install Git**
```bash
sudo apt update && sudo apt install git -y
```

### 2. **Initialize Git Repository**
```bash
cd /path/to/your/project
git init
git remote add origin https://github.com/your-username/your-repo.git
```

### 3. **Authenticate with GitHub**

- Create a [Personal Access Token (PAT)](https://github.com/settings/tokens) with `repo` scope.
  - Go to **GitHub → Settings → Developer settings → Personal Access Tokens → Tokens (classic)**
  - Click Generate new token (classic)
  - Give it a name (e.g., `ubuntu-server-backup`)
  - Select scopes: at least `repo` (for private repos) or `public_repo` (for public)
  - Copy the token (you won’t see it again!)

- Configure Git to store credentials:
  ```bash
  git config --global credential.helper store
  ```
- Perform an initial push (use your GitHub username and PAT as password):
  ```bash
  git add .
  git commit -m "Initial commit"
  git push origin main
  ```

### 3. **Install Backup Script**
Create `/usr/local/bin/github-backup.sh`:
```bash
#!/bin/bash
PROJECT_DIR="/path/to/your/project"
BRANCH="main"

cd "$PROJECT_DIR" || exit 1
git add .

if ! git diff-index --quiet HEAD --; then
    git commit -m "Auto-backup: $(date '+%Y-%m-%d %H:%M:%S')"
    git push origin "$BRANCH"
    echo "[$(date)] Backup pushed successfully."
else
    echo "[$(date)] No changes to commit."
fi
```
Make it executable:
```bash
sudo chmod +x /usr/local/bin/github-backup.sh
```

### 4. **Schedule Daily Backup**
Add to crontab (`crontab -e`):
```cron
59 23 * * * /usr/local/bin/github-backup.sh >> /var/log/github-backup.log 2>&1
```

### 5. **Enable Manual Sync (Optional but Recommended)**
Add to `~/.bashrc`:
```bash
alias git-backup='/usr/local/bin/github-backup.sh'
alias git-sync='cd /path/to/your/project && git pull origin main'
```
Reload config:
```bash
source ~/.bashrc
```

---

## 🖐️ Usage

### Automatic
- Nothing to do! Changes are backed up nightly at **11:59 PM**.

### Manual
| Command | Action |
|--------|--------|
| `git-backup` | Commit & push all local changes to GitHub |
| `git-sync`   | Pull latest changes from GitHub to local |

> 💡 Run these from **any directory** in your terminal.

---

## 📁 Logs
Backup activity is logged to:
```bash
/var/log/github-backup.log
```
View recent entries:
```bash
tail -f /var/log/github-backup.log
```

---

## 🔒 Security Notes
- Your GitHub PAT is stored in `~/.git-credentials`. Ensure your server is secured.
- Restrict file permissions if needed:
  ```bash
  chmod 600 ~/.git-credentials
  ```

---

## 🛠️ Requirements
- Ubuntu Server (or any Linux with `cron` and `git`)
- Git ≥ 2.0
- Internet access to GitHub

---

> ✨ **Peace of mind through automation** — your code is always backed up!  

---

Feel free to customize the paths, branch name (`main` vs `master`), or add your project name at the top! Let me know if you'd like a version with **SSH key authentication** instead of PAT.
