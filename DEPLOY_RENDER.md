# Deploy to Render

## Quick Start (5 minutes)

### Step 1: Connect Your Repository
1. Go to [render.com](https://render.com)
2. Click "New" → "Web Service"
3. Connect your GitHub repo
4. Select your branch (usually `main`)

### Step 2: Configure Build & Deploy
| Setting | Value |
|---------|-------|
| **Name** | fruitopia |
| **Environment** | Node |
| **Build Command** | `npm install && npm run build:render` |
| **Start Command** | `npm run start` |
| **Auto-Deploy** | Yes |

### Step 3: Add Environment Variables
Click **Environment** and add these variables:

```
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=1:000000000:web:abc123
VITE_FIREBASE_DATABASE_ID=(default)
```

**IMPORTANT:** Set "Sync to Git" to **OFF** for each variable (keeps secrets safe)

### Step 4: Deploy
Click "Create Web Service" and wait ~3-5 minutes for the build to complete.

---

## Firebase Config Handling

**How it works on Render:**

1. You add `VITE_FIREBASE_*` env vars in the Render dashboard
2. Render runs `npm run build:render`
3. The build script reads env vars and writes `firebase-config.json` to `dist/`
4. Express server serves the file at `/firebase-config.json`
5. React app loads the config and initializes Firebase ✅

**No file uploads needed!** Everything is handled automatically.

---

## Verify It's Working

1. Wait for deployment to show "Live" (green status)
2. Open your app URL
3. Press **F12** to open DevTools
4. Go to **Console** tab
5. Paste: `localStorage.getItem('fruitopia_firebase_source')`
6. Should return: `"env"` ✅

If you see `"none"`, check:
- All 7 env vars are set
- Env var names are EXACT (case-sensitive)
- You clicked "Create Web Service" (not just configured)
- Build completed without errors (check Logs tab)

---

## Troubleshooting

### "firebase-config.json not found"
**Fix:** Check that all VITE_FIREBASE_* variables are set in Render dashboard

### "Permission denied" on deploy
**Fix:** Make sure your repo is public or Render has access to your GitHub account

### Firebase not loading (shows "Local Mode")
**Fix:** 
1. Check Render **Logs** tab
2. Look for errors mentioning `firebase` or `VITE_FIREBASE`
3. Verify env var values are correct in Render dashboard
4. Redeploy by pushing a new commit

### Port 10000 in use
**Fix:** Render automatically uses port 10000. This is normal.

---

## Update Environment Variables

If you need to change Firebase credentials:

1. Go to Render dashboard → Your service → Settings
2. Click **Environment**
3. Update the variable value
4. Render automatically redeploys (takes 2-3 minutes)

No git push needed! 🎉

---

## Optional: Add More Services

Once Firebase is working, configure via InstallWizard GUI OR add env vars:

- **Email:** `SMTP_HOST`, `SMTP_USER`, `SMTP_PASS`
- **SMS:** `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`
- **Payments:** `STRIPE_SECRET_KEY`, `RAZORPAY_KEY_ID`, etc.

---

## Next Steps

1. ✅ Firebase configured
2. 🔌 Connect payment gateways (optional)
3. 📧 Configure email (optional)
4. 🔒 Set up Firestore security rules
5. 🚀 Monitor logs and users
