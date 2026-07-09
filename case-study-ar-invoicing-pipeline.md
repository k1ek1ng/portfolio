# Case study: AR invoicing pipeline

*Cypress Industries · AI Automation Internship · code proprietary, write-up shared with permission*

## Context

Customer invoicing at Cypress ran off a daily ERP batch: staff generated invoice documents one at a time, then wrote individual emails to send them. The manual step meant invoices went out late, formatting was inconsistent, and a backlog of exceptions (odd billing terms, consolidated invoices, credit holds) built up because routine invoices consumed the team's time.

## What I built

A Python pipeline that turns the daily ERP invoice batch into finished invoice PDFs and staged, context-aware email drafts — reviewed and sent by a person.

## Architecture

- A scheduled job pulls the day's invoice batch from the ERP
- Each invoice is rendered to a branded PDF from a template (line items, terms, remit-to)
- For each customer, the pipeline drafts an email that reflects context — first invoice vs. repeat, past-due balance reminders, multi-invoice consolidation — and stages it as a draft in the mail system
- Staff review the drafts each morning and send; anything unusual is flagged rather than guessed at

## Impact

- Routine invoicing went from a person-hours task to a review-and-send pass
- Invoices go out the same day the ERP batch runs
- The exception backlog shrank because staff time shifted from routine generation to actually resolving exceptions

## My role

Sole developer. Built the batch integration, PDF templating, and the email-drafting logic including the context rules, and iterated with the AR team on tone and edge cases (they rewrote my first draft templates, correctly).
