# Fundraise Reconciliation SOP

**Purpose:** Operating manual for running a multi-channel personal fundraise with end-to-end audit-grade reconciliation. The agent (Cowork) executes every procedure inline; there are no standalone scripts.

**Roles**

| Role | Definition |
|---|---|
| **Fundraiser** | The person running the fundraise. Owns the contributor list, sends updates, takes accountability for the published numbers. |
| **Beneficiary** | The person or family the funds are being raised for. Owns the receiving bank accounts. |
| **Intermediaries** | Trusted people who help collect or forward funds. They may hold money temporarily before consolidation. |
| **Contributor / donor** | Anyone who sends money. |
| **Recipient** | Anyone on the daily-update distribution list. |
| **Primary agent** | The Cowork agent that does the day-to-day reconciliation: receives screenshots, imports bank statements, updates Excel, generates the daily message. |
| **Audit agent** | A separate Cowork agent run on demand or after each send. It does not trust anything the primary agent did. It re-reads this SOP, re-reads the Excel and the bank-stmt PDFs, re-derives every total, and either approves the published file or files a discrepancy report. The safety net. |

---

## 1. North-Star

> **The fundraiser is publicly accountable only for what the beneficiary's bank statements show. Everything else is honest disclosure of pending claims. A single ₹100 discrepancy that a donor or third party catches is a trust failure, even if technically explainable.**

---

## 2. The trust hierarchy

| Source | What it proves | Treatment |
|---|---|---|
| Bank statement PDF | Money is provably in a known account | **Truth** |
| Sender screenshot | Sender attempted a transfer | Claim — pending bank-stmt match |
| Sender SMS notification | Sender's bank acknowledged the attempt | Claim — pending bank-stmt match |
| Verbal / WhatsApp / "I paid" | Sender says they paid | Claim — pending bank-stmt match |
| Forwarder testimony | A third party vouches | Claim — pending bank-stmt match |

Only the first row inflates "verified received". Everything else inflates "pending verification" — visible but tagged.

---

## 3. The four-state recon matrix

| Donor evidence | Bank evidence | State | Trust |
|---|---|---|---|
| Yes | Yes (refs match) | **VERIFIED** | High — publish as verified |
| Yes | Yes (refs disagree) | **MISMATCH** | Investigate before publishing |
| Yes | No | **CLAIMED-PENDING** | Show as pending; wait for next stmt |
| No | Yes | **STMT-UNIDENTIFIED** | Show in publication with bank-side hints |
| No | No | (n/a) | Nothing to track |

Plus bookkeeping states:
- **INTERNAL** — beneficiary's own movements, bank charges
- **NON-FUNDRAISE** — confirmed unrelated credits

Default state for any uncategorised bank-stmt line is **UNREVIEWED** — surfaces in Action_Items.

---

## 4. Ask before assume

When any input is ambiguous, the agent asks the fundraiser before recording. Never infer, guess, or silently drop.

| Situation | Ask |
|---|---|
| Screenshot does not show recipient UPI | "Which account did this go to — beneficiary primary, secondary, or the intermediary pool?" |
| Sender name in chat differs from screenshot | "The screenshot says [X], you said [Y] — same person?" |
| Donor pledge amount unknown | "Has this donor pledged? What amount?" |
| Bank-stmt credit has no obvious match | "Do you recognise this sender? Unidentified or do you have context?" |
| Forwarder mentions a donor with no evidence | "Confirmed received or expected?" |
| Banking name on stmt diverges from contributor name | "[Banking name] credited ₹X. Same person as [contributor name] (#N)?" |

Questions are short, specific, binary where possible. Record what is known, flag the rest, continue.

---

## 5. Architecture — three layers

```
LAYER 1 — Raw evidence (read-only, never edited)
  payments_log.md         — every donor-side input chronologically
  bank_stmts/<date>/*.pdf  — original bank statements

LAYER 2 — Source of truth (Excel)
  Bank sheets — 1:1 mirror of bank stmt PDFs
    (one sheet per bank account in scope; new sheets are
    created automatically when a new bank channel is introduced)
  Master_List — one row per contributor
  Distribution_List — recipients of the daily message + per-send status
  Unmapped_Credits — FILTER view over bank sheets
  Audit_Log — append-only
  Action_Items — derived queue
  Dashboard — one-glance summary

LAYER 3 — Publications (generated, never typed)
  daily_updates/YYYYMMDD_update.md
  Final ledger at fundraise close
```

### 5.1 Bank-sheet columns

| Col | Field | Notes |
|---|---|---|
| A | Date | DD-MM-YYYY |
| B | Time | HH:MM. Populated only if the source statement prints time. PNB statements are date-only (no time field); their `Time` cells stay blank by design. Canara e-passbook prints time; populated from PDF. Holding times come from contributor screenshots. We do NOT backfill PNB time from screenshots — bank-sheet rows reflect the bank's truth. |
| C | Amount | numeric |
| D | Type | CR / DR |
| E | Sender (bank-name) | as the bank shows it |
| F | UTR / UPI Ref | unique transaction reference |
| G | Mapped_To_# | Master_List # if matched; else blank |
| H | Status | Matched / Unidentified / Internal / Non-Fundraise / Unreviewed / Mismatch |
| I | Source | always "Stmt" |
| J | Linked Name | =VLOOKUP(G, Master_List!A:B, 2, FALSE) |
| K | Notes | edge cases |

### 5.2 Master_List columns

| Col | Field | Notes |
|---|---|---|
| A | # | sequence |
| B | Name | display name |
| C | Batch / Group | identity tag |
| D | Pledged_INR | what they said they would give |
| E | Received_Confirmed | =SUMIFS from bank sheets where Mapped_To_# = this A |
| F | Received_Pending | manual; sum of pending claims |
| G | Total_Claimed | =E + F |
| H | Payment Status | OK / PARTIAL / PENDING / OVER / REMOVED |
| I | Verification Status | Verified / Pending Verification / No Evidence |
| J | Bank_Refs | concatenated UTRs/Refs feeding Confirmed |
| K | Pending_Refs | concatenated screenshot/SMS evidence feeding Pending |
| L | Notes | narrative / audit trail |

### 5.3 Distribution_List columns

| Col | Field | Notes |
|---|---|---|
| A | # | sequence |
| B | Display Name | how the recipient is shown in WhatsApp (saved contact name) |
| C | Phone | if the recipient is not saved as a contact (used for deep-link send) |
| D | Tier | "Existing" / "Added <date>" / "Group" / "External" |
| E | Master_List Link | the contributor's # if the recipient is also a contributor; blank otherwise |
| F | Notes | identity context, alternate names, group hints |
| G | Last Sent Date | YYYY-MM-DD of the last successful send |
| H | Last Send Status | msg-check / msg-dblcheck / failed / pending — from WhatsApp delivery icon |
| I | Removed | TRUE if the recipient has been removed from the list (kept for audit, not sent to) |

New recipients are added when:
- The fundraiser names a new person who should be on the list
- A contributor is added to Master_List whose contact is known
- A forwarder relays a message and wants future updates

### 5.4 Audit_Log columns

| Col | Field |
|---|---|
| A | Timestamp (ISO 8601) |
| B | Master_List # affected |
| C | Sheet name |
| D | Row # |
| E | Field |
| F | Before |
| G | After |
| H | Trigger |
| I | Evidence ref |

Append-only. Never edited.

---

## 6. Daily triggers and processing

### 6.1 Trigger A — Donor screenshot / SMS / claim arrives

1. Inspect input. If anything ambiguous → ask (§4). Do not proceed on ambiguity.
2. Append to `payments_log.md`: `YYYY-MM-DD HH:MM | sender | amount | channel | ref | summary`.
3. Find or create the Master_List entry. If new, assign next `#`, fill Name, Batch, Pledged.
4. Update Master_List:
   - `Received_Pending += amount`
   - Append to `Pending_Refs`: `Screenshot · <channel> · <ref> · <date> <time>`
5. Bank sheets — **DO NOT TOUCH**.
6. Append Audit_Log entry: `ts | # | Master_List | <row> | Received_Pending | <old> | <new> | Donor screenshot | <ref>`.
7. Report new totals + row that changed.

### 6.2 Trigger B — Bank statement PDF arrives

1. File the PDF under `bank_stmts/<YYYY-MM-DD>/<bank>.pdf`.
2. Take Excel backup (Procedure §8.5).
3. Run **Parse bank statement** (Procedure §8.3).
4. For each parsed row, run **Import bank statement** logic (Procedure §8.4).
5. Append Audit_Log entries for every change.
6. Run **Generate daily message** (Procedure §8.2).
7. Run **Verification** (Procedure §8.1) — must PASS.
8. Report: matched / newly-unmatched / non-fundraise / mismatches.

### 6.3 Trigger C — Identification of an unmapped credit

1. Find the unmapped bank-sheet row by Ref or sender hint. If not unique, ask.
2. If donor is new → create Master_List entry first.
3. Update bank-sheet row: `Mapped_To_# = <donor #>`, `Status = Matched`.
4. Master_List `Received_Confirmed` auto-updates; append Ref to `Bank_Refs`.
5. Append Audit_Log entry.

### 6.4 Trigger D — Reconciliation query

1. Investigate against Master_List + bank sheets.
2. Reply with full lineage.
3. Never change data silently — ask before any fix.

### 6.5 Trigger E — Schema or process change

1. Excel backup (Procedure §8.5).
2. Document change in SESSION_CONTEXT.
3. Apply migration inline.
4. Re-run Verification (§8.1).
5. Report before/after summary.

---

## 7. Lineage — pick any row, get full history

### 7.1 From a bank-sheet row → contributor

1. Read `Mapped_To_#` → jump to Master_List row.
2. Master_List Notes explain the story.
3. Audit_Log filtered on this `#` shows every change.

### 7.2 From a Master_List entry → bank evidence

1. `Bank_Refs` lists every UTR/Ref feeding `Received_Confirmed`.
2. `Pending_Refs` lists every claim feeding `Received_Pending`.
3. Audit_Log filtered on `#` shows chronological history.

### 7.3 Ctrl-F audit

Every UTR/Ref appears in exactly two places: one bank-sheet Ref column AND the matched donor's `Bank_Refs`. Ctrl-F any Ref surfaces both — they must be consistent.

### 7.4 Many bank rows → one donor

Native support. Each row's `Mapped_To_#` points to the same donor. `Received_Confirmed` is a SUMIFS across all such rows. `Bank_Refs` concatenates all refs.

---

## 8. Procedures (the agent does each of these in plain steps)

Five procedures. The agent knows the technical details; the SOP only says what to do. Every procedure is deterministic — same inputs, same outputs, every time.

### 8.1 Verify the daily message before sending

**Why:** Catch any drift between Excel and the message before donors see it.

**What the agent does:**

1. Reopen the Excel so all formulas recompute (the workbook may have been edited and the cached values can be stale).
2. From Excel, add up three numbers:
   - Money confirmed in bank statements and tagged to a named contributor.
   - Money where a donor sent us a screenshot or SMS but the bank statement has not caught up yet.
   - Money in our bank statements that we cannot yet attribute to anyone.
3. From the daily message, read the headline total and add up every line item.
4. Check three things:
   - The headline total in the message matches the sum of the line items below it, to the rupee.
   - The headline total matches the three Excel sums added together, to the rupee.
   - No line uses a `K` or `L` shortcut for an amount that has a non-round remainder (so `25K` for ₹25,000 is fine, but `1.00L` for ₹1,00,100 is not).
5. If all three checks pass, write an entry in the Audit_Log saying "Verification PASS at <time>" and report success.
6. If any check fails, report the exact failure and which line caused it. **Do not send.**

### 8.2 Generate the daily message from Excel

**Why:** No amount in any donor-facing message is ever hand-typed. The message is built straight from the source of truth.

**What the agent does:**

1. Reopen Excel so all formulas recompute.
2. Read every active contributor from Master_List (skip ones marked REMOVED).
3. Read every bank-statement row whose Status is `Unreviewed` or has no contributor mapping (excluding `Internal` and `Non-Fundraise`) — these are real credits in the bank but with no name attached.
4. Group everything into three buckets in this order:
   - **Verified + named** — every contributor whose money is in the bank and tagged to them.
   - **Awaiting latest account statement for reconciliation** — contributors who have sent us a screenshot/SMS but whose money has not yet shown up in the bank statement. (Per-line label: `(Awaiting latest acc statement for recon)`. Avoid the older shorthand `(pending stmt)` which read as ambiguous.)
   - **Awaiting identification** — bank credits with no name; each line shows the bank, date, sender as the bank wrote it, and the UPI/UTR hint so the donor can recognise themselves.
5. Write the markdown file with the headline total at the top, a one-line summary of the three buckets, and the numbered list below.
6. Format rules:
   - Amounts shown as exact rupees with commas (`1,00,100`, never `1.00L`).
   - Round-number `K` (like `25K`) is allowed only when the actual amount is an exact thousand multiple.
   - Any UPI handle that ends in a full mobile number is masked to last four digits (`98XXXX2785@ptye`).
7. **Header timestamp rule:**
   - Default: `Updated <next-day> 0800 hrs` for the scheduled morning send.
   - For ad-hoc / test sends earlier (e.g. an evening preview to an intermediary or co-organiser), change the timestamp to the actual send time (e.g. `Updated May 21, 2130 hrs`) so the recipient is not confused about when the snapshot was taken.
8. **M1 transparency footer (mandatory):** include the line `Claude (AI) is helping me reconcile and send these messages. AI can make mistakes — please highlight if you see anything that missed both of us.` at the bottom of M1. The fundraiser is owning the send; the AI assistance must be disclosed.
9. Run §8.1 on the freshly written file. If it fails, don't save — fix and retry.
10. Append an Audit_Log entry saying when the file was generated and what total it carried.

### 8.3 Read a bank statement PDF

**Why:** The bank statement is the source of truth. We need it parsed into structured rows we can compare against Excel.

**What the agent does:**

1. Open the PDF. If it is password-protected, use the supplied password.
2. Read every transaction line. For each line, pick out: date, time (if shown), amount, whether it is a credit or debit, sender name as the bank shows it, and the UTR/UPI reference.
3. Hand back the list of rows.

If the bank's statement format is one the agent has not seen before, the agent stops and asks the fundraiser to confirm how one or two sample lines should be read. No best-effort parsing.

### 8.4 Import a bank statement into Excel

**Why:** New bank credits need to land in the right bank sheet, matched to a contributor if possible, flagged honestly if not.

**What the agent does:**

1. File the PDF under `bank_stmts/<date>/` so we have an immutable copy.
2. Take an Excel backup (§8.5).
3. Read the PDF (§8.3).
4. **Filter to credits only (default rule).** For beneficiary bank sheets (PNB, Canara, etc.), skip every DR row — they are the beneficiary's own outflows (bank charges, self-transfers, bill payments) and have nothing to do with the fundraise. The only exception is intermediary holding accounts (e.g. Holding_Account), where DR rows representing sweeps to the beneficiary are kept (per §8.8).
5. For each remaining CR row in the PDF, decide what to do:
   - **Already in Excel?** If the same UTR/Ref already appears in the matching bank sheet, skip — we don't duplicate.
   - **New row in bank sheet.** Append it with everything we know from the PDF (date, time, amount, type=CR, sender, ref). Mark Source = "Stmt".
   - **Try to match it to a pending claim.** Look at every contributor in Master_List who has unpaid pending claims. If any of their pending evidence matches this new bank row (by UTR, or by amount + sender hint + date within three days), tag the bank row to that contributor, reduce the contributor's pending amount, and move that piece of evidence from `Pending_Refs` to `Bank_Refs`.
   - **If no match found:** if the fundraiser has previously flagged the sender as non-fundraise (e.g., a known unrelated credit), mark it Non-Fundraise. If it is a known sweep credit landing from a tracked intermediary, mark it Internal with cross-reference to the Holding sheet sweep row (per §8.8). Otherwise mark it Unreviewed and add it to Action_Items asking the fundraiser whether they recognise the sender.
6. After all rows are processed, scan Master_List for contributors who have pending claims older than 48 hours with still no match. Add each of those to Action_Items as "claim has not landed — investigate".
7. Save Excel.
8. Write an Audit_Log entry for every change made above, including the count of DR rows skipped.
9. Regenerate the daily message (§8.2).
10. Report to the fundraiser: how many CR rows were newly matched, how many are unreviewed, how many internal, how many non-fundraise, and how many DR rows were skipped.

### 8.5 Back up the Excel before any big change

**Why:** A change can break things. We always want a way back.

**What the agent does:**

1. Copy the Excel file to `archive/reconciliation_BACKUP_<timestamp>.xlsx`.
2. Tell the fundraiser where the backup lives.

The agent runs this automatically before importing a bank statement (§8.4), adding a new bank sheet (§8.6), or making any schema change (Trigger E). The fundraiser can also ask for a backup any time.

### 8.6 Send the daily message (distribution)

**Why:** Generating the message (§8.2) is not the same as delivering it. Distribution is its own step with its own integrity rules — wrong-person sends are a real failure mode, and "sent to whom on what day" is part of the audit trail.

**Channel:** WhatsApp Web, driven by the agent through the browser. For unsaved contacts, the agent uses the WhatsApp deep-link (`https://web.whatsapp.com/send?phone=<digits>`) instead of search.

**Pre-flight (one-time before the loop):**

1. Confirm the message file to send (`daily_updates/<YYYYMMDD>_update.md`) has just been verified (§8.1 PASS within the last 5 minutes).
2. Confirm the audit agent (§9.2) returned APPROVED on this file.
3. Open WhatsApp Web in the browser. Confirm the fundraiser is logged in.

**For each recipient in Distribution_List where Removed ≠ TRUE and Last Sent Date ≠ today:**

1. **Find the recipient's chat.**
   - If `Phone` is set: open the chat via the WhatsApp deep-link with that number.
   - Else: search the recipient's `Display Name` in WhatsApp.
2. **Strict-verify identity.** The chat header's "Type a message to <name>" must contain every distinct token of `Display Name` (case-insensitive). If only the first token matches or the chat appears to be a different person with the same first name, abort this recipient and add a row to Action_Items: "Verify contact for <Display Name>".
3. **Send M1, then M2, then M3** by pasting each block in order and pressing Enter.
4. **Capture delivery icon** after each send (msg-time / msg-check / msg-dblcheck). The terminal status is the icon on the last (M3) message.
5. **Update Distribution_List for this row:**
   - `Last Sent Date` = today
   - `Last Send Status` = the captured icon
6. **Append Audit_Log entry:** `ts | (Master_List Link if any) | Distribution_List | <row> | Last Send Status | <old> | <new> | Daily send | <message file path>`
7. If anything went wrong (chat not found, verify failed, paste failed, status icon missing), do not retry inside the loop — write to Action_Items and continue with the next recipient. The fundraiser handles flagged items by hand.

**After the loop:**

- Report counts: total recipients, sent successfully, skipped (Removed), failed (Action_Items rows).
- The audit agent reads `Last Sent Date == today` rows and confirms each one has a corresponding Audit_Log line and a delivery status that is `msg-check` or `msg-dblcheck`.

**Distribution list maintenance:**

- When the fundraiser adds a new recipient mid-day, the agent adds a row to Distribution_List with Tier = "Added <date>" and immediately sends to them (after the same verify gate).
- When a recipient asks to be removed, the agent sets `Removed = TRUE` (never deletes the row, for audit).

### 8.7 Add a new bank sheet (when a new bank channel appears)

**Why:** A fundraise can collect money across several accounts — beneficiary's primary bank, beneficiary's secondary bank, an intermediary holding account, an online platform like Milaap, a newly-opened account mid-fundraise. Each account needs its own sheet so it stays a 1:1 mirror of its own statements. The set of bank sheets is not fixed up front.

**When this fires:**

- A bank statement PDF arrives for an account we have never seen before.
- The fundraiser tells us money has started landing in a new account.
- An intermediary opens a new pool account.
- A new fundraise platform (Milaap-equivalent) is added.

**What the agent does:**

1. Ask the fundraiser to confirm: account holder name, bank name, account-number tail, IFSC, and the role (beneficiary primary / beneficiary secondary / intermediary holding / platform / other). No assumptions.
2. Take an Excel backup (§8.5).
3. Create a new sheet in the workbook. Name it generically using the role and bank, e.g., `PNB_Statement`, `Canara_Statement`, `Holding_Account`, `Milaap_Statement`, `HDFC_Statement_2`. Avoid person-specific names — use the role.
4. Populate the standard bank-sheet columns exactly as §5.1.
5. Update Master_List `Received_Confirmed` formula to include this new sheet in the SUMIFS so future credits flow into the right contributor's confirmed total.
6. Update the Dashboard channel-breakdown to include the new sheet's gross-CR total.
7. Document the new channel in SESSION_CONTEXT: which account, what role, when it was added, who manages it.
8. Append an Audit_Log entry recording the new sheet and the formula updates.
9. Run Verification (§8.1) — must PASS to confirm formulas did not break.

The first import into the new sheet then follows §8.4 normally.

### 8.8 Handle internal money movement (sweeps)

**Why:** Intermediaries (a trusted person who holds a pool account) collect money from contributors and periodically transfer the accumulated amount to the beneficiary's main account. This sweep is NOT a new contribution — the rupees were already counted when each contributor paid into the pool. Counting the sweep again would inflate the fundraise total.

**When this fires:**

- An intermediary sends a screenshot showing they transferred a lump sum to the beneficiary.
- The beneficiary's bank statement shows a large credit from a known intermediary account.
- The fundraiser asks "how do we handle this transfer".

**What the agent does:**

1. Confirm with the fundraiser that this is an internal sweep, not a fresh contribution. Identify the source account (intermediary) and destination account (beneficiary).
2. Log the sweep in `payments_log.md` with `Type = Sweep`. This flags it as non-contribution intake. Sweep entries are excluded from the contribution total.
3. Add a DEBIT row to the intermediary's sheet (e.g. `Holding_Account`):
   - `Type = DR`
   - `Amount = <sweep amount>`
   - `Mapped_To_# = empty` (no contributor owns a sweep)
   - `Status = Internal`
   - `Notes = "SWEEP — <source> → <destination>. <UTR/Txn ID>. NOT a contribution."`
4. When the beneficiary's bank statement arrives showing the matching credit, add the CREDIT row to that bank sheet with:
   - `Type = CR`
   - `Mapped_To_# = empty`
   - `Status = Internal`
   - `Notes = "Sweep from <intermediary>. Cross-ref UTR <ref>."`
5. Maintain a running balance on the intermediary's sheet: `Balance = SUMIFS(Amount, Type, "CR") − SUMIFS(Amount, Type, "DR")`. This tells the fundraiser exactly what is still sitting in the pool.
6. Verify after: grand total Received_Confirmed in Master_List is unchanged by the sweep (delta must be ₹0).
7. Append an Audit_Log entry recording the sweep.

**The rule that makes this safe:** the Master_List `Received_Confirmed` SUMIFS matches on `Mapped_To_# = contributor's #` AND `Type = "CR"`. A sweep DR row has no `Mapped_To_#` and is not type CR — it is automatically excluded. A sweep CR row in the destination sheet has `Mapped_To_# = empty` — it does not match any contributor.

**Never:** map a sweep row to a contributor #. Never count an Internal-status row toward the fundraise total. Never delete a sweep row to "clean up" — the audit trail must show every rupee movement.

---

## 9. Safety net — the audit agent (second pass)

The primary agent runs the day-to-day work. The audit agent exists to catch anything the primary missed. It is run on demand, and at minimum after every send, before the fundraiser walks away.

**Plain-English description of what the audit agent does:**

Imagine a careful second person who has not seen any of today's work. They sit down, read the SOP, open the Excel, open the bank-stmt PDFs from `bank_stmts/`, and re-do the math from scratch. They compare their answer to what was actually published in `daily_updates/`. If the two agree to the rupee, they sign off. If they disagree, they write a discrepancy report and stop the day.

The audit agent never edits anything. It only reads and reports.

### 9.1 When to run the audit pass

| When | Trigger |
|---|---|
| After every Generate-daily-message + Send | Mandatory before considering the day done |
| After every bank-stmt import | Recommended to confirm matching was correct |
| At any time the fundraiser asks | Ad hoc |
| At end of fundraise | Mandatory — produces the final ledger |

### 9.2 What the audit agent does, step by step

**Inputs:** the Excel file, the most recent daily message that was sent, and the folder of today's bank-stmt PDFs (if any).

**Steps:**

1. **Read this SOP again.** The audit agent does not assume any prior knowledge of today's work.
2. **Independently re-compute the three totals from Excel:**
   - Money confirmed in bank statements and tagged to a named contributor.
   - Money where a donor sent evidence but the bank statement has not caught up yet.
   - Money in bank statements that we cannot yet attribute to anyone.
3. **Independently re-read the daily message** and pick out its headline total and the sum of its line items.
4. **Run these cross-checks:**

   | Check | What it confirms | What a failure means |
   |---|---|---|
   | Headline matches sum of line items | The message is internally consistent | A line was hand-edited or rounded |
   | Headline matches Excel three-bucket total | The message reflects Excel exactly | Excel changed after the message was generated, or the generator drifted |
   | No `K` or `L` shortcut on non-round amounts | No precision was lost in publication | A donor's amount was silently rounded |
   | Every line in the message traces back to a real Excel row | No phantom or hand-added entries | Someone wrote a line that does not exist in source data |
   | Every bank-statement credit older than a day has been categorised | We are not sitting on uncategorised money | A credit slipped through review |
   | Every contributor showing a pending amount has at least one piece of evidence on file | Pending money has a screenshot or SMS behind it | A pending claim has no proof |
   | Every contributor showing a confirmed amount has at least one bank reference | Confirmed money points to a real bank row | A confirmation was inflated without evidence |
   | The Audit_Log shows a Verification PASS entry near the time the message was sent | The send went through the right gate | The send happened without verification |
   | No two bank-sheet rows share the same UTR/Ref | We did not double-count any credit | A statement was imported twice |
   | No removed contributor still shows confirmed money | We are not counting deleted rows | Cleanup was incomplete |

5. **Random-sample re-import.** Pick five rows at random from the latest bank-stmt PDF. For each, confirm it appears in Excel with matching date, amount, sender and ref. Any mismatch is a discrepancy.
6. **Write the audit report** at `archive/audit_<timestamp>.md` with one of two outcomes:
   - **APPROVED** — every check passed. The report says so, lists the three totals re-derived, and signs off.
   - **DISCREPANCY** — at least one check failed. The report lists every failing check with what was expected, what was observed, and which row caused it. The fundraiser is notified.
7. **The audit agent never edits anything.** Fixes are the primary agent's job. After any fix, the audit runs again.

### 9.3 What the audit agent does NOT do

- It does not edit Excel.
- It does not re-send the daily message.
- It does not silently fix discrepancies.
- It does not trust any audit-log entry written by the primary agent for the same check.

### 9.4 Resolution flow

1. Audit reports DISCREPANCY → fundraiser sees the report.
2. Primary agent investigates the specific check that failed (using lineage paths in §7).
3. Primary agent applies the minimum necessary fix, with backup (§8.5) and audit-log entries.
4. Audit agent re-runs §9.2. Must report APPROVED before the day is considered closed.

---

## 10. File and folder structure

```
<fundraise-folder>/
├── Fundraise_Reconciliation_Sheet.xlsx                ← single source of truth
├── RECON_SOP.md                       ← this file
├── SESSION_CONTEXT.md                 ← session restart context
├── payments_log.md                    ← raw donor-side intake (Layer 1)
├── daily_updates/
│   └── YYYYMMDD_update.md             ← generated daily messages
├── bank_stmts/
│   └── YYYY-MM-DD/
│       └── <bank>.pdf                  ← archived original statements
└── archive/
    ├── reconciliation_BACKUP_<ts>.xlsx
    └── deprecated files
```

No `.py` files at the top level. All logic is in §8 of this SOP.

---

## 11. Workflow checklist for a typical day

Morning (primary agent):
1. Open Excel and SESSION_CONTEXT.
2. If bank stmt PDFs arrived → run **Trigger B** (procedures §8.3–§8.4).
3. For each row marked Unreviewed → ask the fundraiser (§4).
4. Resolve any Mismatch flags in Action_Items.
5. Run **Generate daily message** (§8.2).
6. Run **Verification** (§8.1) — must PASS.
7. **Run the audit agent (§9.2). Must report APPROVED.**
8. Stop here. **Do not auto-send.** The fundraiser inspects the generated message and then explicitly says "send it" before the agent runs **§8.6 Send the daily message**. Distribution is never automatic.

Through the day (primary agent):
9. New donor screenshot in chat → **Trigger A** (§6.1). Ask if anything ambiguous.
10. New contributor reply identifying an unmapped row → **Trigger C** (§6.3).
11. Reconciliation query in chat → **Trigger D** (§6.4). No silent changes.

End of day:
12. Primary runs **Verification** (§8.1) once more on the published file.
13. Audit agent runs §9.2 over the published file and Excel. Either APPROVED → day closed, or DISCREPANCY → fix and re-audit (§9.4).
14. Append the day's summary to SESSION_CONTEXT incident log if anything notable happened.

---

## 12. Failure modes seen so far (learn from these)

| Type | Failure | Root cause | Rule strengthened |
|---|---|---|---|
| Rounding | Daily message sent with ₹400 visible-sum gap vs claimed headline | `K`/`L` abbreviation hid sub-lakh remainders | R1 (exact amounts), R2 (header = sum), enforced by §8.1 |
| Recipient mix-up | Donor message bundle sent to a stranger with the same first name | Recipient search used first-name token only | Strict-verify by ALL name tokens; use direct deep-link for unsaved numbers |
| Phantom credits | A contributor had bank rows in Excel that never appeared in any bank statement | Screenshots were added directly to bank sheets | Screenshots update Master_List only; bank sheets are 1:1 with PDFs (§6.1 step 5) |
| Assumed identity | Banking name on stmt differed from contributor name; recorded as same person without asking | No "ask before assume" gate | §4 requires asking when names diverge significantly |
| In-sheet TOTAL rows | Naive row-sum scripts treated the in-sheet TOTAL row as another contributor, doubling the grand total during audit | TOTAL row coexisted with data rows on the same sheet | Non-negotiable: NO total/subtotal/sum rows live inside data sheets. All totals live only in `Dashboard` |

---

## 13. Non-negotiables

1. Bank sheets are touched ONLY by bank stmt PDFs (and identification mapping changes per Trigger C).
2. Screenshots update only Master_List `Received_Pending` + `Pending_Refs` + `payments_log.md`.
3. No amount in any public message is hand-typed — always generated by §8.2.
4. **Verification §8.1 must PASS before any send.** No exceptions.
5. **Audit agent §9.2 must report APPROVED before the day is closed.** No exceptions.
6. Audit_Log is append-only. Never edited, never deleted.
7. Source-of-truth precedence: bank stmt > screenshot > SMS > verbal claim.
8. The headline "Total raised" never exceeds bank-sheet credits + intermediary pools + platform balances. Anything above is in "pending" disclosure, never claimed.
9. Ambiguity is resolved by asking the fundraiser, not by silent inference.
10. The audit agent never edits anything. It only reads and reports.
11. **No total/subtotal/sum rows live inside data sheets** (Master_List, bank sheets, Holding_Account, etc.). Each data sheet is data-only — one row per real entry, nothing else. All aggregations live on the `Dashboard` sheet or as separate header cells (e.g. Holding Balance) clearly labelled. This prevents naive row-sum scripts from double-counting and keeps `max_row` honest.
12. **AI assistance is disclosed in every public message.** The M1 footer carries the standing line that an AI (Claude) is helping reconcile and draft messages, and invites recipients to flag mistakes. The fundraiser is accountable; the AI is acknowledged as a tool, not hidden.

---

*This document is intended to be packaged as a re-usable Claude skill so future fundraises follow the same trust contract. All procedures in §8 and §9 are deterministic — given the same inputs, the agent produces the same outputs every time. The audit agent in §9 is the safety net: an independent second pass that never trusts the primary agent's prior work.*
