# Deploy to VPS (Virtual Private Server)

## Overview

VPS gives you full control. This guide covers:
- Ubuntu/Debian VPS setup
- Node.js installation
- PM2 process management
- Nginx reverse proxy
- SSL with Let's Encrypt
- Firebase config management

---

## Prerequisites

- VPS with Ubuntu 20.04+ or Debian
- SSH access to your server
- Domain name (optional but recommended)
- ~30 minutes setup time

### Recommended VPS Providers
- DigitalOcean ($6+/month)
- Linode ($6+/month)
- Vultr ($2.50+/month)
- Hetzner (EU, €3+/month)

---

## Step 1: Server Setup

### Connect via SSH
```bash
ssh root@YOUR_SERVER_IP
```

### Update System
```bash
apt update && apt upgrade -y
```

### Install Node.js (18 LTS)
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
apt install -y nodejs
node --version   # Should show v18.x
npm --version    # Should show 9.x
```

### Install PM2 (Process Manager)
```bash
npm install -g pm2
pm2 startup
pm2 save
```

### Install Nginx (Web Server)
```bash
apt install -y nginx
systemctl enable nginx
systemctl start nginx
```

### Install Certbot (SSL Certificates)
```bash
apt install -y certbot python3-certbot-nginx
```

---

## Step 2: Clone & Setup Your App

### Create App Directory
```bash
mkdir -p /var/www/fruitopia
cd /var/www/fruitopia
```

### Clone Your Repository
```bash
git clone https://github.com/YOUR_USERNAME/fruitopia.git .
```

Or upload via SCP:
```bash
scp -r ./fruitopia/* user@server:/var/www/fruitopia/
```

### Create .env File
```bash
cat > .env << 'EOF'
# Firebase Configuration
VITE_FIREBASE_API_KEY=your_api_key_here
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=1:000000000:web:abc123
VITE_FIREBASE_DATABASE_ID=(default)

# Server Configuration
NODE_ENV=production
PORT=3005
EOF
```

### Install Dependencies & Build
```bash
npm install
npm run build:vps
```

---

## Step 3: Setup Nginx

### Create Nginx Config
```bash
cat > /etc/nginx/sites-available/fruitopia << 'EOF'
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Redirect to HTTPS (after SSL is setup)
    # return 301 https://$server_name$request_uri;

    location / {
        proxy_pass http://localhost:3005;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
EOF
```

### Enable Site
```bash
ln -s /etc/nginx/sites-available/fruitopia /etc/nginx/sites-enabled/
nginx -t  # Test config
systemctl reload nginx
```

---

## Step 4: Setup SSL (HTTPS)

### Get Certificate
```bash
certbot certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

This will:
1. Generate free SSL certificate
2. Auto-configure Nginx
3. Setup auto-renewal

### Verify Auto-Renewal
```bash
certbot renew --dry-run
```

---

## Step 5: Start Your App

### Start with PM2
```bash
cd /var/www/fruitopia
pm2 start server.ts --name "fruitopia"
pm2 save
```

### Verify It's Running
```bash
pm2 status
pm2 logs fruitopia
```

### Visit Your Site
- Open `https://yourdomain.com` 🎉

---

## Step 6: Monitor & Maintain

### View Logs
```bash
pm2 logs fruitopia
# Or latest errors
pm2 logs fruitopia --err
```

### Monitor Performance
```bash
pm2 monit
```

### Restart App
```bash
pm2 restart fruitopia
```

### Stop App
```bash
pm2 stop fruitopia
pm2 delete fruitopia
```

---

## Firebase Config on VPS

### How It Works
1. You set `VITE_FIREBASE_*` env vars in `.env`
2. Run `npm run build:vps`
3. Build script reads `.env` and creates `firebase-config.json`
4. Server.ts serves it at `/firebase-config.json`
5. React app loads it ✅

### Update Firebase Credentials

Edit `.env`:
```bash
nano /var/www/fruitopia/.env
# Edit values
# Save: Ctrl+O, Enter, Ctrl+X
```

Rebuild and restart:
```bash
cd /var/www/fruitopia
npm run build:vps
pm2 restart fruitopia
```

---

## Firewall & Security

### Setup Firewall
```bash
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

### Create Non-Root User
```bash
adduser appuser
usermod -aG sudo appuser
su - appuser
```

Then run your app as non-root (recommended for production).

---

## Backup Strategy

### Manual Backup
```bash
# Backup your app
tar -czf fruitopia-backup-$(date +%Y%m%d).tar.gz /var/www/fruitopia

# Download locally
scp user@server:fruitopia-backup-*.tar.gz ~/backups/
```

### Automated Backup (Cron)
```bash
# Edit crontab
crontab -e

# Add this line (backup every day at 2 AM)
0 2 * * * tar -czf /home/backup/fruitopia-$(date +\%Y\%m\%d).tar.gz /var/www/fruitopia
```

---

## Upgrade Node.js

```bash
# Check current version
node --version

# Update
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt install -y nodejs

# Restart app
pm2 restart fruitopia
```

---

## Troubleshooting

### App Won't Start
```bash
pm2 logs fruitopia --err
# Check for errors in output
```

### Port Already in Use
```bash
# Find process using port 3005
lsof -i :3005
# Kill it
kill -9 PID
# Or change PORT in .env and rebuild
```

### SSL Certificate Error
```bash
# Renew certificate manually
certbot renew --force-renewal

# Check certificate
certbot certificates
```

### Firebase Config Not Loading
```bash
# Check if file exists
ls -la /var/www/fruitopia/dist/firebase-config.json

# Check contents
cat /var/www/fruitopia/dist/firebase-config.json

# Rebuild if missing
npm run build:vps
```

### High CPU/Memory Usage
```bash
# Check memory
free -h

# Check CPU
top

# Check Node.js processes
pm2 monit
```

---

## Costs

Typical VPS hosting:
- DigitalOcean: $6/month (1GB RAM)
- Linode: $6/month
- Vultr: $2.50-5/month

Much cheaper than Render/Vercel long-term if you need high traffic.

---

## Advantages of VPS
- ✅ Full control
- ✅ Better performance
- ✅ Cheap for high traffic
- ✅ Can install anything
- ✅ No deployment limits

## Disadvantages
- ❌ More setup required
- ❌ You maintain security
- ❌ More complex monitoring
- ❌ Need SSH knowledge

---

## When to Use VPS

✅ Use VPS if:
- You expect high traffic
- You want full control
- You need custom configurations
- You plan to run multiple apps
- You're familiar with server management

❌ Use Render/Vercel if:
- You want zero-config deployment
- You're just starting out
- You want managed updates
- Traffic is moderate

---

## Next Steps

1. ✅ Deploy to VPS
2. 🔍 Monitor with PM2
3. 🔒 Setup regular backups
4. 🚨 Setup monitoring/alerts
5. 📈 Scale as traffic grows
