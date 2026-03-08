# 💰 Paisa — Personal Expense Manager

> *Track every rupee, own your story*

A personal expense manager web app with real-time cloud sync, multi-user support, split expenses with friends, and a clean dark/light UI. Built as a single HTML file — no frameworks, no build process.

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
- 🤝 Split chip shown on expenses that have been split

### 🟢 Received Payments
- Add / Edit / Delete received payment entries
- Fields: Description, From (person/org), Amount, Date, Type, Via, Notes
- 4 default types + custom user-defined types
- 5 received via options: UPI, Bank Transfer, Cash, Cheque, Credit Card
- Optionally link a received payment to a person to auto-reduce their balance

### 👥 People & Split Expenses
- Add people (name + optional phone number)
- Split any expense with one or more people with **custom amounts per person**
- Live counter while splitting shows remaining amount (your share)
- People tab shows each person's outstanding balance:
  - 🟢 They owe you
  - 🔴 You owe them
  - ✓ Settled
- Net banner showing total owed to you vs total you owe across all people
- **Full transaction history** per person — every split and payment listed
- **Settle Up** button — marks all dues with a person as cleared in one click
- **Delete expense** → associated split dues removed automatically
- **Edit expense** → old splits cleared, re-split with new amounts
- **Block person deletion** if they have unsettled dues — must settle up first
- Sidebar shows people summary and per-person balance bars

### 📊 Dashboard & Insights
- Personalised greeting: *Good morning/afternoon/evening, [Name] 👋*
- Monthly summary: Total Spent, Total Received, Net Balance
- Monthly Insights: % change vs last month, top category, budget warnings, total owed by people
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

> The `{document=**}` wildcard covers ALL sub-collections automatically — expenses, received, profile, people, and splits are all covered by this single rule. No rule changes needed when adding new collections.

---

## 🗄️ Firestore Data Structure

```
users/
  {uid}/
    profile/
      info          → { name, email, tagline, customCats[], customRecTypes[], budgets{}, createdAt }
    expenses/
      {id}          → { desc, amount, date, category, payment, notes, hasSplit }
    received/
      {id}          → { desc, from, amount, date, type, via, notes, adjustedPersonId }
    people/
      {id}          → { name, phone, createdAt }
    splits/
      {id}          → { personId, amount, desc, date, expenseId, type, settled, createdAt, settledAt? }
```

### Split document types
| `type` field | Meaning | Effect on balance |
|---|---|---|
| `split` | Person owes you their share of an expense | `+amount` (they owe you) |
| `payment` | You received money from this person | `-amount` (reduces what they owe) |

### Balance calculation
`computeBalances()` sums all **unsettled** split documents per person:
- Positive balance = they owe you
- Negative balance = you owe them
- Zero = settled

> **Migration note:** Old data (pre-multi-user) was stored at top-level `/expenses` and `/received`. A one-time auto-migration runs on first login, copying docs to the user's folder and marking originals with `_migrated: true`.

---

## 🧠 For AI Assistants — How to Make Changes

If you are an AI assistant helping to update this app, here is everything you need to know:

### Architecture
- The **entire app is one file**: `index.html`
- No build process — edit and upload directly to GitHub
- Firebase credentials are embedded — do not change them
- `renderedPeople` is a module-level array that mirrors `people` at the time the split modal opens — used to map checkbox index → person

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

> `switchAuthTab` and `toggleTheme` are exceptions — pure DOM, defined directly in the regular script block.

### Key Paths
| What | Path |
|---|---|
| User expenses | `users/{uid}/expenses/{id}` |
| User received | `users/{uid}/received/{id}` |
| User profile | `users/{uid}/profile/info` |
| People | `users/{uid}/people/{id}` |
| Splits | `users/{uid}/splits/{id}` |
| Custom categories | `profile.customCats[]` → `{ name, i (emoji), c (hex color) }` |
| Budgets | `profile.budgets{}` → `{ categoryName: amount }` |
| Theme preference | `localStorage` key: `paisa_theme` → `"dark"` or `"light"` |

### Split UI — How It Works
- Checkbox IDs: `sc_0`, `sc_1`, `sc_2` … (index-based, NOT Firestore doc IDs)
- Amount input IDs: `sa_0`, `sa_1`, `sa_2` …
- `renderedPeople[idx]` maps index back to the person object
- `renderSplitPeople(existingSplits=[])` accepts existing splits to pre-fill on edit
- `getSplitData()` reads checked checkboxes and their amounts → returns `[{personId, amount}]`

### Split Lifecycle Rules
| Action | What happens to splits |
|---|---|
| Add expense with split | New split docs created in `splits/` collection |
| Edit expense | All old splits for that `expenseId` deleted, new ones written |
| Delete expense | All splits for that `expenseId` deleted atomically |
| Receive payment, link to person | A `type:'payment'` split doc with negative amount is added |
| Settle Up | All unsettled splits for that person marked `settled: true` |
| Delete person with balance ≠ 0 | **Blocked** — toast shown, deletion prevented |
| Delete person with balance = 0 | Person + all their split history deleted cleanly |

### Bugs Fixed — Do Not Reintroduce
1. **Empty `catch()` blocks** — always use `catch(e)`, never `catch()` — causes SyntaxError in some browsers
2. **`window.fn` vs `window._fn` pattern** — HTML onclicks use stubs, module uses `_fn` implementations
3. **`switchAuthTab`** — must stay in regular `<script>`, not inside the ES module
4. **Data migration** — old top-level `/expenses` docs are migrated once on login; do not remove this logic
5. **Split checkbox double-toggle** — div onclick and checkbox onclick used to fire together, cancelling each other; fixed by checking `event.target`
6. **Orphan code after replacement** — a previous edit left dead code outside any function causing `Unexpected token '}'`; always verify file after edits
7. **Firestore ID special chars** — never use Firestore doc IDs as DOM element IDs; use index-based IDs (`sc_0`, `sa_0`) instead

### How to Deploy a Change
1. Make the change to `index.html`
2. Go to `github.com/[username]/paisa`
3. Click `index.html` → pencil ✏️ icon → paste new content → **Commit changes**
4. Wait 1–2 minutes → live site updates automatically
5. If old behaviour persists: **Ctrl+Shift+R** (hard reload) to bypass browser cache

---

## 🚀 Planned / Future Features
- [ ] Export to CSV / Excel
- [ ] Recurring expenses with monthly reminders
- [ ] Kotak Bank CSV statement import
- [ ] Yearly summary and analytics view
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
