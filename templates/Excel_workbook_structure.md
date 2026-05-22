# Fundraise_Reconciliation_Sheet.xlsx — workbook structure

This document specifies the structure of the Excel workbook that is the single source of truth (SOP Layer 2). A `.xlsx` template builder is a planned follow-up — for now this markdown is the authoritative spec, and a fresh workbook can be created by hand or by an agent following these column definitions.

The workbook holds these sheets:

1. `Master_List` — one row per contributor
2. One or more `Bank sheets` — 1:1 mirror of each bank account's statement
3. `Holding_Account` — for an intermediary pool (optional)
4. `Distribution_List` — recipients of the daily message
5. `Audit_Log` — append-only change log
6. `Action_Items` — derived queue of things needing the fundraiser's attention
7. `Dashboard` — the ONLY place totals live

Non-negotiable: no total / subtotal / sum row sits inside any data sheet. All aggregations live on `Dashboard`. Violating this once doubled the grand total during a self-audit.

---

## Sheet 1 — `Master_List`

One row per contributor.

| Col | Field | Notes |
|---|---|---|
| A | # | sequence number, primary key |
| B | Name | display name |
| C | Batch / Group | identity tag (school batch, work circle, family, etc.) |
| D | Pledged_INR | what they said they would give |
| E | Received_Confirmed | `=SUMIFS` across every bank sheet where `Mapped_To_# = this A` |
| F | Received_Pending | manual; sum of pending claims (screenshots / SMS not yet matched to bank) |
| G | Total_Claimed | `=E + F` |
| H | Payment Status | OK / PARTIAL / PENDING / OVER / REMOVED |
| I | Verification Status | Verified / Pending Verification / No Evidence |
| J | Bank_Refs | concatenated UTRs / Refs that feed `Received_Confirmed` |
| K | Pending_Refs | concatenated screenshot / SMS evidence that feeds `Received_Pending` |
| L | Notes | narrative / audit trail |

---

## Sheets 2+ — `<Bank>_Statement`

One sheet per bank account in scope. Each is a 1:1 mirror of that account's statement PDF.

| Col | Field | Notes |
|---|---|---|
| A | Date | DD-MM-YYYY |
| B | Time | HH:MM — populated only if the source statement prints time; blank otherwise |
| C | Amount | numeric |
| D | Type | CR / DR |
| E | Sender (bank-name) | as the bank shows it |
| F | UTR / UPI Ref | unique transaction reference |
| G | Mapped_To_# | `Master_List` `#` if matched; else blank |
| H | Status | Matched / Unidentified / Internal / Non-Fundraise / Unreviewed / Mismatch |
| I | Source | always `Stmt` |
| J | Linked Name | `=VLOOKUP(G, Master_List!A:B, 2, FALSE)` |
| K | Notes | edge cases |

Touch this sheet only when a new bank statement arrives. Never type into it from screenshots — screenshots go to `Master_List` pending only.

---

## Sheet — `Holding_Account` (optional)

For an intermediary who collects contributions into a pool and periodically sweeps to the beneficiary's main account. Same column structure as a bank sheet, plus a running balance.

`Balance = SUMIFS(Amount, Type, "CR") − SUMIFS(Amount, Type, "DR")` (placed in a clearly-labelled header cell on the sheet, NOT a row inside the data).

Sweep DR rows have `Mapped_To_# = empty`, `Status = Internal`, and a note describing the sweep destination. The matching CR row on the beneficiary's bank sheet is also `Status = Internal`. Neither inflates the fundraise total.

---

## Sheet — `Distribution_List`

| Col | Field | Notes |
|---|---|---|
| A | # | sequence |
| B | Display Name | how the recipient is shown in WhatsApp (saved contact name) |
| C | Phone | for unsaved contacts (used for `wa.me` deep-link) |
| D | Tier | "Existing" / "Added <date>" / "Group" / "External" |
| E | Master_List Link | the contributor's `#` if the recipient is also a contributor; blank otherwise |
| F | Notes | identity context, alternate names, group hints |
| G | Last Sent Date | YYYY-MM-DD of the last successful send |
| H | Last Send Status | msg-check / msg-dblcheck / failed / pending (from WhatsApp delivery icon) |
| I | Removed | TRUE if removed from the list (kept for audit, never sent to) |

---

## Sheet — `Audit_Log`

Append-only. Never edited, never deleted.

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

---

## Sheet — `Action_Items`

Derived queue of things needing the fundraiser's attention: unreviewed bank rows, stale pending claims older than 48 hours, mismatches, recipients flagged on send.

| Col | Field |
|---|---|
| A | # | sequence |
| B | Created | timestamp |
| C | Type | Unreviewed / Stale-Pending / Mismatch / Send-Failed / Other |
| D | Pointer | sheet name + row # |
| E | Summary | one line |
| F | Status | Open / Resolved |
| G | Resolved by | reference to fix |

---

## Sheet — `Dashboard`

The ONLY place totals live.

Includes at minimum:

- Grand total raised (sum of all `Received_Confirmed` from `Master_List`).
- Three-bucket headline: Verified + Pending + Unidentified.
- Per-channel breakdown (one cell per bank sheet's gross CR total).
- Count of contributors by `Payment Status`.
- Open `Action_Items` count.

The agent reads from this sheet (and only this sheet) when generating the daily message headline.
