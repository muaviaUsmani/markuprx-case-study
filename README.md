# MarkupRx — a drug-pricing engine built solo with Claude Code


Independent pharmacies routinely fill prescriptions where the reimbursement is lower than what they paid for the drug. Those "underwater" claims are hard to spot one at a time and harder to contest, because the evidence lives across a dozen public government datasets that don't share clean identifiers.

Earlier this year I built MarkupRx to see how far one engineer could take that problem, end to end, using Claude Code as the implementation agent.

## What it does
- Pulls 13 US government feeds (NADAC acquisition costs, RxNorm, the FDA Orange Book, state Medicaid data) into one model: ~248K NDC products and ~439K weekly acquisition-cost rows.
- Flags claims where a pharmacy's cost exceeds what Medicaid reimbursement would justify.
- Drafts a MAC appeal letter for each flagged claim, with the supporting price data attached, as a signed PDF.

## The engineering problems that mattered
**Entity resolution without guessing.** The same drug shows up as different NDCs, packages and RxCUIs across sources. I used a four-tier, identifier-based chain from NDC to RxCUI with no fuzzy matching, so every price comparison can be traced to an exact identifier, not a likely match. A wrong match in a pricing appeal is worse than no match.

**Ingestion that can't double-count.** Feeds update on different schedules and sometimes republish. Ingestion is concurrency-guarded and idempotent, and price history lives in year-partitioned tables so weekly loads stay fast as history grows.

**Auditability.** Every flagged claim keeps the source rows and as-of dates behind it, the same habit I learned building bank reconciliation and GL reporting at Darwin Homes: if a number can't be traced, it can't be trusted.

## How I built it: the agent writes, the tests decide
I treated Claude Code as the implementation agent and myself as the architect and reviewer. The rule was simple: no change lands unless the automated suite passes. The suite grew to about 570 tests, and they were the acceptance gate for every agent-generated change. That let me move fast without losing control. I wrote specs and edge cases, the agent proposed code, the tests and I decided what shipped.

Stack: Django/DRF, PostgreSQL 17, Celery/Redis, React 19 + TypeScript, Docker on Railway.

## What I learned about the market
I validated the idea with pharmacy owners and got blunt feedback from people who know drug pricing well. The data problem was real, but many independents get this kind of analysis bundled through their pharmacy services administrative organization (PSAO), so a standalone tool is a hard sell. I shelved MarkupRx as a product. The engineering stayed useful: it's the clearest example I have of taking an unfamiliar, regulated domain from zero to a working system quickly, with AI agents doing the heavy lifting under test discipline.

*I'm a senior product engineer working on card products at Capital One; before that I built accounting and payments systems at Darwin Homes.*
