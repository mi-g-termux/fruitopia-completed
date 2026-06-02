# Deployment Guide - All Platforms

## Quick Platform Comparison

| Platform | Cost | Setup Time | Difficulty | Performance | Best For |
|----------|------|-----------|-----------|-------------|----------|
| **Render** | $7-25/mo | 5 min | Easy | ⭐⭐⭐⭐ | Small-medium projects |
| **Vercel** | Free-$20/mo | 3 min | Very Easy | ⭐⭐⭐⭐⭐ | Startups, fast growth |
| **cPanel** | $3-15/mo | 15 min | Medium | ⭐⭐ | Budget, existing hosting |
| **VPS** | $6-20/mo | 30 min | Hard | ⭐⭐⭐⭐ | High traffic, control |

---

## Choose Your Platform

### I want the easiest setup → **VERCEL** or **RENDER**
- ✅ Automatic deployments from Git
- ✅ No server management
- ✅ Free tier available (Vercel)
- ⏱️ 3-5 minutes to deploy

**Next step:** Go to [DEPLOY_VERCEL.md](DEPLOY_VERCEL.md) or [DEPLOY_RENDER.md](DEPLOY_RENDER.md)

---

### I already have cPanel hosting → **CPANEL**
- ✅ Reuse existing hosting
- ✅ Manual control
- ✅ Cheap
- ⏱️ 15 minutes to deploy

**Next step:** Go to [DEPLOY_CPANEL.md](DEPLOY_CPANEL.md)

---

### I want full control → **VPS**
- ✅ Unlimited customization
- ✅ Better performance at scale
- ✅ Cheapest for high traffic
- ⏱️ 30 minutes to setup

**Prerequisites:** Linux/SSH knowledge

**Next step:** Go to [DEPLOY_VPS.md](DEPLOY_VPS.md)

---

## Universal Setup (For All Platforms)

### Before You Deploy

#### 1. Get Firebase Credentials
1. Open [Firebase Console](https://console.firebase.google.com)
2. Select your project
3. Go to Settings ⚙️ → Project Settings
4. Under "Your Apps", click your web app
5. Copy these values:
   - `apiKey`
   - `authDomain`
   - `projectId`
   - `storageBucket`
   - `messagingSenderId`
   - `appId`

#### 2. Prepare Your Repository
```bash
# Clone your repo
git clone https://github.com/YOUR_USERNAME/fruitopia.git
cd fruitopia

# Create .env (copy from .env.example)
cp .env.example .env

# Edit .env with your Firebase values
nano .env  # or use your editor
```

#### 3. Test Locally
```bash
npm install
npm run dev
# Visit http://localhost:5173
```

#### 4. Build & Test
```bash
npm run build
npm start
# Visit http://localhost:3005
# Should load without errors
```

---

## Firebase Config Resolution

Your app tries to load Firebase config in this order:

```
1. /firebase-config.json (file in dist/)
   ↓ If not found...
2. localStorage (InstallWizard GUI)
   ↓ If not found...
3. VITE_FIREBASE_* (environment variables)
   ↓ If not found...
4. src/firebase-applet-config.json (fallback)
   ↓ If all empty...
5. MOCK MODE 🔴 (App runs locally only)
```

### How Each Platform Handles It

**Render & Vercel:**
- Use `VITE_FIREBASE_*` env vars
- Build script creates `firebase-config.json`
- Everything automatic ✅

**cPanel:**
- Upload `firebase-config.json` via FTP
- OR set env vars if host supports Node.js

**VPS:**
- Set `VITE_FIREBASE_*` in `.env`
- Build script creates config
- Nginx serves it ✅

---

## Post-Deployment Checklist

After deploying to your platform:

### 1. Verify Firebase Loaded
Open DevTools (F12) → Console:
```javascript
localStorage.getItem('fruitopia_firebase_source')
```

Should return:
- `"env"` (from environment variables) ✅
- `"file"` (from firebase-config.json file) ✅
- `"localstorage"` (from InstallWizard) ✅
- `"none"` (Firebase not loaded) ❌

### 2. Check Console for Errors
Look for any red errors in the Console tab. Common ones:
- Firebase initialization errors
- Missing environment variables
- File not found errors

### 3. Test Admin Panel
1. Visit `/admin` (or admin button in UI)
2. Try uploading Firebase credentials via InstallWizard
3. Settings should save

### 4. Test a Transaction
If you have products:
1. Add item to cart
2. Try checkout
3. Should reach payment gateway

---

## Update Environment Variables

### After Initial Deployment

If you need to change Firebase credentials or add payment gateways:

#### Render
1. Dashboard → Your Service → Settings
2. Environment Variables
3. Edit or add variables
4. Render auto-redeploys (2-3 min)

#### Vercel
1. Dashboard → Your Project → Settings
2. Environment Variables
3. Edit or add variables
4. Redeploy: `vercel deploy --prod`

#### cPanel
1. Edit `.env` via cPanel File Manager
2. Rebuild: `npm run build:cpanel`
3. Re-upload updated files

#### VPS
1. SSH into server
2. Edit `/var/www/fruitopia/.env`
3. Rebuild: `npm run build:vps`
4. Restart: `pm2 restart fruitopia`

---

## Monitoring & Logs

### Render
→ Dashboard → Logs tab

### Vercel
→ Dashboard → Deployments → View Logs

### cPanel
→ File Manager → View error logs

### VPS
```bash
pm2 logs fruitopia
# Or check at /var/www/fruitopia/logs/
```

---

## Troubleshooting - All Platforms

### Firebase Config Not Loading

**Check List:**
1. ✅ Environment variables set correctly
2. ✅ Variable names are EXACT (case-sensitive)
3. ✅ App has been redeployed/rebuilt
4. ✅ No typos in Firebase credentials
5. ✅ Browser console shows no errors

**If still not working:**
1. Check the deployment logs (see "Monitoring & Logs")
2. Verify Firebase project still exists
3. Try the InstallWizard: `/install` in the browser
4. Check if `firebase-config.json` exists in the served folder

### App Crashes on Startup

**Check:**
```bash
# Render/cPanel: Check logs
# VPS: pm2 logs fruitopia
# Vercel: Check deployment logs
```

Look for:
- `VITE_FIREBASE` undefined
- `Cannot find module`
- `Port already in use`

### Slow Performance

**Render/Vercel:**
1. Check build time (should be < 2 min)
2. Check if cold starts are happening
3. Consider upgrading plan

**VPS:**
```bash
# Check server resources
free -h  # Memory
df -h    # Disk
top      # CPU
```

---

## Costs Breakdown

### Monthly Costs (Estimate)

**Vercel:**
- Free tier: $0
- Pro tier: $20 (unlimited deployments)

**Render:**
- Starter (shared): $7
- Standard (container): $25

**cPanel:**
- Shared hosting: $3-15
- cPanel license included

**VPS:**
- Basic: $6-10
- Mid-range: $15-25
- High-performance: $50+

**Firebase (All Platforms):**
- Spark (free): ✅ Sufficient for testing
- Blaze (pay-as-you-go): ~$1-5/month for small traffic

---

## When to Upgrade Platforms

### Outgrowing Free Tier?
**Move from:** Free Vercel → Paid Vercel/Render

### Scaling to High Traffic?
**Move from:** Render → VPS or self-managed

### Need More Control?
**Move from:** Vercel/cPanel → VPS

### Reducing Costs?
**Move from:** Render → VPS (if tech-savvy)

---

## Recommended Path for Different Users

### Beginner
1. Deploy to **Vercel** (easiest)
2. Free tier is sufficient
3. Upgrade to Pro when needed

### With cPanel Hosting
1. Deploy to **cPanel** (reuse existing)
2. Test everything works
3. Consider upgrading to VPS later

### Technical Founder
1. Deploy to **VPS** (DigitalOcean)
2. More control from day one
3. Scales easily

### Startup Ready
1. Deploy to **Render** (middle ground)
2. Good balance of ease and cost
3. Easy to move to VPS later

---

## File Changes in This Version

Files that were updated/added for multi-platform support:

```
✅ package.json          - Added platform-specific build scripts
✅ .env.example          - Comprehensive env documentation
✅ .gitignore            - Secret protection (NEW)
✅ scripts/post-build.js - Intelligent config handler (NEW)
✅ DEPLOY_RENDER.md      - Render guide (NEW)
✅ DEPLOY_VERCEL.md      - Vercel guide (NEW)
✅ DEPLOY_CPANEL.md      - cPanel guide (NEW)
✅ DEPLOY_VPS.md         - VPS guide (NEW)
```

All other files remain unchanged. Your app logic is **identical** across platforms.

---

## Security Notes

### Secrets Management

**DON'T:**
- ❌ Commit `.env` to Git
- ❌ Share `firebase-config.json`
- ❌ Log environment variables

**DO:**
- ✅ Use .gitignore (already done)
- ✅ Set env vars in platform dashboard
- ✅ Rotate credentials regularly
- ✅ Use Firestore Rules for data security

### Firestore Rules

Your `firestore.rules` file defines who can access what:

```firestore
// Only owner can read/write
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /orders/{orderId} {
      allow read, write: if request.auth.uid == resource.data.userId;
    }
  }
}
```

Apply these rules in Firebase Console!

---

## What's Next?

### Immediate
- ✅ Choose a platform above
- ✅ Follow the platform's deployment guide
- ✅ Verify Firebase loads
- ✅ Test the admin panel

### Short-term (This Week)
- 🔌 Configure payment gateways
- 📧 Setup email (SMTP)
- 📱 Setup SMS (Twilio) [optional]
- 🔒 Deploy Firestore security rules

### Medium-term (This Month)
- 📊 Monitor performance
- 🚨 Setup error alerting
- 🗄️ Setup backups (VPS only)
- 🔄 Test recovery procedures

### Long-term (Ongoing)
- 📈 Monitor traffic trends
- 💰 Optimize costs
- 🚀 Scale infrastructure as needed
- 🔄 Keep dependencies updated

---

## Support & Troubleshooting

### Getting Help

1. **Check the platform-specific guide** (Render/Vercel/cPanel/VPS)
2. **Check deployment logs** (see "Monitoring & Logs")
3. **Check browser console** (F12 → Console tab)
4. **Check Firebase Console** for errors

### Common Issues Solved
- [Firebase config not loading](./FIREBASE_CONFIG_PERMANENT_FIX.md)
- [Platform-specific help](./DEPLOY_RENDER.md) [Vercel](./DEPLOY_VERCEL.md) [cPanel](./DEPLOY_CPANEL.md) [VPS](./DEPLOY_VPS.md)

---

## You're Ready! 🚀

Pick your platform and follow the guide. You've got this! 💪
