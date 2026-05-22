# What an old friend's passing taught me about agentic automation

A few days ago, I lost a friend.

An old friend went too early. We were classmates at Pantnagar, batch of 1998. More than that, he was my roommate.

For anyone who has lived through hostel life in the days before phones and internet and TV — the shared cigarettes, the shared drinks, the long conversations when the lights went out and the small hours stretched on — you know the kind of bond that gets built. The world had less in it back then. What we had was each other. That kind of friendship is hard to describe to anyone who has only known a connected life.

When the call came, I sat still for a while. A few batchmates started talking about putting together a fundraise for his wife and family. The need was real and the people closest to him were spread across cities and time zones. Someone had to run it, so I took it on.

Doing useful work in the days after a death is one way to keep going. This is mine.

What followed is the subject of this post. Not the grief — that belongs elsewhere. But the unexpected technical story that came with it.

## The fundraise quickly outgrew a spreadsheet

By day three, I had a small problem. Money was coming in from four channels — the family's PNB account, the family's Canara account, an intermediary pool that another batchmate was holding for some contributors, and a Milaap campaign for our overseas friends. Screenshots were arriving on WhatsApp at all hours: PhonePe receipts, NEFT confirmations, Google Pay slips. Bank statements would land once a day or sometimes once every two days.

Every morning at 0800 IST I was sending out a status update to a list that grew from 30 to 86 people across batches, work circles, and cross-references. The message had three parts — a personal note, the contribution instructions, and a complete numbered list of every rupee received so far, named and traced. People were forwarding it onward; some of them were donors themselves and they needed to see their entry confirmed by the next morning.

By day four I was making mistakes. A ₹400 visible-sum gap because I had written "1.00L" for someone who actually gave ₹1,00,100. A wrong-recipient send because two of my contacts share a first name. A contributor whose ₹3,000 sat in a screenshot for six hours before I logged it.

This wasn't a tech problem. It was a trust problem. Reputation is the asset when you're holding other people's money.

## What I built, with Claude

I am the co-founder of an AI and data company, so the instinct to automate is reflexive. But I wanted to be careful. This was not a product. This was a hand-tended, high-trust workflow. I needed automation that protected trust, not one that optimized for speed at the cost of it.

I worked with Claude — in the Anthropic desktop app called Cowork — as a pair. Over several long sessions, we built four things.

**A single source of truth.** One Excel workbook called `Fundraise_Reconciliation_Sheet.xlsx`. Master_List of contributors. One sheet per bank channel, mirroring the actual statement PDFs. A Holding_Account sheet for the intermediary pool. A Dashboard that recomputed on every save. The non-negotiable rule was that no in-sheet TOTAL row could ever live inside a data sheet — every aggregation lived only on Dashboard. The first time we violated that, a naive row-sum doubled the grand total during an audit and nearly went out the door. Lesson learned, encoded, never revisited.

**An SOP that wasn't code.** A document — `RECON_SOP.md` — written in plain English. It covered the four-state reconciliation matrix (Verified / Pending / Unidentified / Mismatch), the trust hierarchy (bank stmt > screenshot > SMS > verbal claim), the daily triggers, and a set of twelve non-negotiables. Anyone who reads it can run the workflow. Future-me could come back to it after weeks and pick up exactly where I left off.

**A safety net.** An audit agent that runs after every send and reports either APPROVED or DISCREPANCY. It doesn't trust the primary agent's prior work. It re-reads the bank statement PDFs, re-does the math from scratch, and compares to what was actually published. If they don't agree to the rupee, the day doesn't close.

**Distribution that scaled.** This is where it got interesting.

## The browser-automation story

The daily send went to 86 recipients across WhatsApp. On day one it took me three hours — copying lists, formatting messages, finding the right chat, double-checking each name. Once I had a rhythm, it settled to about forty minutes of careful clicking every morning, with a non-zero chance of sending the wrong message to the wrong person.

The first automation approach used WhatsApp Web search. For each recipient, the agent would type the name in the search box, find the right chat, verify, and send. It worked, mostly, until two days in when a common first name matched both a contributor and an unrelated contact saved in my phone. That stranger got the entire fundraise message. I had to delete and apologize. We immediately added a rule that all name tokens must match before sending, and we shifted strategy.

The better approach was to use phone numbers and `wa.me/send?phone=` deep-links. The chat opens directly, the chat header shows exactly who you're sending to, and there is no name collision possible. But I only had phone numbers for 14 of my 86 recipients. So I did the unglamorous work — exported my Mac Contacts, matched them against the Distribution_List by name, and populated the Phone column once and for all. From the next morning on, the same blast that used to take forty minutes ran in about four.

The browser automation itself is built on Anthropic's Claude in Chrome extension. The agent navigates to the deep-link, waits for the composer to load, pastes the message via a ClipboardEvent, clicks send, and verifies the chat header matches the intended recipient before continuing. Three messages per recipient, sequentially. The whole thing logs every send back to the Excel — Last Sent Date, Last Send Status — so the next day's run knows not to send duplicates.

## The numbers

The fundraise crossed ₹13.32 lakhs by day five. The first day's send took me three hours (typing, formatting, dispatching, fixing typos). Today's send took the agent under ten minutes for 66 individuals; the six group chats still need manual touch because WhatsApp's search input is uncooperative when scripted, and that's a fight I haven't won yet.

The reconciliation work — matching screenshots to bank credits, identifying unmapped UPI receipts, chasing pending claims — used to be my entire evening. Now it's about thirty minutes, including audit.

What used to be a third of my day is now a quarter of an hour.

## A word on ethics

WhatsApp is a personal space. The fact that you can automate sending to it does not mean you should.

A few non-negotiables I will state out loud, because they should be:

Do not use this kind of tooling to send marketing, promotion, surveys, recruitment pings, or any form of unsolicited contact. Use it only when you have a real, prior, named relationship with the recipient and they have already consented (explicitly or by long-standing convention) to receive messages from you.

Do not let the automation see anything it doesn't need to see. The recipient list I worked with lived inside my own Excel on my own machine, in a workspace folder the agent had explicit access to. No contact data, no phone numbers, no message content ever leaves that boundary. I disclose in every public message I send that an AI is helping me draft and dispatch — and I invite recipients to flag any mistake they see. That disclosure is non-negotiable.

Do not skip the verification step. Source-of-truth is the bank statement, not the screenshot. The agent never claims a rupee received until the bank confirms it. If the agent says ₹13 lakhs raised, all ₹13 lakhs sit in named bank rows that I can show to anyone who asks.

The technology is powerful. The wrong hands or the wrong intent turns it into spam at scale. Please don't.

## Where this is going

What we built is still a Cowork-mode interactive workflow — me with the agent, in conversation, one step at a time. The next version is obvious, even if I haven't built it yet: a CLI, scheduled jobs, full automation, scale to NGO-grade.

Imagine a small NGO that fundraises annually. They have a recurring donor list of a few hundred names, bank statements that arrive on a schedule, and a treasurer who currently spends two evenings a week reconciling. The same architecture I sketched here — Master_List + bank-channel sheets + Dashboard + audit agent — converts directly. Add a scheduler. Add a CLI front-end. Have the agent fetch bank PDFs from a connected email, auto-reconcile overnight, generate the morning donor digest, and only escalate to a human when something requires judgment. The human stays accountable; the agent removes the toil.

This is the agentic future I want to see — not autonomous systems acting without oversight, but agents that take the mechanical work off our plates while keeping us firmly in the loop on consequential decisions. The trust contract gets stronger, not weaker, with this architecture.

## For anyone who wants to reuse this

I've put the working files in a GitHub repo so anyone running a similar fundraise — or building tooling for one — can lift what's useful. Repo: **https://github.com/myExperimentsWithTruth/AIDrivenFundRaising**

What's in the repo:

- `RECON_SOP.md` — the full operating manual: 13 sections, 13 non-negotiables, five core procedures, two agents.
- A sample Excel workbook with the sheet structure (Master_List, Distribution_List, bank sheets, Dashboard, Audit_Log) — donor data redacted, formulas intact.
- Sample daily-message templates and an audit-report template.
- A README that explains how to adapt it to a new fundraise, or to a small-NGO recurring donor cycle.
- A packaged Claude skill, `fundraise-recon`, that drives the whole workflow inside Cowork or Claude Code.

### The skill itself

`fundraise-recon` is the same SOP wrapped as an installable Claude skill. Installing it teaches the agent the trust hierarchy, the four-state matrix, the procedures, and the non-negotiables — so it runs the workflow correctly from the first interaction instead of needing to be re-briefed each session.

| Field | Value |
|---|---|
| Name | `fundraise-recon` |
| Triggers | Phrases like "reconcile this", "import the bank statement", "generate today's update", "run the audit", "screenshot from a donor", "add a new contributor", or any mention of fundraise / donor reconciliation. |
| Inputs | A folder with the Excel workbook, the SOP, and a `bank_stmts/` subfolder. New donor screenshots and bank PDFs dropped in over time. |
| Outputs | Updated Excel (single source of truth), a generated daily-update markdown, an audit report (APPROVED / DISCREPANCY), and an append-only Audit_Log inside the workbook. |
| Companion | A separate audit pass that re-reads the SOP, the Excel, and the bank PDFs from scratch and either signs off or files a discrepancy. The primary agent never marks its own homework. |
| Status | Working in Cowork as a hand-tended workflow today; being packaged into an installable `.skill` for the repo. |

The skill is deliberately small — no scripts, no custom code. All logic lives in the SOP, which the skill loads on activation. That keeps it portable: any Claude agent that can read markdown and edit Excel can run the workflow. Swap out the Excel template and the SOP rules, and the same shape works for any fundraise, treasurer cycle, or donor reconciliation.

The architecture in five lines: three layers (raw evidence, source of truth, generated publications); one Excel workbook as the single source of truth; two agents (a primary that does the work, an audit that re-derives totals from scratch); five deterministic procedures (verify, generate message, parse bank PDF, import, back up); thirteen non-negotiables that the agents are not allowed to break.

A few snippets are worth reading even if you don't clone the repo.

**The trust hierarchy.** Every input is treated by what it proves, not by how it arrived.

| Source | What it proves | Treatment |
|---|---|---|
| Bank statement PDF | Money is provably in a known account | Truth |
| Sender screenshot | Sender attempted a transfer | Claim — pending bank-stmt match |
| Sender SMS | Sender's bank acknowledged the attempt | Claim — pending bank-stmt match |
| Verbal / WhatsApp | Sender says they paid | Claim — pending bank-stmt match |

Only the first row inflates the verified-received number. Everything else inflates the pending bucket and is shown to donors under that label.

**The four-state recon matrix.** Donor evidence and bank evidence intersect into four states, plus bookkeeping states for internal sweeps and confirmed-unrelated credits.

| Donor evidence | Bank evidence | State |
|---|---|---|
| Yes | Yes, refs match | VERIFIED |
| Yes | Yes, refs disagree | MISMATCH — investigate |
| Yes | No | CLAIMED-PENDING |
| No | Yes | STMT-UNIDENTIFIED |

**Three non-negotiables out of thirteen.** The full set is in the SOP; these are the ones to read first.

- No total or subtotal row ever lives inside a data sheet. All aggregations live on the Dashboard sheet only. Violating this once doubled the grand total during a self-audit.
- No amount in any public message is hand-typed. The daily message is always generated from Excel by the procedure in §8.2 of the SOP.
- The audit agent never edits anything. It only reads and reports. Fixes are always the primary agent's job, followed by a re-audit before the day closes.

Each non-negotiable came out of a specific failure earlier in the fundraise. Read them as a checklist before you start.

If you adapt this for a fundraise or for an NGO use case, write to me. I'd like to hear how it shifted.

## A small request

If you knew my friend, or if you simply want to help the family, the Milaap campaign is here: **[Milaap link — to be filled]**

If you're a developer or NGO operator who'd like to adapt any of what I described, write to me. The SOP and the reconciliation architecture are not proprietary. They were built for one fundraise, but they will work for many.

Thank you for reading. And to everyone who has already given — the calls, the chats, the rupees, the quiet check-ins — thank you. My friend did not leave me without a gift. His gift is the warmth that all your conversations are wrapping me with.

— Dobi (Vaibhav Dobriyal)
Pantnagar 1998, Co-founder & CPO, LUMIQ
