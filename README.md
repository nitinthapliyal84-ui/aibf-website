# AIBF Website and Admin Panel

The public website for the All India Brahmin Federation, plus an admin panel at `/admin` where approved admins sign in with an email (their admin ID) and password.

**What admins can do**

| Section | What it does |
|---|---|
| Dashboard | Applications waiting, approved members, upcoming events, gallery count |
| Membership | Approve, reject, undo or delete applications from the website form. Search them and export to Excel (CSV) |
| Events | Add, edit and delete events, with an optional photo or poster. Past events show as "Held" automatically |
| Photo gallery | Upload many photos at once (resized automatically), edit captions, delete |
| Leadership cards | The office bearers shown as photo cards at the top of the home page (add their photos here) |
| Committees | The full member list for each term (2026–2029, 2022–2025, and future terms). Edit names and posts, paste rows from Excel, and choose which committee the site shows as current |
| Initiatives | The programs in the Initiatives section, with category, highlight figure and display order |
| Site information | Announcement bar, homepage title, the four key figures, About text, membership fees, office addresses and phone numbers, bank and UPI details |

It runs on **Firebase** (by Google). The free "Spark" plan is enough to start, and no credit card is needed.

---

## Folder contents

```
public/               ← the website (everything here goes online)
  index.html          public site
  admin.html          admin panel
  assets/             styles
  js/config.js        ← you paste your Firebase settings here (step 3)
  js/defaults.js      starter text, events, initiatives, office bearers and photos
  js/committees.js    2026–29 and 2022–25 committees, founders (1984) and Roll of Honours
  assets/starter/     six event photos from the Brahma Jyoti newsletters
firestore.rules       security rules: who can read and change what
firebase.json         hosting settings
```

---

## Setup (about 30 minutes, one time)

### 1. Create a Firebase project
1. Go to <https://console.firebase.google.com> and sign in with the Google account that should **own** the website.
2. Click **Create a project**, name it (for example `aibf-website`) and continue. Google Analytics is optional.

### 2. Turn on the database and login
1. In the left menu: **Build → Firestore Database → Create database**.
   - Location: **asia-south1 (Mumbai)**. This can't be changed later.
   - Choose **Start in production mode**.
2. **Build → Authentication → Get started → Email/Password** → enable the first switch → **Save**.
3. Recommended: **Authentication → Settings → User actions** → untick **Enable create (sign-up)**. Then only you can create admin logins.

### 3. Connect the website to your project
1. Click the gear icon → **Project settings** → under "Your apps" click the **web icon `</>`**.
2. Give it a nickname (for example `aibf-web`). You don't need to tick "Firebase Hosting" here. Click **Register app**.
3. Firebase shows a `firebaseConfig = { ... }` block. Copy the six values into **`public/js/config.js`**:
   ```js
   export const FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "aibf-website.firebaseapp.com",
     projectId: "aibf-website",
     storageBucket: "aibf-website.appspot.com",
     messagingSenderId: "123...",
     appId: "1:123...:web:abc..."
   };
   ```
   These values are meant to be public. The security rules protect your data, not this key.

### 4. Put the website online
You need **Node.js** (LTS version) from <https://nodejs.org>. Then open a terminal (Command Prompt on Windows) **inside this folder** and run:

```bash
npm install -g firebase-tools     # once per computer
firebase login                    # opens the browser; use the owner Google account
firebase use --add                # pick your project, type "default" as the alias
firebase deploy                   # uploads the site AND the security rules
```

When it finishes, it prints your website address, for example **https://aibf-website.web.app**.

> Always deploy with `firebase deploy` (not only `--only hosting`) the first time, so the security rules are uploaded. Without them, the database stays locked and the site falls back to its sample content.

### 5. Create the first admin
1. Firebase console → **Authentication → Users → Add user**. Enter the admin's email and a strong password (12+ characters).
2. Open **https://YOUR-SITE.web.app/admin** and sign in with that email and password.
3. You'll see **"This account isn't an admin yet"** with a **User ID**. Click **Copy**.
4. Firebase console → **Firestore Database → Start collection**:
   - Collection ID: `admins`
   - Document ID: paste the **User ID**
   - Add a field: `email` (string) = the admin's email → **Save**
5. Reload the admin page. You're in.
6. On the Dashboard, click **Load starter content** once and keep the page open until it finishes (about a minute). It copies the text, both committees, 14 events, 11 initiatives, 9 office-bearer cards and 6 photos into the database so you can edit them.

**Adding more admins:** repeat step 5 for each person.
**Removing an admin:** delete their document from the `admins` collection. Deleting their user under Authentication also blocks them from signing in.
**Forgotten password:** use "Forgot password?" on the login page. Firebase emails a reset link.

### 6. Use your own domain (optional)
1. Firebase console → **Hosting → Add custom domain** (for example `aibf.org` or `www.aibf.org`) and add the DNS records it shows at your domain registrar. HTTPS is set up automatically.
2. **Authentication → Settings → Authorized domains → Add domain**, and add the same domain. Admin login won't work on the domain without this.

---

## Updating the website code later
Edit the files, then run `firebase deploy` again from this folder. Day-to-day content (events, photos, members, text) is changed in the admin panel and needs no deploy.

## Try it on your computer first (optional)
```bash
cd public
npx serve .          # or: python -m http.server 8000
```
Open the address it prints. While `config.js` is empty the site runs in **preview mode** with sample content, and the form doesn't save anything.

---

## Free plan limits (Spark)
| Limit | Free amount | What it means for this site |
|---|---|---|
| Database reads | 50,000 / day | About 60–75 reads per homepage visit, so roughly **650+ visits a day** |
| Data downloaded | 10 GB / month | Gallery thumbnails load only when a visitor scrolls near them, about 1–2 MB per visit at most |
| Database storage | 1 GB | Roughly 1,500+ gallery photos |
| Hosting | 10 GB storage, 360 MB/day transfer | Plenty for the site's code |

If the site gets busier, switch to the **Blaze (pay as you go)** plan under *Usage and billing*. You keep the same free amounts each month and pay only for usage above them. Check Firebase's current pricing page, and set a **budget alert** so there are no surprises.

## How security works
- Visitors can read public content and **submit** a membership application. They can never read anyone's application.
- The security rules check every application: exact fields, a valid 10-digit Indian mobile number, length limits, status forced to "pending", and a server-set time.
- Only signed-in users listed in `admins` can see applications or change anything. Nobody can add themselves as an admin from the website. The `admins` list can only be changed in the Firebase console.
- Everything shown on screen is escaped, so text typed into the form can't inject code into the admin panel.
- Recommended once live: turn on **App Check** (Build → App Check) with reCAPTCHA to cut down spam applications.

## Where the content came from
The starter content comes from AIBF's own documents, not from the earlier AI Studio site:
- **Working Committee 2026–29** (latest list dated 10.11.2026): all 104 members and 36 ex-officio state presidents and secretaries, office addresses, emails and the three office phone numbers.
- **Activity Report 2022–25**: founding details (Regd. No. 193/1984), objectives, founders, Roll of Honours, the 2022–25 office bearers, national meetings, campaigns and programs.
- **Brahma Jyoti newsletters** (July 2024, November 2024, January 2025): event details and the six starter photos.
- **Membership flyer** (in the report): membership tiers, UPI ID and WhatsApp number.

**Privacy:** members' personal mobile numbers are **not** on the website. Only the President, Secretary General and Treasurer numbers that are printed on AIBF's letterhead are shown.

**Founders and Roll of Honours** are fixed history kept in `public/js/committees.js`. When a new term starts, add a row to `ROLL_OF_HONOURS` there and redeploy. Committee member lists are edited in the admin panel.

## Before you go live, check these
- **Bank account number and IFSC** came from the AI Studio site. They don't appear in any AIBF document. The UPI ID `9414072750@psbpay` (Punjab & Sind Bank) is confirmed by the membership flyer.
- **Tax exemption:** no AIBF document mentions 12A/80G, so that text is left empty and hidden. Add it in Site information only if the registration is current.
- **Dates where the documents disagree:**
  - Pushkar meeting: the report says 10–11 June 2024, but the newsletter says it began on a Sunday and ended on a Monday, which is 9–10 June. The site uses 9–10 June.
  - 40th Foundation Day: the report says 24–26 November 2024 in one place and 22–24 November in another. The invitation says 22, 23 and 24 November, which the site uses.
  - National Core Committee meeting, Hyderabad: only "March 2025" is known.
- **Treasurer's phone:** the 2026 committee letter gives +91 94412 00610. The older membership flyer gives +91 96524 93610. The site uses the 2026 letter.
- **Photo captions** were written from the newsletter each photo came from. Correct any in Photo gallery if needed.
- **Figures:** 29 state bodies and 65,000 Kutumb members are from the 2025 report. Update them as they change.
