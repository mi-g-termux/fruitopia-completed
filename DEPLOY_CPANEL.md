# Deploy to cPanel (Shared Hosting)

## Overview

cPanel shared hosting is **limited** (no SSH, file-based only). This guide uses:
- Local build on your computer
- FTP upload to cPanel
- Node.js (if your host supports it)
- Static file serving

---

## Option 1: Node.js on cPanel (RECOMMENDED)

### Requirements
- cPanel with Node.js support enabled
- SSH or Node.js App manager in cPanel

### Step 1: Build Locally
On your computer:
```bash
npm run build:cpanel
```

This creates a `dist/` folder with:
- Your React app
- `firebase-config.json`
- All static files

### Step 2: Prepare .env for Production
Create `.env` in your project root:
```
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=1:000000000:web:abc123
VITE_FIREBASE_DATABASE_ID=(default)

NODE_ENV=production
PORT=8080
```

### Step 3: Upload via FTP
Using FileZilla or cPanel File Manager:

1. Connect to your hosting FTP server
2. Navigate to `public_html/` folder
3. Create a folder: `fruitopia`
4. Upload these files:
   - `server.ts`
   - `package.json`
   - `package-lock.json`
   - Entire `dist/` folder
   - `.env` file (NOT `.env.example`)

Folder structure:
```
public_html/
  └── fruitopia/
       ├── package.json
       ├── package-lock.json
       ├── server.ts
       ├── .env
       └── dist/
           ├── index.html
           ├── firebase-config.json
           └── js/
```

### Step 4: Install Dependencies in cPanel

Option A: Using cPanel SSH Terminal
```bash
cd public_html/fruitopia
npm install
npm start
```

Option B: Using cPanel Node.js App Manager
1. Login to cPanel
2. Go to "Node.js App Manager"
3. Click "New Node.js App"
4. Set:
   - **Node.js Version:** 18 or higher
   - **Application Root:** `public_html/fruitopia`
   - **Application Entry Point:** `server.ts`
   - **Port:** 8080
5. Click "Create"

### Step 5: Verify
- Visit `yoursite.com:8080` (or whatever port was assigned)
- App should load with Firebase 🎉

---

## Option 2: Static Build Only (No Node.js)

If your cPanel doesn't support Node.js:

### Step 1: Build React to Static HTML
```bash
npm run build
```

This creates `dist/` with pure static files.

### Step 2: Upload to cPanel
Using cPanel File Manager or FTP:

1. Navigate to `public_html/`
2. Upload all files from `dist/` directly (not in a subfolder):
   - `index.html`
   - `firebase-config.json`
   - `js/`, `css/`, etc.

### Step 3: .htaccess for React Router
Create a `.htaccess` file in `public_html/`:

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  
  # Don't rewrite files and folders
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  
  # Rewrite to index.html for all routes
  RewriteRule ^ index.html [QSA,L]
</IfModule>
```

### Step 4: Verify
- Visit `yoursite.com`
- All routes should work
- BUT: Backend APIs won't work (no Node.js)

**Limitation:** Email, SMS, and payments won't work. Consider upgrading to Node.js hosting.

---

## Firebase Config on cPanel

### With Node.js:
The build script automatically creates `firebase-config.json`. Your `server.ts` serves it:
```
GET /firebase-config.json → dist/firebase-config.json
```

### Static Only:
1. Make sure `firebase-config.json` is in `public_html/`
2. React will fetch it at `/firebase-config.json`
3. Make sure it's valid JSON

### Verify Config Loaded:
1. Open DevTools (F12)
2. Console tab
3. Check: `localStorage.getItem('fruitopia_firebase_source')`
4. Should be: `"file"` or `"env"` (not `"none"`)

---

## Update Firebase Credentials

If you need to change Firebase credentials:

### Method 1: Edit .env and Re-upload
1. Update `.env` locally
2. Rebuild: `npm run build:cpanel`
3. Re-upload `dist/firebase-config.json` via FTP
4. Restart Node.js app in cPanel

### Method 2: Edit via cPanel File Manager
1. Login to cPanel
2. File Manager
3. Navigate to `public_html/fruitopia/dist/`
4. Right-click `firebase-config.json`
5. Click "Edit"
6. Update values
7. Save

---

## Troubleshooting

### "firebase-config.json not found"
**Fix:** Make sure the file exists in `public_html/fruitopia/dist/`

### "Connection refused" on port 8080
**Fix:**
- cPanel Node.js port might be different
- Check cPanel Node.js App Manager for assigned port
- Use that port in the URL (e.g., `yoursite.com:3000`)

### App loads but Firebase is empty
**Fix:**
1. Check `firebase-config.json` exists and has values
2. Verify it's valid JSON (not corrupted during upload)
3. Check browser console for errors

### "npm install failed"
**Fix:**
- Node.js might not be installed on your host
- Contact your hosting provider
- Consider upgrading to a Node.js-enabled plan

### FTP upload too slow
**Fix:**
- Don't upload `node_modules/` folder (it's huge)
- Run `npm install` on the server instead
- Only upload essential files

---

## Costs

Typical cPanel hosting with Node.js support:
- ~$5-15/month with Node.js
- ~$3-5/month without Node.js (static only)

---

## Advantages of cPanel
- ✅ Affordable
- ✅ User-friendly
- ✅ No credit card needed (usually)
- ✅ Easy file management

## Disadvantages
- ❌ Limited scalability
- ❌ Shared resources
- ❌ Slower than cloud platforms
- ❌ Less monitoring/debugging tools

---

## Better Alternatives (If cPanel is slow)

For production use, consider:
- **Render** - Easy, affordable (~$7/month)
- **Vercel** - Zero-config, free tier available
- **Railway** - Simple, similar to Render
- **Heroku** - Reliable, but more expensive

---

## Next Steps

1. ✅ Deploy to cPanel
2. 🔌 Test payment gateways
3. 📊 Monitor performance
4. Consider upgrading to Render/Vercel if traffic grows
