# Customer Profile & Loyalty Modernisation

**Client:** Global Retailer
**Engagement Type:** Legacy RDBMS → Document Model Migration
**Status:** ```DRAFT```
**Last Updated:** [19 April 2026]

---

## About This Repository

This repository contains the working solution design
for the Customer Profile and Loyalty modernisation
engagement. It is structured as a live delivery
document — updated as decisions are made and
assumptions are validated.

---

## Navigation

| Section | Description | Status |
|---------|-------------|--------|
| [01 — Scoping & Requirements](01-scoping.md) | Assumptions, open questions, constraints | In Progress |
| [02 — Architecture](02-architecture.md) | Target state design, technology selection | Draft |
| [03 — Delivery Roadmap](03-roadmap.md) | Phased plan, gates, rollback triggers | Draft |
| [04 — Risks & Mitigation](04-risks.md) | Identified risks and management strategy | Draft |

---

## The Problem in One Paragraph

A customer profile distributed across 45 normalised
tables is not a database problem — it is a product
velocity problem. Every new loyalty feature triggers
a chain reaction of schema changes across multiple
tables, multiple teams, and multiple sprints.
This engagement fixes that within 6 months,
without disrupting existing application integrations.

---

## Quick Reference — Delivery At a Glance

```
Month 1  →  Discovery + schema design agreed
Month 2  →  Core APIs live, dual-write pipeline running
Month 3  →  Full data migrated, 5% traffic shifted
Month 4  →  50% traffic, performance validated at scale
Month 5  →  100% cutover, new features shipping
Month 6  →  Team handover, engagement closed
```

---

## Architecture Diagram



---

> This is a working document.
> All recommendations are subject to refinement based on discovery findings and panel input.


---

