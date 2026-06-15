# 🚀 CI/CD Pipeline using Git + Cron + Nginx (Ubuntu)

This project demonstrates a **simple automated CI/CD pipeline without GitHub Actions or Jenkins** using:

- Git (version control)
- Cron (automation)
- Nginx (web server)
- Node.js build system (optional)
- Ubuntu server

---

## 📁 Project Structure
/home/test_project_cicd/test_todo → Git repository (source code)
/var/www/html/myapp → Nginx deployed build
---

## ⚙️ Requirements

Install required tools:

```bash
sudo apt update
sudo apt install nginx git nodejs npm -y
```
🔐 Step 1: Required Permissions Setup (IMPORTANT)

This avoids using sudo inside cron jobs.

1. Create deployment folder
```
sudo mkdir -p /var/www/html/myapp
```
3. Set ownership (VERY IMPORTANT)
```
sudo chown -R $USER:www-data /var/www/html/myapp
```
4. Set safe permissions
```
sudo chmod -R 755 /var/www/html/myapp
```
5. Add user to www-data group
```
sudo usermod -aG www-data $USER
newgrp www-data
```
📌 Step 2: Create Git Repository
```
cd ~/test_project_cicd/test_todo
git init
git add .
git commit -m "Initial commit"
```

Connect GitHub:
```
git remote add origin https://github.com/yourusername/your-repo.git
git branch -M main
git push -u origin main
```
📌 Step 3: Install and Start Nginx
```
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
📌 Step 4: Configure Nginx

Edit default config:
```
sudo nano /etc/nginx/sites-available/default
```
Set root path:
```
server {
    listen 80;
    server_name _;

    root /var/www/html/myapp;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}


```
Restart Nginx:
```
sudo nginx -t
sudo systemctl restart nginx
```
📌 Step 5: CI/CD Deployment Script

Create script:
```
nano ~/deploy.sh
```
🚀 Full CI/CD Script
```
#!/bin/bash

REPO="/home/pranav/test_project_cicd/test_todo"
DEPLOY="/var/www/html/myapp"

cd "$REPO" || exit 1

# Get latest changes
git fetch origin main

LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse origin/main)

# Skip if no changes
if [ "$LOCAL" = "$REMOTE" ]; then
    echo "$(date): No changes"
    exit 0
fi

echo "$(date): Changes detected, deploying..."

git pull origin main

# Build step (if needed)
npm install
npm run build

# Deploy files
rm -rf "$DEPLOY"/*
cp -r dist/* "$DEPLOY"/
```
echo "$(date): Deployment successful"

📌 Step 6: Give Execution Permission
```
chmod +x ~/deploy.sh
```
📌 Step 7: Setup Cron Job

Open cron editor:
```
crontab -e
```
Add:
```
*/2 * * * * /home/pranav/deploy.sh >> /home/pranav/deploy.log 2>&1
```

📌 Step 8: How It Works
Developer pushes code to GitHub
Cron runs every 2 minutes
Script checks Git difference
If changes exist:
Pull latest code
Install dependencies
Build project
Deploy to Nginx folder


Website updates automatically
🌐 Access Your App
http://your-server-ip/

## ⚙️ Upgrade(optional):
```
#!/bin/bash

REPO="/home/pranav/test_project_cicd/test_todo"
DEPLOY="/var/www/html/myapp"
BRANCH="main"
LOG="/home/pranav/deploy.log"

echo "==============================" >> "$LOG"
echo "$(date): CI/CD started" >> "$LOG"

cd "$REPO" || exit 1

git fetch origin "$BRANCH" >> "$LOG" 2>&1

LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse origin/"$BRANCH")

if [ "$LOCAL" = "$REMOTE" ]; then
    echo "$(date): No changes detected" >> "$LOG"
    exit 0
fi

echo "$(date): Changes found, deploying..." >> "$LOG"

git pull origin "$BRANCH" >> "$LOG" 2>&1

# Build step (if Node project)
if [ -f "package.json" ]; then
    npm install >> "$LOG" 2>&1
    npm run build >> "$LOG" 2>&1
fi

rm -rf "$DEPLOY"/*

if [ -d "dist" ]; then
    cp -r dist/* "$DEPLOY"/
elif [ -d "build" ]; then
    cp -r build/* "$DEPLOY"/
else
    cp -r ./* "$DEPLOY"/
fi

echo "$(date): Deployment successful" >> "$LOG"
echo "==============================" >> "$LOG"
```

## NGINX config 

use this for separate out multiple sites
```
nano /etc/nginx/sites-enabled
```
```
# save this entry to nano /etc/hosts :  127.0.0.1 myapp.local
server {
    listen 80;
    server_name myapp.local;

    root /var/www/html/myapp;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

```




