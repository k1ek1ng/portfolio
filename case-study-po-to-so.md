# Customer PO to sales order

*Cypress Industries — AI Automation Internship, summer 2026. Code is proprietary;
this write-up is shared with permission. Customer names are omitted.*

Customer purchase orders arrive as PDFs, in as many layouts as there are
customers. A program manager reads each one and keys a sales order into the ERP.
A wrong part number there does not stay local — it cascades into the work order,
the purchase order, kitting, and shipping.

I built the pipeline that extracts the PO, validates every line against the part
master, and produces a reviewable draft. Phase 2 added the write path: after a
person approves, it books the sales order.

## Extraction coverage, one customer at a time

There is no general PO parser. Coverage came from working the real corpus and
adding a layout at a time, measuring after each:

| change | coverage |
|---|---|
| baseline | 54% |
| first large-customer parser | 72% |
| second parser + a variant fix on the first | 80% |
| date-format fix + third parser | 83% |
| fourth parser (line format) | 84% |

Each step is a commit with the before/after number in the message. Unsupported
layouts are flagged as unsupported — never guessed at.

## Every line says how it matched

A line resolves through a ranked set of paths, and the worksheet shows which one:

| status | meaning |
|---|---|
| `matched` | exactly one part, via a **strong** path (our item ID, the customer's item ID, or a curated alias) |
| `needs_review` | resolved only on a manufacturer number (not customer-specific), **or** matched more than one part at the top path, **or** not found — the `reason` column says which |

A PO is "ready for review" only when **every** line is `matched`. One weak line
holds the whole document, because a PM checking a 40-line PO will not
independently re-verify the one line that quietly matched on a manufacturer
number.

## The diagnosis I am proudest of

One customer's parts come in two number styles. One style resolved; the other
always came back `not_found`. The obvious read was a coverage gap in the
cross-reference file — seed the missing numbers and move on.

That would have been a workaround for the wrong problem. Four candidate causes,
each checked against the sandbox read-only:

| candidate | verdict |
|---|---|
| the cross-reference file is missing these parts | **symptom, not cause** — the file already holds 76 rows of that same family, all resolving fine |
| the ERP's customer-item field is decorated | **this is it** — the ERP stores the number with a trailing `" Rev A"`, so the bare number on the PO cannot exact-match |
| the sandbox is stale relative to production | **ruled out** — the part is present in the sandbox |
| the number lives in a drawing/spec field | **ruled out** — the table has no such column |

So it was never a numbering-family problem. Seeding the file would have hidden a
revision-suffix mismatch that would resurface on every new part. No code changed
on that pass; the finding was written up as a recommendation.

## Writing to the ERP without writing twice

The write path is where a retry stops being free. A crash between "the ERP
created the sales order" and "we recorded that it did" must not produce a second
order on the next run. What the idempotency work covers:

- A local ledger written atomically — temp file, `os.replace`, `fsync` — so a
  crash mid-write cannot corrupt it. If the tail is torn on the next start, it
  heals rather than losing the next record.
- A blank PO number halts the run instead of booking an unidentifiable order.
- On a read-back that cannot be verified, the run **never deletes** the sales
  order. An unverifiable success is not a failure, and deleting a correct order
  is worse than leaving a duplicate for a person to find.
- Extracted lines are reconciled against the PO's own total before anything is
  booked; if they come up short, the run halts.
- Multiple revisions of the same base part are held for a human rather than
  collapsed to one.
- The queue is duplicate-aware at intake: a PO already in the ERP shows its
  existing order number before anyone reviews it.

Verified end to end against the live sandbox against a documented runbook — all
gates pass, checked through both the browser and direct HTTP.

## My role

Sole developer. Extraction and the per-customer parsers, the part-resolution
paths and their ranking, the diagnostic work above, the write path and its
idempotency guarantees, and the review web application the PMs use.
