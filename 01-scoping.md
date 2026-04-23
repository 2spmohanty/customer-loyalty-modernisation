# Scoping

> **Document:** 01-scoping.md  
**Status:** ```APPROVED``` .     
**Last Updated:** 20 April 2026 .   
**Approved By:** Customer ```(Daniel Parrott)``` . 

---

## The Problem 

A customer profile that lives across 45 normalised tables is not a database problem. It is a product velocity problem.

Every new loyalty feature triggers a chain reaction —schema changes across multiple tables, multiple teams, multiple sprints. The architecture is working against the business, not for it.

This engagement fixes that within 6 months, without touching your existing application integrations.

---

## What I Understand the Goal to Be

| Dimension | Current State | Target State                  |
|-----------|--------------|-------------------------------|
| Profile read | 45-table JOIN | Single document fetch         |
| New loyalty attribute | 3-sprint DB migration | Add a field, ship in 1 sprint |
| Read latency (p95) | Degraded under load | 30ms **(Dan confirmed)**      |
| Operational model | DBA-dependent | Team self-sufficient          |
| Migration risk | — | Zero downtime cutover         |

---

## Assumptions 

These materially affect the architecture.

> Confirmed as True by Daniel Parrott — 20th April 2025

- **A1**  : Legacy RDBMS stays live during migration (not decommissioned on day one).
- **A2**  : Read latency is the primary SLA (not write throughput).
- **A3**  : Eventual consistency is acceptable for profile reads across regions.
- **A4**  : Loyalty transaction history is large volume but read patterns are predictable.

---

## Clarifying Questions for the Panel

> Answered by Daniel Parrott — 20th April 2025

| # | Question | Answer | Architecture Impact |
|---|----------|--------|---------------------|
| Q1 | Current p95 read latency? Target? | Current: 60ms. Target: 30ms. Already missing target outside peak season | Read replica and caching strategy confirmed as day-one requirement — not optional |
| Q2 | How many active customer records? | 12M active, 20M inactive (no transactions in 12 months) | 32M total records. Sharding strategy must account for full dataset, not just active |
| Q3 | Data residency regulations? | GDPR applies to EU customers (~15% of base). No other regional requirements | EU customer data must remain in EU region. Data residency built into multi-region architecture from day one |
| Q4 | Peak season traffic multiplier? | 2X average. Busiest days hit 5X | 5X peak moves sharding from stretch to day-one design decision |


---

## What 6 Months Delivers

### Committed — I Am Signing Up For These

> Accepted by Daniel Parrott with suggestion for GDPR, Latency Budget, Loyalty API — 20th April 2025

```
1)  Customer profile on document model
    Single document fetch replaces 45-table JOIN
    Target: p95 read latency < 30ms
    (current baseline: 60ms, already outside target pre-peak)

2)  Sharding from day one
    32M total records + 5X peak traffic
    Single-node architecture is not an option

3)  GDPR-compliant multi-region architecture
    EU customer data (15% of base) pinned to EU region
    Data residency enforced at the storage layer

4)  Zero downtime cutover
    Dual-write strategy, traffic shifted gradually (Strangular fig pattern)
    Legacy RDBMS remains hot until validation complete

5)  Loyalty points API live at month 6
    Moved from stretch to committed:  Own milestone gate in the delivery plan

6)  New loyalty attributes ship without DB migrations
    Schema flexibility by design

7)  Team operational independence
    Runbooks, playbooks, and knowledge transfer delivered as first-class outputs — not afterthoughts
```

### Stretch — Achievable If Decisions Are Made Fast

- ~~Real-time loyalty balance API (sub-10ms reads): Requires caching strategy confirmed in month 2.~~
- Full-text customer search Profile search by name, email, loyalty tier.
-  ~~Multi-region read replicas: Low-latency reads across global retail markets. Requires regions confirmed in month 1~~
- ~~Low-latency reads for non-EU retail markets~~   
  ```EU region is already a committed architecture decision driven by GDPR — this stretch item refers to additional regions for performance optimisation only.``` .   
  Requires: traffic distribution data per region confirmed before month 2 closes .  

### Backlog — After Stabilisation

- Legacy RDBMS decommission:   
    All integrations must be validated first  
    Realistic: month 9-12  

- Full application refactoring  
    Removing legacy SQL patterns from application code  
    Beyond current budget constraint  

---

## Proposed Phasing

**MONTH 1 — Discovery + Foundation** . 
  Full data audit across 45 tables . 
  Target document schema designed and validated . 
  Sharding key strategy confirmed (32M records,5X peak traffic is a day-one constraint not a stretch).
  EU data residency architecture **Agreed**
  Infrastructure provisioned across required regions
  - **Gate: Panel sign-off on document model and multi-region topology**



**MONTH 2 — Build + Pilot** . 
  Core customer profile APIs live on new stack
  Dual-write pipeline operational . 
  EU region isolated and GDPR controls verified . 
  Team upskilling begins . 
  - **Gate: p95 read latency validated against 30ms target on representative dataset** . 

**MONTH 3 — Migration + Validation** . 
  Full 32M record dataset migrated and reconciled. (12M active + 20M inactive — both in scope) . 
  5% production traffic on new stack . 
  Rollback procedure tested and documented . 
  - Gate: **Zero data inconsistency confirmed. EU customer records confirmed in EU region only** . 

**MONTH 4 — Scaling + Hardening** . 
  50% traffic shifted . 
  Performance validated at 2X normal load . 
  GDPR compliance audit completed . 
  Operational runbooks drafted by customer team . 
  - Gate: **p95 latency target of 30ms met at 50% load. No GDPR findings outstanding** . 

**MONTH 5 — Cutover + Peak Readiness** . 
  100% traffic on new stack . 
  Legacy RDBMS in read-only mode . 
  Loyalty points API live (committed deliverable) . 
  5X load simulation run and validated . 
  New loyalty features shipping on new architecture . 
  - Gate: **2 weeks stable at 100% traffic** . 
          **5X load test passed** . 
          **No rollback triggered** . 

**MONTH 6 — Stabilisation + Handover** . 
  Team fully autonomous on new stack .   
  Legacy RDBMS decommission plan scoped and formally handed to customer team . 
  Backlog items prioritised and documented . 
  Engagement closed . 
  - Gate: **Ops signs off on operational readiness** . 
          **Customer team leads — consulting team shadows** . 
---

## Next

[02 — Architecture](02-architecture.md)   








