# Daily fundraise update — template

The daily message is GENERATED, never typed. The agent builds it from Excel by following SOP §8.2. This file shows the shape only.

The actual file is named `daily_updates/YYYYMMDD_update.md` and is composed of three blocks (M1, M2, M3) that are pasted sequentially into each WhatsApp chat.

---

## M1 — Headline + summary

```
Updated <next-day> 0800 hrs

Friends, gentle update on the fundraise for <one-line beneficiary context>.

Total received so far: ₹<grand total in rupees with commas>

  • Verified in bank statements (named contributors): ₹<bucket A>
  • Awaiting latest acc statement for recon (donor evidence in, bank statement not yet): ₹<bucket B>
  • Awaiting identification (bank credits without a name attached): ₹<bucket C>

If you have given and don't see your entry yet, please reply with a screenshot — we will reconcile it in the next pass.

Claude (AI) is helping me reconcile and send these messages. AI can make mistakes — please highlight if you see anything that missed both of us.
```

## M2 — Verified contributors (numbered list)

```
Verified received (named, traced to bank statement):

1. <Name>, <Batch/Group>, ₹<exact rupees with commas>
2. <Name>, <Batch/Group>, ₹<exact rupees with commas>
3. ...

Awaiting latest acc statement for recon:

A. <Name>, <Batch/Group>, ₹<exact rupees with commas>
B. ...
```

## M3 — Unidentified credits (so donors can claim themselves)

```
Awaiting identification — these credits hit our bank but we don't yet have a name:

I. <Bank>, <date>, sender as bank wrote it: <BANK-SENDER-NAME>, ref <UTR>, ₹<amount>
II. ...

If any of these is yours, please reply with your name and the channel you sent from — we'll tag it in the next update.
```

---

## Format rules (enforced by §8.1 verification)

- Amounts are exact rupees with thousands separators (`₹1,00,100`, never `₹1.00L`).
- `K` shortcut allowed only for exact thousands (`₹25K` for ₹25,000 is fine; `₹25.5K` for ₹25,500 is not).
- Any UPI handle ending in a full mobile number is masked to last four digits (`98XXXX2785@ptye`).
- M1's headline ₹ total must equal the sum of every line item in M2 + M3, to the rupee.
- M1's headline ₹ total must equal the Excel three-bucket sum, to the rupee.
- M1 carries the AI-disclosure footer. Non-negotiable.
