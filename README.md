# 💰 Paisa — Personal Expense Manager

> *Track every rupee, own your story*

A personal expense manager web app with real-time cloud sync, multi-user support, and a clean dark/light UI. Built as a single HTML file — no frameworks, no build process.

🔗 **Live App:** [your-username.github.io/paisa](https://your-username.github.io/paisa)

---

## 📸 Features

### 🔐 Authentication
- Self-registration (name, email, password)
- Sign in / Sign out
- Forgot password (Firebase reset email)
- Each user's data is completely private and isolated

### 🔴 Expense Tracking
- Add / Edit / Delete expenses
- Fields: Description, Amount, Date, Category, Payment Method, Notes
- 22 default categories (alphabetical) + custom user-defined categories
- 3 payment methods: Credit Card, Cash, UPI
- Filter by payment method, search by text
- 📎 Notes indicator on rows that have notes

### 🟢 Received Payments
- Add / Edit / Delete received payment entries
- Fields: Description, From (person/org), Amount, Date, Type, Via, Notes
- 4 default types + custom user-defined types
- 5 received via options: UPI, Bank Transfer, Cash, Cheque, Credit Card

### 📊 Dashboard & Insights
- Personalised greeting: *Good morning/afternoon/evening, [Name] 👋*
- Monthly summary: Total Spent, Total Received, Net Balance
- Monthly Insights: % change vs last month, top category, budget warnings
- Payment method breakdown with amounts
- Category breakdown with animated progress bars
- Month navigation (browse any past/future month)

### 💰 Budget Limits
- Set a monthly spend limit per category
- Visual badges: 🟢 under 80% · 🟡 80–99% · 🔴 over budget
- Insight panel alerts when any category exceeds its budget

### ⚙️ Settings
- Change display name and personal tagline
- Add / remove custom expense categories (emoji + name)
- Add / remove custom received payment types (emoji + name)

### 🌓 UI / UX
- Dark / Light mode toggle (preference saved in browser)
- Real-time sync indicator (Live sync / Saving / Error)
- Toast notifications for all actions
- Fully responsive — works on mobile and desktop
- Hover-to-reveal edit/delete on table rows (always visible on touch)

---

## 🗂️ Default Expense Categories (22)

| | | | |
|---|---|---|---|
| 👗 Clothing | 💳 Credit Card Bill | ⚡ Electricity Bill | 🎬 Entertainment |
| 🍽 Food | ⛽ Fuel | 🔥 Gas / LPG | 🛒 Grocery |
| 💊 Health & Wellness | 🧹 House Cleaning | 🏠 House EMI | 🔧 Installations |
| 🏦 Loan EMI | 📱 Mobile Recharge / WiFi | 🏢 Office Other | 📦 Office Supply |
| 🚌 Office Travel | ✂ Personal Care | 🛍 Shopping | ✈ Travel |
| 🚗 Vehicle Maintenance | 📌 Other | | |

## 💰 Default Received Payment Types (4)
- 🏢 Reimbursement
- 🤝 Loan Returned
- 💼 Freelance / Work
- 💰 Other Income

---

## 🔧 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (single file) |
| Database | Firebase Firestore (real-time, cloud) |
| Auth | Firebase Authentication (Email/Password) |
| Hosting | GitHub Pages |
| SDK | Firebase JS SDK v10.12.0 (loaded via CDN) |

---

## 🏗️ Firebase Configuration

> ⚠️ These are embedded in `index.html`. Do not change unless migrating projects.

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyC8kbENBtBUead5RFBtvBG7qXv4qMzeBmo",
  authDomain: "expense-tracker-app-272c0.firebaseapp.com",
  projectId: "expense-tracker-app-272c0",
  storageBucket: "expense-tracker-app-272c0.firebasestorage.app",
  messagingSenderId: "438870400275",
  appId: "1:438870400275:web:ced8154c4ad3fb65780ee6"
};
```

### Firestore Security Rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

---

## 🗄️ Firestore Data Structure

```
users/
  {uid}/
    profile/
      info          → { name, email, tagline, customCats[], customRecTypes[], budgets{}, createdAt }
    expenses/
      {id}          → { desc, amount, date, category, payment, notes }
    received/
      {id}          → { desc, from, amount, date, type, via, notes }
```

> **Migration note:** Old data (pre-multi-user) was stored at top-level `/expenses` and `/received`. A one-time auto-migration runs on first login, copying docs to the user's folder and marking originals with `_migrated: true`.

---

## 🧠 For AI Assistants — How to Make Changes

If you are an AI assistant helping to update this app, here is everything you need to know:

### Architecture
- The **entire app is one file**: `index.html`
- No build process — edit and upload directly to GitHub
- Firebase credentials are embedded — do not change them

### JavaScript Pattern
All functions exposed to HTML `onclick` attributes follow this pattern to avoid timing issues with ES module loading:

```javascript
// In regular <script> (runs immediately):
window.fnName = function(...args) {
  if (window['_fnName']) return window['_fnName'](...args);
};

// In <script type="module"> (runs after Firebase loads):
window._fnName = function() { /* actual implementation */ };
```

> `switchAuthTab` is an exception — it is pure DOM and defined directly in the regular script block.

### Key Paths
| What | Path |
|---|---|
| User expenses | `users/{uid}/expenses/{id}` |
| User received | `users/{uid}/received/{id}` |
| User profile | `users/{uid}/profile/info` |
| Custom categories | `profile.customCats[]` → `{ name, i (emoji), c (hex color) }` |
| Budgets | `profile.budgets{}` → `{ categoryName: amount }` |
| Theme preference | `localStorage` key: `paisa_theme` → `"dark"` or `"light"` |

### Bugs Fixed — Do Not Reintroduce
1. **Empty `catch()` blocks** — always use `catch(e)`, never `catch()` — causes SyntaxError in some browsers
2. **`window.fn` vs `window._fn` pattern** — HTML onclicks use stubs, module uses `_fn` implementations
3. **`switchAuthTab` warning** — must stay in regular `<script>`, not inside the ES module
4. **Data migration** — old top-level `/expenses` docs are migrated once on login; do not remove this logic

### How to Deploy a Change
1. Make the change to `index.html`
2. Go to `github.com/[username]/paisa`
3. Click `index.html` → pencil ✏️ icon → paste new content → **Commit changes**
4. Wait 1–2 minutes → live site updates automatically

---

## 🚀 Planned / Future Features
- [ ] Export to CSV / Excel
- [ ] Recurring expenses with monthly reminders
- [ ] Kotak Bank CSV statement import
- [ ] Yearly summary and analytics view
- [ ] Bill split / shared expense tracking
- [ ] Change password from within the app

---

## 📁 Repository Structure

```
paisa/
├── index.html    ← entire app (HTML + CSS + JS)
└── README.md     ← this file (project context + AI handover guide)
```

---

*Built with ❤️ using Claude + Firebase + GitHub Pages*
