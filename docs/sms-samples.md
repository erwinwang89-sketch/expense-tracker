# Bank SMS samples (synthetic)

These are **fake look-alikes** of real bank SMS alerts. Each one copies the exact
wording, punctuation and layout of a real message, but the amounts, dates, card
digits and merchant names are made up. No real SMS content is stored in this repo
(see CLAUDE.md §8).

The parser is built and tested against these formats. When a bank changes its
wording, add a new sample here instead of editing the old one, because old phones
may still send the old format.

For each bank: the template, the fake samples, and notes on what's tricky.

---

## UOB

### Card transaction

**Template**

```
A transaction of {CURRENCY} {AMOUNT} was made with your UOB Card ending {LAST4} on {DD/MM/YY} at {MERCHANT}. If unauthorised, call 24/7 Fraud Hotline
```

**Samples**

| ID | Message |
|---|---|
| uob-card-001 | A transaction of SGD 48.20 was made with your UOB Card ending 1234 on 05/03/26 at Cafe Lumiere. If unauthorised, call 24/7 Fraud Hotline |
| uob-card-002 | A transaction of SGD 125.00 was made with your UOB Card ending 9876 on 21/03/26 at SMP**AB. If unauthorised, call 24/7 Fraud Hotline |
| uob-card-003 | A transaction of SGD 2,345.80 was made with your UOB Card ending 1234 on 14/02/26 at HOTELSITE.COM HOTEL SUNRISE. If unauthorised, call 24/7 Fraud Hotline |
| uob-card-004 | A transaction of HKD 1,856.30 was made with your UOB Card ending 5555 on 09/01/26 at HARBOUR VIEW HTL HK,KOWLO. If unauthorised, call 24/7 Fraud Hotline now |

**Parsing notes**

- **Date is DD/MM/YY**, day first (Singapore style). `05/03/26` = 5 March 2026, not 3 May. There's **no time** in the message, so use the time the SMS arrived.
- **Amounts of 1,000 or more have a thousands comma** (`2,345.80`). Remove the comma before converting to a number.
- **Currency is any 3-letter code** (`SGD`, `HKD`, …), not just SGD. Store the original currency and amount; convert to SGD only when a rate is available, and mark it as an estimate (CLAUDE.md §4).
- **The ending varies**: sometimes `…Fraud Hotline`, sometimes `…Fraud Hotline now`. Don't require the message to end at a fixed point; anchoring on `. If unauthorised` is enough.
- **Merchant can contain commas** (`HARBOUR VIEW HTL HK,KOWLO`) and is often cut off (the bank truncates long names).
- **Merchant ends just before `. If unauthorised`.** Match up to that phrase, not up to the first full stop, because merchant names can contain dots (`HOTELSITE.COM HOTEL SUNRISE`).
- **Merchant can be a payment-processor code** such as `SMP**AB`. The real merchant name is hidden behind the processor's prefix. These are hard to categorise by keyword, so expect them to fall through to AI or "Uncategorised".
- The same phone can get alerts for **more than one UOB card**. Keep the last 4 digits as a label (e.g. "UOB ••1234") so the user can tell the cards apart.

### Refund

**Template**

```
A refund of {CURRENCY}{AMOUNT} from {MERCHANT} has been made to your UOB card ending {LAST4} on {DD Mon YYYY}. The refund will be posted in your next statement
```

**Samples**

| ID | Message |
|---|---|
| uob-refund-001 | A refund of SGD23.40 from SHOPMART.COM has been made to your UOB card ending 1234 on 12 Mar 2026. The refund will be posted in your next statement |

**Parsing notes**

- **No space between currency and amount** (`SGD23.40`), unlike card transactions. Allow an optional space: `SGD ?`.
- **Different date format:** `12 Mar 2026` (day, short month name, 4-digit year).
- **Lowercase "card"** here vs "Card" in transactions. Match case-insensitively.
- Merchant sits between `from` and ` has been made`.
- **A refund is money coming back, not spending.** Store it as an Expense with a **negative amount** (e.g. `-23.40`), and try to match it to the original purchase so it inherits that category.

### Not covered

- Declined transactions: no sample yet (skipped for now). If one arrives, the parser won't recognise it and it becomes a needs-review entry, so add it here then.
