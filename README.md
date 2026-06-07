# florrent MRP

A lightweight, browser-based Materials Requirements Planning system for florrent, hosted on GitHub Pages with real-time shared data via Firebase Firestore.

---

## How it works

| Layer | Service | Cost |
|---|---|---|
| Hosting | GitHub Pages | Free |
| Shared real-time data | Firebase Firestore | Free (Spark plan) |
| Authentication | Password gate (SHA-256) | Built-in |

All users who know the password share the same live database. Changes made by one user appear for others within ~2 seconds.

> **Note on file attachments:** Uploaded files (shipping labels, vendor docs, customer POs) are stored in the browser session only and are not synced to other users via Firestore (Firestore has a 1MB document limit). All other data — quotes, POs, invoices, work orders, customers, vendors, inventory — syncs in real time.

---

## Setup: Step by step

### Step 1 — Create a Firebase project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**, name it `florrent-mrp`, click through the setup (disable Google Analytics if you want)
3. Once created, click **Firestore Database** in the left sidebar
4. Click **Create database** → choose **Production mode** → pick a region close to you (e.g. `us-east1`) → **Enable**
5. Go to **Firestore → Rules** and replace the rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /mrp/shared {
      allow read, write: if true;
    }
  }
}
```

Click **Publish**. (This allows anyone with the URL to read/write — the password gate in the HTML is your access control.)

### Step 2 — Get your Firebase config

1. In Firebase Console, click the ⚙️ gear icon → **Project settings**
2. Scroll down to **Your apps** → click the `</>` (Web) icon
3. Register the app (name it anything, e.g. `mrp-web`)
4. Copy the `firebaseConfig` object — it looks like:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "florrent-mrp.firebaseapp.com",
  projectId: "florrent-mrp",
  storageBucket: "florrent-mrp.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

### Step 3 — Edit `florrent_mrp_v5.html`

Open the file in any text editor. Find this block near the top (around line 120):

```js
const firebaseConfig = {
  apiKey: "REPLACE_WITH_YOUR_API_KEY",
  authDomain: "REPLACE_WITH_YOUR_AUTH_DOMAIN",
  projectId: "REPLACE_WITH_YOUR_PROJECT_ID",
  storageBucket: "REPLACE_WITH_YOUR_STORAGE_BUCKET",
  messagingSenderId: "REPLACE_WITH_YOUR_MESSAGING_SENDER_ID",
  appId: "REPLACE_WITH_YOUR_APP_ID"
};
```

Replace each `"REPLACE_WITH_YOUR_..."` value with your actual values from Step 2. Save the file.

### Step 4 — Put it on GitHub Pages

1. Create a new GitHub repository (can be private or public — GitHub Pages works on both with the right plan, or just make it public)
2. Upload `florrent_mrp_v5.html` and rename it to `index.html` in the repository root (GitHub Pages serves `index.html` by default)
3. Go to **Settings → Pages** in your repository
4. Under **Source**, select **Deploy from a branch** → branch: `main` → folder: `/ (root)` → **Save**
5. After ~60 seconds, your MRP will be live at:
   ```
   https://YOUR-GITHUB-USERNAME.github.io/YOUR-REPO-NAME/
   ```

---

## Changing the password

The password is checked client-side using SHA-256. To change it:

1. Compute the SHA-256 hash of your new password. You can do this at [https://emn178.github.io/online-tools/sha256.html](https://emn178.github.io/online-tools/sha256.html)
2. In `index.html`, find the line:
   ```js
   const PW_HASH = "7648691c81e4de169762f3bd860b246364e9b819e2e767ee9e1b60c2275a7319";
   ```
3. Replace the hash value with your new hash.
4. Commit and push — GitHub Pages will update within ~60 seconds.

---

## Users

The three users (Jose, Joe, Kaitlin) are defined in the HTML. To add or change users, find this section and edit it:

```js
const USERS=[{id:"u1",name:"Jose"},{id:"u2",name:"Joe"},{id:"u3",name:"Kaitlin"}];
```

---

## Resetting all data

To wipe the Firestore database and start fresh:
1. Go to Firebase Console → Firestore Database
2. Find the `mrp` collection → `shared` document → click the three-dot menu → **Delete document**

The next user to make any change will create a fresh document.

---

## Repository structure

```
/
├── index.html        ← the entire MRP app (rename florrent_mrp_v5.html to this)
└── README.md         ← this file
```

---

## Security notes

- The password is checked in the browser via SHA-256. This prevents casual access but is not enterprise-grade security — anyone who can see the HTML source can extract the hash and attempt an offline brute-force.
- For stronger security, consider adding Firebase Authentication (email/password) and updating the Firestore rules to require `request.auth != null`.
- The Firestore API key in the HTML is safe to expose publicly — Firebase API keys are not secrets. Access is controlled by Firestore Security Rules.
