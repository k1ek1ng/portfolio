# AP invoice matching and vouchering

*Cypress Industries — AI Automation Internship, summer 2026. Code is proprietary;
this write-up is shared with permission. Customer and supplier names are omitted.*

Supplier invoices at Cypress were entered by hand: open the PDF, key the header
and lines into the ERP, check each line against the purchase order and against
what was actually received, create the voucher. Steps 4-7 of that process — verify
part number, quantity, and unit price against the PO — are mechanical, and they
are where mis-keys turn into payment errors.

I built the pipeline that does those steps and hands a person a decision.

```
invoice PDF ──► extractor ──► matcher ──► review app ──► human ticks "approve" ──► ERP voucher
                             (read-only SQL      (web page)                     (REST API,
                              vs PO + receipts)                                  prod double-gated)
```

Stages 2-6 run live. Posting to production requires **both** the `--allow-prod-write`
flag and `ALLOW_PROD_WRITE=true` in the environment — one without the other does
nothing. Sandbox is the default target everywhere.

## The part that took the longest: what counts as a price match

The v1 threshold was a guess — auto-clear within 2% or $1.00, whichever was more
forgiving. Both numbers were wrong, and the dollar floor was worse than loose: on a
$0.30 part, a $1.30 invoice is 330% over and still cleared, because the delta was
under $1.00.

I replaced the guess with the distribution. `backtest.py` replays invoices the AP
lead already approved — a posted voucher is a recorded human decision — through
the real matcher and measures agreement. Across the 800 most recent approved
invoices (836 priced lines):

| price delta | lines |
|---|---|
| exactly 0% | 804 of 805 |
| non-zero | 1 line, **-0.26% / $0.0040** |

Real variance on approved invoices is essentially zero, so the band was doing no
work. I moved to exact matching, compared at a fixed rounding precision. Choosing
that precision was the actual decision:

| round to | of the 38 raw differences, how many become equal | would flag |
|---|---|---|
| 6 dp | 37 | 1 |
| **4 dp** | **37** | **1** |
| 3 dp | 36 | 2 |
| 2 dp | 37 | 1, but a *different* one |

The ERP stores prices as 10-decimal Decimals, so most raw differences are
representation noise (`0.0890000004` vs `0.0890000000`, a gap of 4e-10). 4 dp
absorbs the 37 noise lines and flags the one genuine difference — a $1.5600 PO
price billed at $1.5560. **2 dp is unsafe**: it rounds that real gap away and
flags an unrelated line instead. 4 dp is the floor of "safe."

Cost of the tightening, measured on identical inputs: auto-clear goes 829 -> 828
of 836 (99.2% -> 99.0%). Exactly one line flips, and it is the real one.

The band is not hard-coded away. `price_tol_pct` / `price_tol_abs` still exist and
default to zero, so finance can loosen it later without a code change.

## What the matcher refuses to decide

Three categories never auto-clear, regardless of the numbers:

- **Unit-conversion lines** (~16% of PO lines — priced per box, reel, or foot).
  Convert-then-compare is a later refinement; until then a conversion mismatch
  must not be able to slip through inside a tolerance.
- **Split-price PO lines**, where the schedule changed. These clear only on an
  authorized end of the range, never a value in the gap. A line with three or
  more distinct prices clears only on its two extremes — a middle authorized
  price stays flagged. Conservative on purpose.
- **Ambiguous parts** — the same part on more than one PO line. Disambiguating
  needs the PO line number, which suppliers do not reliably print, so the line
  goes to a human rather than to a guess.

Plus a vendor-independent backstop: if an invoice's line amounts do not sum to
its stated total, every line on that invoice routes to exceptions. That is how
un-itemized tariff charges get caught — one distributor bakes them into the total
with no line and no keyword to scan for. On the first real batch it fired on
exactly 1 of 7 parsed invoices, correctly: an un-itemized tariff of roughly 9% of the invoice total.

## Two findings that changed the design

**The AP control account is not derivable.** Before wiring the voucher's
accounting fields I tried to infer which AP account each bill posts to. Across
the full history of posted bills only two accounts are in use, and the choice is
not a function of anything the invoice or PO carries: a substantial minority of
vendors have used both, every invoice is in the same region, and the same vendor
still splits across both within a single year. It is a per-transaction accounting
judgment, tracking a migration between accounts, not a rule. So I did not invent one — the account stays visible
on the worksheet as a human override, with the current-era default configurable.

**A required field the API docs did not require.** The voucher line's GL account
is named for purchase-price variance, and at zero variance it looks optional.
Some items default it server-side; others reject the POST outright with *"Required
field GLL_Account has no value."* Found live, on a real sandbox post. It is now
sent on every receipt.

## What the backtest does not prove

It replays invoices that were **approved**. That measures recall — do we agree
with past approvals — not precision. It cannot tell me whether the matcher would
auto-clear something the AP lead would have rejected, because the sample contains
no rejections.

That asymmetry is why the human approval step is permanent rather than
scaffolding. Posting an AP voucher creates a payable and a GL entry; a validation
that structurally cannot measure false positives is not a basis for removing the
person. The review app makes approving fast — tick the clean invoices, one button,
only the ticked ones post — which is a different goal from approving automatically.

## My role

Sole developer, and the only person in the AI function. I built the extractor
hardening, the matcher and its routing rules, the read-only SQL layer against the
ERP, the OAuth REST client and voucher assembly, the review web app, and the
backtest harness. I wrote the operator runbook for AP and ops, and tuned the
thresholds with the AP lead against her real invoice batches.
