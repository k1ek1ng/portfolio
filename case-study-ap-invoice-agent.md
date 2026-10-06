# AP invoice matching and vouchering

*Cypress Industries, Software Engineering Internship, summer 2026. Code is
proprietary; shared with permission. Supplier names are omitted.*

**In one line:** supplier invoice PDF in, human-approved voucher in the ERP out.

## The problem

Accounts payable entered every supplier invoice by hand: open the PDF, type it
into the ERP, then check each line's part, quantity, and price against the
purchase order and what was actually received. That checking is mechanical, and
it is where typos turn into wrong payments.

## What I built

```
invoice PDF -> extract -> match against PO + receipts -> review page -> person approves -> ERP voucher
```

- **Extract** the header and lines from the PDF.
- **Match** each line against the PO and receipts using read-only database queries.
- **Review page** where AP ticks the clean invoices and approves them in one click.
- **Post** the approved ones to the ERP through its API. Writing to production
  takes two separate switches, so one mistake alone can't post anything.

## Decisions worth explaining

**Exact price matching instead of a tolerance.** The first version cleared any
price within 2% or $1.00. That sounds safe until you see a $0.30 part billed at
$1.30: 330% over, but under a dollar, so it passed. I replayed 800 invoices the
AP lead had already approved and found that real price differences were
essentially zero. So I switched to exact matching, rounded to 4 decimal places.
That's the precision that ignores tiny rounding noise in the database but still
catches a real $1.5600 vs $1.5560 difference.

**Some lines always go to a person.** The matcher never auto-approves lines
priced per box or reel (unit conversions), POs whose price changed partway
through, or parts that appear on more than one PO line. It also flags any invoice
whose lines don't add up to its total, which is how hidden tariff charges get
caught.

**The human approval step stays.** The replay only contains invoices that were
approved, so it shows the matcher agrees with past approvals. It can't show
whether the matcher would approve something a person would have rejected. Since a
voucher creates a real payable, a person stays in the loop. The review page makes
approving fast instead of automatic.

## Result

Running in production. Most invoice lines clear automatically, and the rest go to
AP with the reason they were flagged.

## My role

Sole developer: extraction, matching rules, database queries, the API client,
the review page, the backtest, and the runbook for the AP team.
