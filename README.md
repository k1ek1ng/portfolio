# Internship write-ups

Three projects I built as the sole Software Engineering Intern at Cypress
Industries, summer 2026. The code is proprietary; these describe the work and the reasoning,
shared with permission. Customer and supplier names are omitted throughout.

- [**AP invoice matching and vouchering**](case-study-ap-invoice-agent.md):
  supplier invoice to human-approved ERP voucher. How an 800-invoice backtest set
  the price tolerance, the three categories the matcher refuses to decide, and why
  the human approval step is permanent rather than scaffolding.

- [**Customer PO to sales order**](case-study-po-to-so.md): PO PDF to a booked
  sales order. Extraction coverage from 54% to 84% one customer layout at a time,
  a part-resolution bug whose obvious fix was the wrong one, and what idempotency
  costs on a write path.

- [**AR invoice delivery**](case-study-ar-invoicing-pipeline.md): the day's
  invoices to one reviewed draft per customer. Why fuzzy name matching was built,
  measured, and then not used for routing.

Public code: [erp-query-mcp](https://github.com/k1ek1ng/erp-query-mcp) ·
[ai-document-extractor](https://github.com/k1ek1ng/ai-document-extractor)
