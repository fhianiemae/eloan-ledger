# Emergency Loan Ledger

A record-keeping system for the Emergency Loan Agreement: fills in the contract, tracks weekly interest (20% → 30% on default), the 3%/day late penalty, payments, and co-maker liability. Data is stored in Firebase (Firestore) and the app is hosted as a static site on GitHub Pages.

## One-time setup

### 1. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and click **Add project**. Name it anything (e.g. `eloan-ledger`) and finish the wizard (Google Analytics is optional, you can skip it).
2. In the left sidebar, go to **Build → Firestore Database → Create database**. Choose a region close to you (e.g. `asia-southeast1`), and start in **production mode**.
3. Go to **Build → Authentication → Get started**. Under **Sign-in method**, enable **Email/Password**.
4. Still in Authentication, go to the **Users** tab → **Add user**, and create the login you (the lender) will use to sign into the app — any email + password.

### 2. Get your web app config

1. In **Project settings** (gear icon, top left) → **General** → scroll to **Your apps** → click the `</>` (web) icon to register a new web app. Any nickname is fine; you don't need Firebase Hosting.
2. Copy the `firebaseConfig` object it shows you.
3. Paste those values into [firebase-config.js](firebase-config.js) in this repo, replacing the `PASTE_...` placeholders.

This config is not secret — it's meant to be public in client code. What actually protects your data is Authentication (only your login can sign in) and the security rules below.

### 3. Publish the security rules

1. In Firestore → **Rules** tab, replace the contents with what's in [firestore.rules](firestore.rules) and click **Publish**.
2. This restricts every loan document to only be readable/writable by the signed-in user who created it (`ownerId`).

### 4. Push your config and deploy

Commit your filled-in `firebase-config.js` and push to `main` — the GitHub Actions workflow in [.github/workflows/deploy.yml](.github/workflows/deploy.yml) publishes the site to GitHub Pages automatically on every push.

If Pages isn't live yet, enable it once in **Settings → Pages → Source → GitHub Actions** on this repo.

## Using the app

Open the Pages URL, sign in with the user you created in step 1.4, and start adding loans. Every change syncs live to Firestore — open the site on another device with the same login and it stays in sync.

- **Dashboard** — every loan with live status (Active / Due soon / Overdue / Paid) and a "Co-maker liable" flag once a loan passes maturity unpaid.
- **New Loan** — fills in the Agreement's clauses from a form; **Download contract** on a loan's page generates a signature-ready copy.
- **Record a payment** — payments are auto-allocated penalty → interest → principal.
- **How calculations work** — the exact assumptions used to turn the Agreement's text into numbers.

## Notes

- The 20%/week (30%/week on default) interest and 3%/day penalty in the Agreement are steep by ordinary lending standards — have a lawyer review the terms and this app's calculation assumptions before relying on them for a real transaction.
- `firebase-config.js` is committed to this (private) repo for convenience. If you'd rather keep it out of git history, add it to `.gitignore` and deploy it via a repository secret injected at build time instead.
