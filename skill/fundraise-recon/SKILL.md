---
name: fundraise-recon
description: >
  Run audit-grade reconciliation for a multi-channel personal fundraise or a small-NGO donor cycle. Use this skill
  whenever the user mentions reconcile, fundraise, donor screenshot, bank statement import, daily donor update,
  pending claim, sweep from intermediary, audit pass, discrepancy, or any phrase implying matching contributor
  evidence against bank credits. Also trigger when the user drops a bank statement PDF, a payment screenshot from
  WhatsApp or SMS, or asks to generate the morning donor message. The skill loads the operating manual on first
  use and follows the trust hierarchy, the four-state recon matrix, and the non-negotiables strictly. It asks
  before it assumes; it never edits bank sheets except from a bank statement; it never types an amount into a
  public message.
---

# fundraise-recon

Operating skill for running a personal fundraise with audit-grade reconciliation. Reads the SOP at
`references/RECON_SOP.md` on activation and executes its procedures inline.

## Quick orientation

**Two agents.**
- **Primary** — does the day-to-day work: ingest screenshots and bank PDFs, update the Excel workbook, generate the daily message, send via WhatsApp.
- **Audit** — runs separately. Re-reads the SOP, re-reads the bank PDFs, re-derives every total from scratch. Either approves the published message or files a discrepancy. Never edits.

**Three layers** (see `references/architecture.svg`).
- Layer 1: raw evidence (`payments_log.md`, `bank_stmts/<date>/*.pdf`) — read-only.
- Layer 2: single source of truth (`Fundraise_Reconciliation_Sheet.xlsx`) — only thing the primary agent edits.
- Layer 3: publications (`daily_updates/YYYYMMDD_update.md`, audit reports) — generated, never typed.

**Five procedures** the primary agent runs (SOP §8).
1. Verify the daily message before sending.
2. Generate the daily message from Excel.
3. Read a bank statement PDF.
4. Import a bank statement into Excel.
5. Back up the Excel before any big change.

Plus two supporting procedures: add a new bank sheet (§8.7), handle internal sweeps (§8.8).

**One audit procedure** (SOP §9.2) — independent re-derivation with ten cross-checks and a random-sample re-import.

## What this skill is allowed to do

- Read bank statement PDFs and extract transactions.
- Append to `payments_log.md` from donor screenshots, SMS, or claims.
- Update the Excel workbook strictly per SOP §6 triggers and §8 procedures.
- Generate daily messages and audit reports into Layer 3 files.
- Send daily messages via WhatsApp browser automation (deep-link based, strict-verify on the chat header).

## What this skill must NOT do

- Touch bank sheets except via the bank-statement import procedure (§8.4).
- Hand-type any amount into a public message.
- Skip the verification gate (§8.1) before any send.
- Send a daily message without an APPROVED audit report (§9.2).
- Map a sweep DR row to a contributor (would inflate totals).
- Place any total or subtotal row inside a data sheet (must live only on Dashboard).
- Send to a recipient whose chat header does not strict-match every distinct name token.
- Omit the AI-disclosure footer from the M1 block of any public message.

## Files this skill references

- `references/RECON_SOP.md` — the full operating manual. Loaded on activation. Re-loaded by the audit agent at the start of every audit pass.
- `references/architecture.svg` — three-layer architecture diagram.
- `references/solution_daily_cycle.svg` — daily cycle flow with the audit gate.
- `references/daily_update.template.md` — shape of a generated daily message.
- `references/audit_report.template.md` — shape of an audit pass / discrepancy report.

## Activation triggers

Phrases that should activate this skill: reconcile this, import this bank statement, donor sent a screenshot, generate today's update, generate the daily message, run the audit, check pending claims, the bank statement is here, sweep from intermediary, add a new contributor, mark this as identified, who has not paid yet, what is the gap, audit the day, close the day.

## Companion: audit agent

The audit agent is a separate invocation of this skill in "audit mode". It does not trust any of the primary agent's prior work for this session. It loads the SOP fresh, opens the Excel, opens the bank PDFs, and re-derives the three buckets (verified, pending, unidentified) independently. It compares its derivation to the published daily message and to Excel. It then writes either `archive/audit_<timestamp>.md` with APPROVED, or a discrepancy report that stops the day.

Run the audit agent: after every bank statement import; after every daily message is generated and before it is sent; at end-of-day; at end-of-fundraise (the audit at fundraise close produces the final ledger).

The primary agent never marks its own homework.
