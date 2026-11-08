# NOTES

Running log for the Expense Tracker project. See `CLAUDE.md` for the full plan.

## Status

**Current phase:** Pre-Phase 1. Planning and test data done on Windows; no Xcode project yet.

## Done

- 2026-11-08: Project folder set up, git initialised, connected to GitHub (`erwinwang89-sketch/expense-tracker`, private). Added `.gitignore`.
- 2026-11-08: `docs/sms-samples.md`: UOB formats (card purchase in SGD and foreign currency, refund) as synthetic look-alikes with parsing notes.
- 2026-11-08: `fixtures/sms-test-cases.json`: 6 parser test cases (4 UOB purchases, 1 refund, 1 unrecognised).
- 2026-11-08: `fixtures/category-rules.json`: 11 categories with emoji, 13 ordered keyword rules for common Singapore merchants, 20 checks (all pass with a quick PowerShell check of the matching logic).

## Next

- [ ] (Mac day) Phase 1, step 1: create the Xcode project (SwiftUI, SwiftData, iOS 17), run "hello world" in the simulator, then on the iPhone
- [ ] Add SMS samples from other banks/wallets as they arrive (DBS/POSB, OCBC, PayLah!, GrabPay, PayNow…)
- [ ] Phase 1, step 3: load `fixtures/category-rules.json` in the app for keyword categorisation

## Things to test on the Mac

- When the Xcode project exists, add both fixtures files to the test target and write unit tests that loop over `cases` / `checks`.
- Check Swift parses the `SGD 1,234.56` amount (comma removed) into an exact `Decimal`.
- Check `DD/MM/YY` dates are read day-first (e.g. `05/03/26` = 5 March 2026).

## Known bugs

- (none yet)

## Open questions

- **Payment-processor merchants** (e.g. `SMP**AB`): no keyword rule can know the real shop. These go to AI or Uncategorised; user corrections will teach the app over time.
- **GrabPay:** unclear how GrabPay-funded purchases appear in SMS. Not covered by rules yet; revisit when a real sample arrives.
- **UOB declined transactions:** no sample. Skipped; they'll land in the review queue if they ever arrive.
- **Email alerts (Phase 3):** user notes few card transactions arrive by email, so this is low priority; may drop it.

## Decisions

- 2026-11-08: Raw / unredacted personal data never goes in the repo. Samples are **synthetic look-alikes**: real wording, fake amounts/dates/cards/merchants. Anything sensitive kept locally goes in `private/` (ignored by git).
- 2026-11-08: **Refunds** are stored as an Expense with a **negative amount**, same merchant, matched to the original purchase where possible so it inherits that category.
- 2026-11-08: In fixtures, **money is a string** (`"48.20"`) so tests build exact `Decimal` values, and **dates are ISO** (`2026-03-05`) to avoid day/month mix-ups.
- 2026-11-08: Keyword rules are **ordered, first match wins**; short keywords match whole words only.
