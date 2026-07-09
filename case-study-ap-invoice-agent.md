# Case study: AP invoice-vouchering agent

*Cypress Industries · AI Automation Internship · code proprietary, write-up shared with permission*

## Context

Cypress's accounts-payable team manually processed every supplier invoice: open the PDF, key the header and line items into the ERP, match lines to the originating purchase order, and create an AP voucher. At ~2,400 invoices a month, this consumed roughly 80 hours of staff time monthly and was the full-time work of two people. Mis-keyed quantities and missed PO matches caused downstream payment errors.

## What I built

An agentic LLM workflow that takes an incoming supplier invoice and produces a draft AP voucher in the ERP, queued for human approval — never auto-posted.

## Architecture

- Invoice PDFs arrive via a watched inbox and are parsed by an LLM extraction step (header fields + line items → structured JSON, with confidence scores)
- A matching engine links each invoice line to open PO lines, using exact matches first, then fuzzy matching on part numbers and vendor name variants
- Matched invoices become draft vouchers written to the ERP through its API; anything below a confidence threshold, or with quantity/price variances outside tolerance, is routed to a human exception queue with the reason attached
- Every voucher requires human approval before posting — the agent drafts, people decide

## Impact

- ~2,400 invoices/month flow through the pipeline
- ~80 hours/month of manual data entry eliminated — work that previously required two full-time staff, who moved to exception handling and vendor management
- Exception queue surfaces genuine problems (price variances, missing POs) that were previously buried in routine keying

## My role

Sole developer. I designed the extraction prompts and JSON schemas, built the PO-matching logic and tolerance rules, integrated with the ERP's API for draft-voucher creation, and worked directly with the AP team to tune the exception thresholds until they trusted the queue.

*A generic, public rebuild of the extraction stage is on my GitHub: a public rebuild coming soon. The fuzzy vendor-name matching is public as another rebuild coming soon.*
