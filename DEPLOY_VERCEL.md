# Deploy to Vercel

## Quick Start (3 minutes)

### Step 1: Install Vercel CLI
```bash
npm install -g vercel
```

### Step 2: Login & Link Project
```bash
vercel login
vercel link
```

### Step 3: Add Environment Variables
```bash
vercel env add VITE_FIREBASE_API_KEY
vercel env add VITE_FIREBASE_AUTH_DOMAIN
vercel env add VITE_FIREBASE_PROJECT_ID
vercel env add VITE_FIREBASE_STORAGE_BUCKET
vercel env add VITE_FIREBASE_MESSAGING_SENDER_ID
vercel env add VITE_FIREBASE_APP_ID
vercel env add VITE_FIREBASE_DATABASE_ID
```

When prompted for each variable, paste your values from Firebase Console.

### Step 4: Deploy
```bash
npm run build:vercel
vercel deploy --prod
```

Or from the Vercel dashboard:
1. Go to [vercel.com](https://vercel.com)
2. Click "Add New" → "Project"
3. Import your GitHub repo
4. Add the 7 environment variables
5. Click "Deploy"

---

## How It Works on Vercel

1. Vercel reads `VITE_FIREBASE_*` environment variables
2. Runs `npm run build:vercel`
3. Build script creates `firebase-config.json` in the `dist/` folder
4. Vercel serves the SPA and static files from `dist/`
5. React app loads Firebase config ✅

---

## Important: vercel.json

Your `vercel.json` already has:
```json
{
  "buildCommand": "npm run build:vercel",
  "functions": {
    "api/**": {
      "memory": 1024,
      "maxDuration": 300
    }
  }
}
```

This is already configured. Don't change it.

---

## Firebase Config on Vercel

The config is handled **automatically**:
- Environment variables are read during build
- `firebase-config.json` is written to `dist/`
- Served as a static file
- React app loads it at runtime

No manual file uploads needed!

---

## Verify Deployment

1. Deploy completes (Vercel shows "Ready")
2. Open your app URL
3. Press **F12** → **Console**
4. Check: `localStorage.getItem('fruitopia_firebase_source')`
5. Should return: `"env"` ✅

---

## Update Environment Variables

Via CLI:
```bash
vercel env pull .env.production.local
# Edit the file
vercel env add VITE_FIREBASE_API_KEY
```

Via Dashboard:
1. Go to [vercel.com](https://vercel.com) → Your Project
2. Settings → Environment Variables
3. Update and save
4. Redeploy: `vercel deploy --prod`

---

## Troubleshooting

### "firebase-config.json not found"
**Fix:** Check that all VITE_FIREBASE_* are set in Vercel dashboard

### Build fails with "VITE_FIREBASE_* undefined"
**Fix:** 
1. `vercel env pull` to download env vars locally
2. Check .env file has correct values
3. Push env vars: `vercel env add VITE_FIREBASE_API_KEY`

### Firebase not initializing
**Fix:**
1. Check Vercel build logs
2. Make sure `vercel.json` exists and has correct buildCommand
3. Redeploy: `vercel deploy --prod`

---

## Costs

Vercel Hobby (Free):
- ✅ 12 deployments/month
- ✅ Unlimited bandwidth
- ✅ Web Analytics

For production, consider Vercel Pro ($20/month) for unlimited deployments.

---

## Advantages on Vercel

- ✅ Zero-config deployments from Git
- ✅ Global edge caching
- ✅ Automatic HTTPS
- ✅ Environment variables in dashboard
- ✅ 1-click rollbacks
- ✅ Analytics included

---

## Next Steps

1. ✅ Deploy to Vercel
2. 🔌 Configure payment gateways (optional)
3. 📧 Set up SMTP (optional)
4. 🔒 Firestore security rules
5. 📊 Monitor via Vercel Analytics
