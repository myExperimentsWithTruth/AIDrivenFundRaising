# AIDrivenFundRaising

Audit-grade personal fundraise reconciliation, run as a hand-tended workflow with a Claude agent.

Repo: https://github.com/myExperimentsWithTruth/AIDrivenFundRaising

This repo packages the operating manual, the architecture, the diagrams, and a Claude skill (`fundraise-recon`) that anyone can install to run the same workflow for a new fundraise or a small-NGO donor cycle.

The story behind it is in [`blog.md`](./blog.md).

## What's in here

| Path | What it is |
|---|---|
| `blog.md` | First-person write-up of why this exists and what was built |
| `RECON_SOP.md` | The full operating manual — 13 sections, 13 non-negotiables, five procedures, two agents |
| `diagrams/architecture.svg` | Three-layer data architecture (evidence → workbook → publications) |
| `diagrams/solution_daily_cycle.svg` | End-to-end daily reconciliation cycle with the audit gate |
| `skill/fundraise-recon/SKILL.md` | The Claude skill manifest — install this to run the workflow |
| `templates/daily_update.template.md` | The shape of a daily donor message |
| `templates/audit_report.template.md` | The shape of an audit pass / discrepancy report |
| `templates/Excel_workbook_structure.md` | Sheet-by-sheet specification of the reconciliation workbook |

## How to use this

1. Read `blog.md` if you want the context.
2. Read `RECON_SOP.md` end-to-end before you run a rupee through this. The non-negotiables in §13 are the ones to internalise.
3. Install the skill at `skill/fundraise-recon/` into your Claude Code or Cowork environment.
4. Create a fresh project folder, drop in the templates from `templates/`, and start by writing your `Master_List` in the Excel workbook.
5. The agent will load the SOP on first interaction and follow it. The audit agent is a separate invocation that re-derives everything from scratch.

## What is deliberately NOT in here

This repo is publishable — meaning it contains nothing personal:

- No donor names, phone numbers, UTRs, or bank-statement contents.
- No real Excel workbook with data.
- No actual daily-update messages.
- No archived bank statement PDFs.

If you adapt this for your own fundraise, keep your live `Fundraise_Reconciliation_Sheet.xlsx`, `payments_log.md`, `daily_updates/`, and `bank_stmts/` outside of any repo. The `.gitignore` in this folder enforces that for the directory you actually run the workflow in.

## License

MIT. See `LICENSE`. Use it, adapt it, ship it. If it helps a fundraise, that is the only credit needed.

## Contributing

If you adapt this for a new use case — a small NGO, a recurring donor cycle, a different bank format, a different messenger — write up what changed and open a pull request against `RECON_SOP.md`. The non-negotiables in §13 are the ones to keep stable; everything else is open to evolve.
