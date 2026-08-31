# AR invoice delivery

*Cypress Industries — AI Automation Internship, summer 2026. Code is proprietary;
this write-up is shared with permission. Customer names are omitted.*

Every day AR generates the day's customer invoices, then writes an email per
customer and attaches their invoices by hand. The work is not hard; it is just
long enough that the exceptions — odd terms, consolidated invoices, credit holds —
wait behind it.

I built the tool that does the assembly. It groups the day's invoice PDFs by
customer and writes **one draft per customer**, with that customer's invoices
attached. A person reviews the drafts and sends them.

It never sends. Not "the send flag defaults to off" — there is no SMTP or network
send path anywhere in the code. The safety is structural, so it cannot be undone
by a misconfiguration.

## The actual problem was identity, not email

The invoice export and AR's master email sheet share no clean key. The export has
a stable customer ID and the full legal name. The master sheet has AR's shorthand
and no ID. The names do not align.

The obvious fix is fuzzy name matching. I built the matcher, ran it on the real
data, and it failed in the way that matters: for the largest customer — which has
US, Mexico, and portal-only variants — **the correct match does not score
highest.** A confident wrong answer here sends one customer's invoice to another.

So the tool does not decide who gets an invoice. Recipients come from a curated
map keyed on the stable customer ID, where a person has confirmed each pairing
once. The matcher still does the legwork — it proposes; it never routes.

Every customer lands in exactly one bucket, and only one of them produces a draft:

| bucket | gets a draft |
|---|---|
| confirmed, has an email | yes |
| portal / Ariba customer | no — handled in their portal |
| unmapped or needs review | no — never guessed |
| a PDF whose invoice number is not in the export | no |

## Making the upkeep survivable

The map needs maintenance, and maintenance is where handed-off tools die. Two
decisions kept it small:

**Seed reactively.** Do not pre-fill the entire customer list. Run on a real batch and let
the unmapped list say exactly who is missing. Within a few batches the recurring
set is covered, and nobody spent a day on customers who invoice twice a year.

**Confirm inline, in the same run.** When an unknown customer appears, the run
stops and shows a best guess: accept, type the correct address, mark as portal,
or skip. Confirmations are saved and included in **that same run's** drafts — no
second pass. Unattended runs skip the prompt and flag instead.

The same pattern covers filing: PDFs are copied into AR's existing per-customer
folders, whose short names do not match the legal names either, so folders are
confirmed once the same way. It copies, never moves, never overwrites, and fails
loudly if the destination is unreachable.

Before attaching, each PDF is verified against its own invoice record, so a
customer cannot receive someone else's invoice even if the grouping is wrong.

## The handoff

The tool is deployed and run by AR, not by me. That meant writing for a reader who
has not seen the code: a one-click `.bat` runner, a plain-language guide, and an
explicit list of what the tool will never do — never sends, never writes to the
ERP, never guesses an address. The guide leads with that list, because the first
question anyone asks about an automation touching customer email is what it can
do when nobody is watching.

## My role

Sole developer. Grouping and drafting, the customer map and its confirmation
flows, the PDF verification, the filing step, deployment, and the operator
documentation. AR rewrote my first email templates, which was the correct
outcome — I know what the pipeline can prove, they know how they talk to
customers.
