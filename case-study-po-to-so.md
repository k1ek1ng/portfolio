# Customer PO to sales order

*Cypress Industries, Software Engineering Internship, summer 2026. Code is
proprietary; shared with permission. Customer names are omitted.*

**In one line:** customer purchase order PDF in, approved sales order in the ERP
out. Saves about an hour per order.

## The problem

Customer POs arrive as PDFs in a different layout for every customer. A program
manager read each one and typed the sales order into the ERP. A wrong part number
doesn't stay small: it flows into the work order, purchasing, and shipping.

## What I built

- **Read the PO.** Each major customer's layout gets its own parser. Layouts it
  doesn't know are flagged, never guessed.
- **Check every line** against the company's part list. Each line shows how it
  was matched, and a PO is only ready when every line matched on a reliable ID.
- **Review page** where the program manager checks the draft and approves it.
- **Book the order** in the ERP after approval.

## Decisions worth explaining

**Never book an order twice.** If the program crashes right after the ERP
creates an order, the next run must not create a second one. The tool keeps a
crash-safe record of what it booked, checks for existing orders before anyone
reviews a PO, and stops if a PO number is missing or the lines don't add up to
the PO total.

**Fix the cause, not the symptom.** One customer's part numbers never matched.
The obvious fix was adding them to a lookup file. Checking the database showed
the real cause: the ERP stores those numbers with a revision suffix like
" Rev A", so the bare number on the PO could never match. Patching the lookup
file would have hidden the problem until the next new part.

## Result

In production. Program managers review and approve instead of typing, saving
about an hour per order (roughly $25K a year).

## My role

Sole developer: the parsers, part matching, the review page, and the ERP write
path.
