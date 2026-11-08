# CLAUDE.md — Expense Tracker (iOS)

This file tells Claude Code what this project is, who it's for, and how to work on it.
Read it at the start of every session.

---

## 1. About the person you're working with

- **Zero prior app development experience.** This is their first app.
- Explain what you're doing in plain English before you do it, and why. Define jargon the first time it appears (e.g. "SwiftData is the built-in database that saves data on the phone").
- When they need to do something themselves (in Xcode, the Shortcuts app, or iPhone Settings), give **numbered click-by-click steps**, naming each button or menu exactly as it appears.
- Work in **small steps that build and run**. After each step, tell them how to check it worked ("Press ⌘R; you should see a list with one fake expense").
- If their idea or instruction is a bad approach, **say so and suggest a better one** before writing code.
- If something is ambiguous, **ask one clear question** rather than guessing.
- They have a Mac **only on weekdays**. On days without the Mac, focus on planning, writing code and reviewing, and keep a list of "things to test on the Mac" in `NOTES.md`.
- They live in **Singapore**. Default currency is SGD.

## 2. What we're building

A **private, on-device expense tracker** for iPhone that:

1. **Captures Apple Pay transactions automatically** using the Shortcuts app's *Wallet / Transaction* automation, which passes merchant, amount and card to our app.
2. **Captures bank SMS alerts** (mostly online shopping and in-app purchases) using a Shortcuts *Message* automation that forwards the SMS text to our app.
3. **Categorises each expense with AI**, using whichever AI provider the user chooses (Claude, OpenAI, Gemini, or Apple's on-device model where available), with their **own API key**.
4. Offers an **"Ask AI" button** for an on-demand analysis of their spending.
5. Later: tracks **investments**.

### Core principles (do not break these)

- **No backend, no accounts, no analytics.** All data stays on the user's iPhone. The developer never sees anyone's data.
- **Bring your own AI.** The user enters their own API key. Keys are stored in the **iOS Keychain**, never in plain files, UserDefaults, logs or source code.
- **AI is optional.** The app must be fully usable without an API key (rule-based categories plus manual edits).
- **Send the AI as little as possible.** For categorisation, send only merchant, amount, currency and the list of categories. Never send card numbers or account numbers. For "Ask AI", send aggregated summaries by default, not raw transaction lists.
- **The user owns their data.** Provide CSV export from version 1.

### Platform facts (verified, so don't propose workarounds that break them)

- iOS **does not** let third-party apps read other apps' notifications or the SMS inbox. Don't try. Shortcuts automations are the supported bridge.
- The Wallet transaction trigger only fires for **Apple Pay** (phone or watch), not physical cards. Physical card and cash spending needs manual entry.
- The Wallet trigger can occasionally time out or fire for declined payments. Handle duplicates and let the user delete or edit entries easily.

## 3. Tech stack

| Area | Choice | Why |
|---|---|---|
| Language / UI | Swift + SwiftUI | Apple's modern standard; best documented |
| Storage | SwiftData | Built-in, on-device, simple |
| Shortcuts bridge | App Intents | How Shortcuts passes data into our app |
| Charts | Swift Charts | Built-in |
| Secrets | Keychain (Security framework) | Secure key storage |
| Networking | URLSession | Built-in; no third-party SDKs needed |
| Minimum iOS | 17.0 | Wallet transaction trigger requires iOS 17 |

**Avoid third-party packages** unless there's a strong reason, and explain the reason first.

## 4. Architecture (keep it simple)

```
ExpenseTracker/
├── App/                 # App entry point
├── Models/              # SwiftData models: Expense, Category, (later) Holding
├── Views/               # SwiftUI screens
├── Intents/             # App Intents called by Shortcuts
│   ├── LogApplePayIntent.swift    # receives merchant, amount, card
│   └── LogFromSMSIntent.swift     # receives raw SMS text
├── Parsing/
│   ├── SMSParser.swift            # regex rules per Singapore bank
│   └── BankFormats/               # one file per bank, with sample texts in tests
├── AI/
│   ├── AIProvider.swift           # protocol: categorise(), analyse()
│   ├── ClaudeProvider.swift
│   ├── OpenAIProvider.swift
│   ├── GeminiProvider.swift
│   └── OnDeviceProvider.swift     # Apple Foundation Models, if the device supports it
├── Services/
│   ├── Categoriser.swift          # rules first, AI fallback, remembers user corrections
│   ├── Deduplicator.swift
│   └── KeychainStore.swift
└── Tests/
```

### Key design rules

- **AIProvider protocol.** Every provider implements the same small interface. The rest of the app never knows which provider is in use. Adding a provider means adding one file.
- **Categorisation order:** (1) the user's past corrections for this merchant, (2) built-in keyword rules, (3) AI, (4) "Uncategorised". Cache results per merchant so the same merchant isn't sent to the AI twice.
- **Deduplication:** an Apple Pay entry and an SMS for the same purchase should merge. Match on the same amount and currency within about 10 minutes. When unsure, keep both and flag them for review rather than silently deleting.
- **Source field:** every expense records how it arrived: `applePay`, `sms`, `manual` or `receipt`.
- **Currency:** store the original amount and currency. Show SGD totals; convert foreign amounts only when a rate is available, and mark converted figures as estimates.

## 5. Data model (version 1)

**Expense:** id, amount (Decimal), currency (String, default "SGD"), merchant, rawText (original SMS, optional), date, category, source, cardLabel (optional, e.g. "DBS Visa"; never a full card number), note, needsReview (Bool)

**Category:** id, name, emoji, isDefault

Default categories: Food & Drink, Groceries, Transport (Grab/Gojek/MRT/taxi), Shopping, Bills & Utilities, Entertainment, Health, Travel, Subscriptions, Transfers, Other.

Use **Decimal** for money, never Double.

## 6. Singapore banks: SMS parsing

Target banks: DBS/POSB, OCBC, UOB, Citibank, HSBC, Standard Chartered, Maybank, Trust Bank. Also consider wallets: DBS PayLah!, GrabPay, PayNow notifications.

- **Do not invent SMS formats.** Ask the user to paste real (redacted) sample messages and build each bank's parser and unit tests from those.
- Parse with regex first. If no rule matches, fall back to AI extraction (if a key is set), then to a "needs review" entry with the raw text attached.
- Common things to extract: amount and currency (e.g. "SGD 12.50", "S$12.50"), merchant ("at XYZ"), card's last 4 digits (store as a label only), date and time.
- Ignore OTPs and non-transaction SMS. Never store OTP messages.

## 7. Roadmap

**Phase 1: working core (start here)**
1. Xcode project setup and a "hello world" running on the simulator, then on the user's iPhone
2. Expense model plus manual add, edit, delete and a list view
3. Categories and simple keyword rules
4. Monthly total and a breakdown by category (Swift Charts)
5. `LogApplePayIntent` plus a step-by-step guide to set up the Wallet automation
6. Settings screen: choose AI provider, enter API key (saved in Keychain), test the connection
7. AI categorisation with per-merchant caching
8. CSV export

**Phase 2: AI insights and SMS**
- "Ask AI" screen: preset questions ("How did I do this month?", "Where can I cut back?") plus free text; send aggregated data
- `LogFromSMSIntent`, Singapore bank parsers, deduplication, review queue
- Budgets per category

**Phase 3: extras**
- Investments segment (manual holdings first; price feeds later)
- Receipt photo capture
- Email alerts (needs careful privacy design)
- Distribution to friends (TestFlight)

Don't start a phase until the previous one works on the real iPhone.

## 8. Working conventions

- Keep `NOTES.md` up to date with what's done, what's next, known bugs and things to test on the Mac.
- Commit to git after each working step, with a clear message. Explain git to the user the first time.
- Write unit tests for parsers, the deduplicator and the categoriser. UI tests aren't needed yet.
- Keep files small and focused. Prefer clarity over cleverness. This codebase must stay understandable to a beginner.
- Never put real API keys, real SMS content or personal financial data in code, tests or commits. Use fake samples.
