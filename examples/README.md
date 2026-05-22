# examples/

Synthetic, end-to-end snapshot of what a live fundraise looks like in this system. Every name, amount, UTR, and reference here is invented. Use this folder to see the shape of the workflow before adapting it to your own.

## Files

| File | What it shows |
|---|---|
| `Fundraise_Reconciliation_Sheet.xlsx` | The full workbook with 10 fake contributors, three bank channels (PNB, Canara, Holding), a sweep that does not double-count, a Distribution_List of 15 recipients, the Audit_Log, Action_Items, and the Dashboard that aggregates everything. |
| `payments_log.md` | Layer 1 raw intake — chronological list of donor screenshots and bank events as they arrived. |
| `SESSION_CONTEXT.md` | What a new agent reads first to pick up the live state of the fundraise. |
| `daily_updates/20260521_update.md` | A complete daily message in M1 + M2 + M3 format, including the AI-disclosure footer. |
| `archive/audit_20260521_075800.md` | An APPROVED audit report from SOP §9.2, with re-derived totals and ten cross-checks. |
| `build_workbook.py` | The Python script that builds the dummy `.xlsx` from scratch — readable if you want to see how the formulas are wired. |

## What the example numbers add up to

- Verified in bank (named): ₹2,85,000
- Awaiting bank statement (donor evidence in): ₹45,000
- Awaiting identification (bank credit, no name): ₹15,000
- **Grand total: ₹3,45,000**

Holding balance after sweep: ₹0 (5,000 + 20,000 in, 25,000 out).

Sweep DR row on `Holding_Account` and its matching CR on `Canara_Statement` are both `Status = Internal`. Neither inflates the grand total — the rupees are counted exactly once, when each contributor paid into Holding.

## How to read the workbook

Open `Fundraise_Reconciliation_Sheet.xlsx` in Excel or Numbers. The sheet order is the order to read them in:

1. **Master_List** — one row per contributor. Note `Received_Confirmed` is a SUMIFS pulling from every bank sheet on `Mapped_To_# = #` and `Type = "CR"`. No formula sits on top of a row containing data.
2. **PNB_Statement / Canara_Statement / Holding_Account** — 1:1 mirror of each bank PDF. PNB has no `Time` column populated (the statement format is date-only). Canara and Holding do. The `Linked Name` column (J) auto-resolves via VLOOKUP.
3. **Distribution_List** — recipients. Notice rows 11–13 are forwarders (no `Master_List Link`), row 14 is a Group, and row 15 is `Removed = TRUE`.
4. **Audit_Log** — append-only. Every change to the workbook (and every send) lands here.
5. **Action_Items** — derived queue. Two open, one resolved.
6. **Dashboard** — the only place totals live. The grand total of ₹3,45,000 is the publishable headline.

## Regenerating the workbook

```bash
cd code/examples
python3 build_workbook.py
```

Then recalc with LibreOffice (Mac/Linux):

```bash
soffice --headless --calc --convert-to xlsx Fundraise_Reconciliation_Sheet.xlsx
```

…or just open and save it once in Excel / Numbers / LibreOffice.
