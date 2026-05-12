# CodeAssess — Online Test Platform

A production-ready online test platform supporting 200–300 concurrent users, built with vanilla HTML/CSS/JS + Firebase Auth + Cloud Firestore.

---

## 📁 Folder Structure

```
online-test-platform/
├── index.html               # Main SPA — all sections in one file
├── css/
│   └── style.css            # Complete design system
├── js/
│   ├── firebase-config.js   # (Reference) Firebase init module
│   ├── app.js               # (Reference) State management
│   └── questions.js         # (Reference) Question data
├── firestore.rules          # Security rules — deploy to Firebase
├── firestore.indexes.json   # Firestore composite indexes
├── firebase.json            # Firebase Hosting + Firestore config
└── README.md
```

> **Note:** `js/` files are reference modules. All production code is self-contained in `index.html` for zero-dependency deployment.

---

## 🚀 Setup Steps

### 1. Create Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add project** → give it a name → continue
3. Disable Google Analytics if not needed → **Create project**

### 2. Enable Authentication

1. In Firebase Console → **Authentication** → **Get started**
2. Go to **Sign-in method** tab
3. Enable **Email/Password** provider → Save

### 3. Create Firestore Database

1. Go to **Firestore Database** → **Create database**
2. Choose **Production mode** (rules will be set via deploy)
3. Select a region closest to your users (e.g., `asia-south1` for India)

### 4. Get Firebase Config

1. Go to **Project Settings** (gear icon) → **General**
2. Scroll to **Your apps** → Click **Web** (`</>`)
3. Register app → copy the `firebaseConfig` object

### 5. Update Config in index.html

Find and replace in `index.html` (around line 220):

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",                    // ← Replace
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",  // ← Replace
  projectId: "YOUR_PROJECT_ID",              // ← Replace
  storageBucket: "YOUR_PROJECT_ID.appspot.com",   // ← Replace
  messagingSenderId: "YOUR_SENDER_ID",       // ← Replace
  appId: "YOUR_APP_ID"                       // ← Replace
};
```

### 6. Install Firebase CLI & Deploy

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login-

firebase login

# Initialize (in project folder)
firebase init
# Select: Firestore + Hosting
# Use existing project → select your project
# Firestore rules file: firestore.rules
# Firestore indexes: firestore.indexes.json
# Public directory: . (dot)
# Single page app: No
# Overwrite index.html: No

# Deploy everything
firebase deploy
```

### 7. Deploy Rules & Indexes Only (Optional)

```bash
firebase deploy --only firestore:rules
firebase deploy --only firestore:indexes
firebase deploy --only hosting
```

---

## 🔧 Customization

### Change Questions (MCQ)

Edit `MCQ_QUESTIONS` array in `index.html`:

```javascript
{ id:"mcq_11", section:"Your Topic", question:"Your question?", options:["A","B","C","D"], correct:0 }
```

### Change Coding Problems

Edit `CODING_QUESTIONS` array — each has `id`, `title`, `difficulty`, `description`, `examples`, `constraints`, `starterCode` (per language).

### Change Test Duration

```javascript
const TEST_DURATION_MS = 90 * 60 * 1000; // Change 90 to your duration in minutes
```

---

## 🏗️ Firestore Data Schema

```
users/{userId}
  ├── name: string
  ├── email: string
  ├── phone: string
  ├── startTime: timestamp
  ├── endTime: timestamp | null
  ├── status: "in_progress" | "submitted"
  ├── tabSwitches: number
  ├── createdAt: timestamp
  │
  ├── mcqResponses/ (subcollection)
  │   └── {questionId}
  │       ├── questionId: string
  │       ├── selectedOption: number (0–3)
  │       ├── flagged: boolean
  │       └── updatedAt: timestamp
  │
  └── codingResponses/ (subcollection)
      └── {questionId}
          ├── questionId: string
          ├── code: string
          ├── language: "javascript"|"python"|"java"|"c"
          └── updatedAt: timestamp
```

---

## ⚡ Scalability for 200–300 Concurrent Users

| Strategy | Implementation |
|---|---|
| **Offline persistence** | `db.enablePersistence()` — local cache reduces reads |
| **Debounced writes** | 800ms–1500ms debounce on all Firestore writes |
| **Merge writes** | `set(..., { merge: true })` avoids full document rewrites |
| **Subcollections** | MCQ + coding in subcollections (not arrays) for partial reads |
| **Server timestamps** | `FieldValue.serverTimestamp()` prevents clock drift |
| **No polling** | Timer runs client-side; only writes on actual changes |
| **Atomic submit** | Single `.update()` for submission — no batched writes needed |

**Firestore throughput:** Each user generates ~1 write/2 seconds during active use. At 300 users: ~150 writes/sec — well within Firestore's 1M writes/day free tier and 10,000 writes/second limit.

---

## 🔐 Security Features

- **Firebase Auth only** — no anonymous access
- **User-scoped rules** — `request.auth.uid == userId` on all paths
- **Immutable email** — cannot change email after account creation
- **No re-submission** — status can only go `in_progress → submitted`
- **Tab switch detection** — logged to Firestore + shown to user
- **Security headers** — X-Frame-Options, X-XSS-Protection via Firebase Hosting

---

## 📊 Admin: Viewing Results

Use the Firebase Console or write a quick admin script:

```javascript
// Node.js admin script (run server-side)
const admin = require('firebase-admin');
admin.initializeApp({ credential: admin.credential.applicationDefault() });
const db = admin.firestore();

async function getResults() {
  const users = await db.collection('users').where('status', '==', 'submitted').get();
  for (const user of users.docs) {
    const data = user.data();
    const mcq = await db.collection('users').doc(user.id).collection('mcqResponses').get();
    console.log(`${data.name} (${data.email}) — ${mcq.size} MCQ answers`);
  }
}
getResults();
```

---

## 🌐 Local Development

```bash
# Serve locally (no Firebase CLI needed)
npx serve .
# or
python3 -m http.server 3000
```

> **Important:** Firebase Auth requires a proper domain. For local dev, Firebase automatically allows `localhost`.

---

## ✅ Feature Checklist

- [x] Welcome page with instructions
- [x] Firebase Auth (register + login)
- [x] User profile saved to Firestore on registration
- [x] MCQ section with 10 questions
- [x] Single-select options with visual feedback
- [x] Question palette (Answered / Unanswered / Flagged)
- [x] Mark for Review (flag) toggle
- [x] Clear answer button
- [x] MCQ auto-save with debounce (800ms)
- [x] Coding section with 3 problems
- [x] Monaco-style textarea editor
- [x] Language selector (C, JS, Python, Java)
- [x] Starter code per language per problem
- [x] Code auto-save with debounce (1500ms)
- [x] Save indicator (saving / saved)
- [x] 90-minute countdown timer (color-coded)
- [x] Timer uses server startTime (prevents cheating)
- [x] Auto-submit on timer expiry
- [x] Tab switch detection + Firestore logging
- [x] Submit page with full summary
- [x] Confirmation checkbox before submit
- [x] Final submit locks test + saves endTime
- [x] Restore session on page refresh
- [x] Offline persistence
- [x] Responsive design (mobile-friendly)
- [x] Security headers via Firebase Hosting
- [x] Firestore security rules
- [x] Composite indexes for admin queries
