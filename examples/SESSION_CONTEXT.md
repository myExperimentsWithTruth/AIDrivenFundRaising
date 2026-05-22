# SESSION_CONTEXT.md — agent restart brief (synthetic example)

This file is what a new Cowork or Claude Code session reads first to pick up where the previous session left off. It is not the SOP — the SOP is fixed. This file captures the live state of the fundraise.

---

## Beneficiary / fundraise

- **Beneficiary:** synthetic-family.
- **Started:** 18-May-2026.
- **Status:** active. Target not declared; pledges drive cadence.

## Channels in scope

| Channel | Role | Sheet | Notes |
|---|---|---|---|
| PNB primary account | Beneficiary primary | `PNB_Statement` | Statement format: date-only, no time field. Password-protected; password held separately. |
| Canara secondary account | Beneficiary secondary | `Canara_Statement` | E-passbook PDF; includes time. |
| Intermediary pool | Holding | `Holding_Account` | Run by a trusted batchmate. Sweeps to Canara every 2–3 days. |
| Milaap campaign | Platform | (incoming) | Overseas contributors. Inflows arrive into Holding as a single lump per day. |

## What the day looks like

1. New donor screenshots / SMS arrive on WhatsApp through the day → run Trigger A (SOP §6.1).
2. Bank statement PDFs land each morning (PNB by 0700 IST, Canara by 0830 IST) → run Trigger B (SOP §6.2).
3. Generate the daily message at 0800 IST (SOP §8.2). Verify (§8.1). Audit (§9.2). Send (§8.6).
4. Through the day: identify new bank-side senders, resolve mismatches, add new recipients.
5. End-of-day audit closes the day (SOP §11 step 13).

## Open Action_Items (snapshot)

1. **PNB row 4 — RAVI KUMAR · ₹15,000 · 20-May.** Sender not on Master_List. Asked the fundraiser; reply pending.
2. **Master_List #8 — Hema V · ₹30,000 pending > 48 hours.** Screenshot received 21-May 19:42; not yet in any bank statement. Investigate before next daily send.
3. **Resolved 22-May:** new forwarder added (Master_List link blank, Distribution_List #11). Send confirmed.

## Recent incidents to remember

- **21-May ₹400 visible-sum gap.** Caused by a `1.00L` shortcut in the daily message for an amount that was actually ₹1,00,100. Rule R1 (exact amounts) and R2 (header = sum of lines) added to verification §8.1. Don't regress.
- **21-May recipient mix-up risk.** Distribution_List recipient #12 has the same first name as an unrelated saved contact. Strict-verify by ALL name tokens added to §8.6. Use the `wa.me/send?phone=` deep-link path whenever a phone number is known.

## Schema changes since fundraise start

| Date | Change | Reason |
|---|---|---|
| 18-May-2026 | Added `Holding_Account` sheet | Intermediary started collecting pool contributions. |
| 19-May-2026 | Removed in-sheet TOTAL rows from `Master_List` and bank sheets | Naive row-sum doubled the grand total during a self-audit. All aggregations now live on `Dashboard` only. |
| 21-May-2026 | Added `Phone` column to `Distribution_List` | Switched WhatsApp send strategy from name-search to deep-link. |

## Conventions (project-specific)

- Currency: rupees, with thousands separators (`₹1,00,100`). `K` allowed only for exact thousands; `L` not allowed.
- Time zone: IST. Bank-stmt timestamps stay as the bank prints them.
- Daily message filename: `daily_updates/YYYYMMDD_update.md`.
- Audit report filename: `archive/audit_YYYYMMDD_HHMMSS.md`.

---

The SOP at `RECON_SOP.md` is canon. If this context file ever disagrees with the SOP, the SOP wins.
