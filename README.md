# Ledger POS

A single-page POS web app. No build step required — but it now needs a free
**Firebase** project so that every browser/device that opens the site sees
the **same shared data** (products, sales, users, settings), instead of each
browser having its own separate copy.

## 1. Create a free Firebase project (~3 minutes)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with any Google account.
2. Click **Add project**, give it any name (e.g. `ledger-pos`), and finish the wizard (you can turn off Google Analytics — not needed).
3. In the left sidebar, click **Build → Firestore Database**.
4. Click **Create database**. Choose any region close to you, and select **Start in test mode** (lets the app read/write for now — see the security note at the bottom before going fully live/public).
5. In the left sidebar, click the gear icon → **Project settings**.
6. Scroll to "Your apps", click the **`</>` (web) icon**, give the app a nickname, and click **Register app**. Do NOT check "Firebase Hosting" — you're using GitHub Pages instead.
7. Firebase will show you a `firebaseConfig` object like this:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "ledger-pos-12345.firebaseapp.com",
     projectId: "ledger-pos-12345",
     storageBucket: "ledger-pos-12345.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef123456"
   };
   ```
   Copy the whole thing.

## 2. Paste your config into `index.html`

1. Open `index.html` in any text editor.
2. Find this block near the top of the `<script>` tag (search for `YOUR_API_KEY`):
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```
3. Replace it with the real config object you copied from Firebase.
4. Save the file.

## 3. Push this project to GitHub

**Option A — via the GitHub website (no git needed)**
1. Go to [github.com/new](https://github.com/new) and create a repository (e.g. `ledger-pos`). It can be public or private — either works with Render.
2. Click **Add file → Upload files**, drag in every file from this project folder (`index.html`, `README.md`, `render.yaml`, `.gitignore`), and commit.

**Option B — using git**
```bash
git init
git add .
git commit -m "Initial commit: Ledger POS with Firestore sync"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/ledger-pos.git
git push -u origin main
```

## 4. Deploy on Render

1. Go to [dashboard.render.com](https://dashboard.render.com) and sign in (you can sign in with your GitHub account).
2. Click **New → Static Site**.
3. Connect your GitHub account if you haven't already, then select the `ledger-pos` repository.
4. Render will detect the included `render.yaml` and pre-fill the settings automatically:
   - **Build Command:** not needed (this is a static HTML file, nothing to build)
   - **Publish directory:** `./` (repo root, since `index.html` lives there)
5. Click **Create Static Site**.
6. After a minute or two, Render gives you a live URL like:
   `https://ledger-pos.onrender.com`

From then on, every time you push a change to the `main` branch on GitHub, Render automatically redeploys the site.

## How the sync works

- All app data (users, products, sales, settings) now lives in a **Cloud
  Firestore** database instead of the browser's local storage.
- The first time the site loads with an empty database, it seeds itself with
  the same default demo users/products it always had.
- Every change (a sale, a new product, editing stock, etc.) is written to
  Firestore immediately, and every open tab/device listens for changes and
  updates within a second or two — so a sale rung up on your phone shows up
  in the inventory view on your laptop automatically.
- If there's no internet connection (or the config hasn't been filled in
  yet), the app quietly falls back to a local-only mode so it still works,
  it just won't sync until the connection/config is fixed.

## ⚠️ Security note

"Test mode" Firestore rules allow **anyone with your config** to read and
write your database — fine for trying this out, but not for a real store
with real sales data on a public GitHub repo. Before relying on this for
real business use, at minimum:

1. In Firebase Console → Firestore Database → **Rules**, tighten the rules
   (e.g. require Firebase Authentication, or restrict to specific fields).
2. Consider adding a login step backed by Firebase Authentication rather
   than the app's current PIN-only login, since PINs are stored in the
   database and are not a strong access control on their own.

If you want help locking this down for production use, just ask.
