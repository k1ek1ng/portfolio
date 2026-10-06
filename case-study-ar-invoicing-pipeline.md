# AR invoice delivery

*Cypress Industries, Software Engineering Internship, summer 2026. Code is
proprietary; shared with permission. Customer names are omitted.*

**In one line:** the day's invoices in, one ready-to-review email draft per
customer out.

## The problem

Every day, accounts receivable wrote an email to each customer and attached their
invoices by hand. It wasn't hard, just slow, and the tricky cases waited behind it.

## What I built

A tool that groups the day's invoice PDFs by customer and creates one email draft
per customer with the right invoices attached. A person reviews and sends. The
tool has no way to send email at all, so it can't send by accident.

## Decisions worth explaining

**Matching customers was the hard part, not the email.** The invoice export and
AR's email list use different customer names and share no ID. I tried fuzzy name
matching, and on real data it picked the wrong company for the largest customer,
which has several similar-looking accounts. That would mean sending one
customer's invoice to another. So the tool doesn't guess. Each customer is
confirmed by a person once and saved by ID. The fuzzy matcher only suggests.

**Keep upkeep small.** Instead of mapping every customer up front, the tool asks
about new customers as they show up, and saves the answer in the same run.
Within a few batches, the regular customers were all covered.

**Double-check attachments.** Before attaching, each PDF is checked against its
own invoice record, so a customer can't receive someone else's invoice.

## Result

Running in production and operated by AR, not me. Invoices go out the same day.
I wrote a one-click launcher and a plain-language guide that starts with what the
tool will never do: send email, write to the ERP, or guess an address.

## My role

Sole developer, deployment, and documentation. AR rewrote my email templates,
which was the right call. They know their customers.
