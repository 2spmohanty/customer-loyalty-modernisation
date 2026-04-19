# Scoping & Requirements

---

## The Problem 

A customer profile that lives across 45 normalised tables is not a database problem. It is a product velocity problem.

Every new loyalty feature triggers a chain reaction —schema changes across multiple tables, multiple teams, multiple sprints. The architecture is working against the business, not for it.

This engagement fixes that within 6 months, without touching your existing application integrations.

---

## What I Understand the Goal to Be

| Dimension | Current State | Target State |
|-----------|--------------|--------------|
| Profile read | 45-table JOIN | Single document fetch |
| New loyalty attribute | 3-sprint DB migration | Add a field, ship in 1 sprint |
| Read latency (p95) | Degraded under load | < 20ms **(My assumption. Need to COnfirm)**|
| Operational model | DBA-dependent | Team self-sufficient |
| Migration risk | — | Zero downtime cutover |

---

## Assumptions 

These materially affect the architecture.

I need validation before the session locks in.

**A1**  Legacy RDBMS stays live during migration (not decommissioned on day one).
**A2**  Read latency is the primary SLA (not write throughput).
**A3**  Eventual consistency is acceptable for profile reads across regions.
**A4**  Loyalty transaction history is large volume but read patterns are predictable.

---

## Clarifying Questions for the Panel

**Q1** Current p95 read latency on customer profile? What is the acceptable target? (Will help me decided on COst factor that will help me with Acceptable Latency with cache or with Read Only Nodes with data duplication)
**Q2** How many active customer records? (determines sharding strategy)
**Q3** Data residency regulations across retail regions? (determines multi-region architecture)
**Q4** Peak season traffic  — 2x or 10x? (determines whether we shard from day one)

---

## What 6 Months Delivers

### Committed — I Am Signing Up For These

- Customer profile on document model: Single fetch replaces 45-table JOIN
- Zero downtime cutover: Dual-write strategy, traffic shifted gradually
- Legacy RDBMS remains hot until validation complete.
- New loyalty attributes ship without DB migrations: Schema flexibility by design
- Team operational independence
- Runbooks, playbooks, and knowledge transfer delivered as first-class outputs — not afterthoughts

### Stretch — Achievable If Decisions Are Made Fast

- Real-time loyalty balance API (sub-10ms reads): Requires caching strategy confirmed in month 2.
- Full-text customer search Profile search by name, email, loyalty tier.
-  Multi-region read replicas: Low-latency reads across global retail markets. Requires regions confirmed in month 1

### Backlog — After Stabilisation

- Legacy RDBMS decommission: 
    All integrations must be validated first
    Realistic: month 9-12

- Full application refactoring
    Removing legacy SQL patterns from application code
    Beyond current budget constraint

---

## Proposed Phasing

**MONTH 1 — Discovery + Foundation**
Data audit across 45 tables
Target schema designed and validated
Infrastructure provisioned

- Gate: Panel sign-off on document model

**MONTH 2 — Build + Pilot**
Core profile APIs live on new stack
Dual-write pipeline operational
Team upskilling begins
- Gate: Read latency validated vs baseline


**MONTH 3 — Migration + Validation**
Full data migrated and reconciled
5% production traffic on new stack
Rollback procedure tested
- Gate: Zero data inconsistency confirmed

**MONTH 4 — Scaling + Hardening**
50% traffic shifted
Performance validated at scale
Operational runbooks drafted
- Gate: p95 target met at 50% load

**MONTH 5 — Cutover**
100% traffic on new stack
Legacy RDBMS in read-only mode
New loyalty features shipping on new architecture
- Gate: 2 weeks stable, no rollback triggered

**MONTH 6 — Stabilisation + Handover**
Team fully autonomous
Backlog scoped and formally handed over
Engagement closed
- Gate: VP Ops signs off on operational readiness

---

## Risks & Mitigation








