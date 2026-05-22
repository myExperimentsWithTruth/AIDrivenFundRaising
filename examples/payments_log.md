# payments_log.md — Layer 1 raw intake (synthetic example)

This file is append-only. Every donor-side event lands here chronologically before any of it touches the workbook. Bank statement events also get a `Stmt` line here for audit completeness.

Format per line: `YYYY-MM-DD HH:MM | sender | amount | channel | ref | summary`

---

```
2026-05-18 11:20 | Gaurav S          | ₹5,000   | Holding   | HOLD/H001              | Paid into intermediary pool. Pantnagar-1998. WhatsApp screenshot received.
2026-05-18 13:45 | Farhan A          | ₹20,000  | Milaap    | MLP/MILAAP-EXT-9921    | Overseas; via Milaap, routed into Holding. Confirmation email from Milaap.
2026-05-19 08:14 | Anand S           | ₹50,000  | PNB-stmt  | UPI/2026051912345      | Stmt · PNB · CR · matched to Master_List #1.
2026-05-19 08:14 | Bhavna K          | ₹25,000  | PNB-stmt  | UPI/2026051923456      | Stmt · PNB · CR · matched to Master_List #2.
2026-05-19 08:14 | Chetan M          | ₹100,000 | Canara-stmt | UPI/2026051967890    | Stmt · Canara · CR · matched to Master_List #3.
2026-05-20 09:02 | Deepa R           | ₹10,000  | WhatsApp  | screenshot 21May 14:20 | Claim only — screenshot received, bank not yet caught up. Family-friend.
2026-05-20 09:35 | (unknown — Ravi K)| ₹15,000  | PNB-stmt  | UPI/2026052034567      | Stmt · PNB · CR · sender RAVI KUMAR not yet on Master_List. Action_Items raised.
2026-05-20 09:36 | Ishan T           | ₹50,000  | PNB-stmt  | UPI/2026052045678      | Stmt · PNB · CR · matched to Master_List #9.
2026-05-20 10:15 | Jaya M            | ₹25,000  | Canara-stmt | UPI/2026052078901    | Stmt · Canara · CR · matched to Master_List #10.
2026-05-20 16:42 | Holding sweep     | ₹25,000  | Holding→Canara | SWP/HOLD-CAN-001  | Sweep · NOT a contribution. Holding DR row + Canara CR row, both Status=Internal.
2026-05-21 14:20 | Eshan P (claim 1) | ₹10,000  | PNB-stmt  | UPI/2026052156789      | Stmt · PNB · CR · matched to Master_List #5. First instalment.
2026-05-22 09:10 | Eshan P (claim 2) | ₹5,000   | WhatsApp  | screenshot 22May 09:10 | Claim — second instalment screenshot received. Awaiting next PNB stmt.
2026-05-21 19:42 | Hema V            | ₹30,000  | WhatsApp  | screenshot 21May 19:42 | Claim only — awaiting next PNB statement to confirm.
```

---

## Rules

- Never edit a line once written.
- Bank-statement events are echoed here with `channel = <bank>-stmt` so the file is a complete chronology — but the bank sheet remains the source of truth, not this file.
- Use `Stmt` in the summary for events sourced from a bank statement.
- Use `Claim` for events sourced from donor-side evidence only (screenshot, SMS, verbal).
- Use `Sweep` for intermediary movements.
- Anything ambiguous goes here AND triggers an Action_Items row in the workbook.
