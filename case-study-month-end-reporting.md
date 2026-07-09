# Case study: Automated month-end reporting

*Cypress Industries · AI Automation Internship · code proprietary, write-up shared with permission*

## Context

Month-end close at Cypress required assembling a pack of ~37 reports from the ERP — inventory valuations, AR/AP agings, sales summaries, WIP, and more. Each was run manually, exported, and formatted. The full close took 4–5 days, most of it mechanical report generation rather than analysis.

## What I built

An automated month-end pack generator built on top of a natural-language ERP query server I'd built earlier in the internship (an MCP server giving an LLM safe, read-only SQL access to the ERP).

## Architecture

- The NL-query server exposes schema discovery and validated, read-only SQL over the ERP database — the same three-tool design as my public rebuild
- Each report in the pack is a saved, parameterized query definition (month, entity, format) rather than a hand-run export
- A runner executes the full pack for a given period, renders the outputs to formatted spreadsheets, and assembles the close packet
- Finance reviews the pack instead of building it; ad-hoc follow-up questions ("why is this aging bucket up?") go through the NL-query interface directly

## Impact

- Report generation for close went from 4–5 days of manual work to minutes of machine time plus review
- The close conversation shifted from "is the pack done?" to "what do the numbers say?"
- The underlying query server became a general self-serve analytics tool beyond close

## My role

Sole developer. Designed and built the query server (schema-discovery tools, SQL validation, read-only enforcement), converted the report pack to query definitions with the finance team, and built the runner and spreadsheet output.

*The public, generic rebuild of the query server is on my GitHub: [erp-query-mcp](https://github.com/k1ek1ng/erp-query-mcp) — same architecture, fully synthetic data.*
