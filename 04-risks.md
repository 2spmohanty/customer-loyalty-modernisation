
# Risks & Mitigation

> **Document:** 04-risks.md   
  **Status:** ``COMPLETED. AWAITING APPROVAL``   
  **Last Updated:** 20 April 2026  



---

## Context

These risks are not hypothetical. They are grounded in what we know today:

- System is already failing its 30ms p95 target on a normal traffic day at 1X load
- 5X peak is arriving in 6 months
- 32M records need migrating with GDPR constraints
- Limited budget for application refactoring
- Six month window with no flex confirmed

Risks are rated by likelihood and impact.
Mitigations are built into the delivery roadmap — not listed as an afterthought.

---

## Risk Atrributes

---

### Risk 1 — Performance at 5X Peak Load

```
Likelihood:   **HIGH**
Impact:      **HIGH**
Owner:       Joint — Consulting Team + Customer Infra Team
```

**What it is:**
System is at 60ms p95 on a normal day.
At 5X peak load without architectural changes that number becomes unusable. 
Holiday season is a hard deadline — there is no option for "we will fix it after peak" option.

**Mitigation:**
Sharding and caching layer are day-one design decisions — not optimisations deferred to month 3.

5X load simulation run in month 5 before cutover.
If simulation fails the gate, cutover does not happen.
Legacy RDBMS stays live as fallback.

Load testing strategy is a shared responsibility — see **Open Items section below**.

---

### Risk 2 — Data Model Complexity Hidden in 45 Tables

```
Likelihood:  MEDIUM
Impact:      HIGH
Owner:       Consulting Team
```

**What it is:**
45 normalised tables rarely document themselves cleanly.
Undocumented business logic, edge case data types, orphaned records, and silent application-level joins are common in patterns observed across domains.
Surprises here is a setback directly into the 6-month window.

**Mitigation:**
Month 1 is a full data audit before any schema design is finalised. 
No assumptions carried forward without evidence.

Discovery gate at end of month 1 — panel sign-off required before build begins.
If audit reveals significantly higher complexity than anticipated, scope is renegotiated at that gate. Not at month 4.

---

### Risk 3 — GDPR Compliance for EU Customer Data

```
Likelihood:  MEDIUM
Impact:      HIGH
Owner:       Joint — Consulting Team + Customer Legal/DPO (Data Stewards etc)
```

**What it is:**
15% of the 32M customer base is EU-resident.
GDPR mandates that PII data for EU customers does not leave the EU region.

During migration there is a window where data exists in two stores simultaneously — the legacy RDBMS and the new document store.
Both stores must be GDPR compliant during the dual-write window, not just the target state.

**Mitigation:**
EU region infrastructure provisioned in month 1 before any data migration begins.

EU customer records flagged and isolated at the data model level — not filtered at the application layer.

GDPR compliance audit scheduled month 4 before traffic exceeds 50% on the new stack.

Customer's Data Protection Officer (DPO) must be engaged in month 1 discovery — this is not optional and not a consulting team decision.

---

### Risk 4 — Application Entanglement

```
Likelihood:  MEDIUM
Impact:      MEDIUM
Owner:       Customer Application Team
```

**What it is:**
Limited refactoring budget means legacy SQL queries targeting the old relational schema may still be running in application code after migration.

If not addressed this creates a two-database problem — new document store for some queries, legacy RDBMS for others — indefinitely.

**Mitigation:**
Strangler Fig pattern applied from day one.
New features written against the document store.
Legacy queries identified and catalogued in month 1.

Compatibility/proxy layer evaluated in month 1 to translate legacy relational queries transparently without touching application code.

Full application refactoring is correctly in the backlog — beyond current budget.
The proxy layer buys time to do it properly.

---

### Risk 5 — Team Readiness and Knowledge Transfer

```
Likelihood:  MEDIUM
Impact:      MEDIUM
Owner:       Customer — Operations
```

**What it is:**
The engagement ends at month 6.
If the customer team is not operationally confident by then, the business becomes dependent on consulting support indefinitely.
That is not a success outcome.

**Mitigation:**
Knowledge transfer is not a month 6 activity. It starts month 2 and runs the full engagement.

Month 2: Customer team embedded alongside delivery team
Month 4: Customer team owns runbook creation
Month 5: Customer team leads, consulting team shadows
Month 6: Consulting team present for questions only

Operations signs off on operational readiness as the final engagement gate.

---

### Risk 6 — Timeline Slippage

```
Likelihood:  MEDIUM
Impact:      HIGH
Owner:       Joint — All Parties
```

**What it is:**
Peak season is a hard deadline.
Any uncontrolled scope addition mid-flight directly threatens the month 5 cutover gate.

**Mitigation:**
Change control process agreed and signed off in month 1 — before any build begins.

Any scope addition requires a formal impact assessment against the timeline before approval.
Stretch items are explicitly deprioritized if core delivery is at risk.

Ideal One person on the customer side holds change control authority. Decisions by committee mid-engagement are a timeline killer.

---

## Open Items — Customer to Resolve

These items cannot be driven by the consulting team.
They require customer decisions or actions.
Each has a resolution deadline tied to the roadmap.

---

### Open Item 1 — 5X Synthetic Load Testing Strategy

```
Owner:       Customer — Operations + DPO
Required by: End of Month 2
Blocks:      Month 5 load simulation gate
```

**The Problem:**
Load testing at 5X requires a representative dataset. The production dataset contains PII — you cannot run a 5X load test against live 
customer records without significant compliance risk.

**Options to Evaluate:**

```
Option A — Synthetic Data Generation
  Generate 32M synthetic customer records
  that mirror the statistical profile of
  production data without containing real PII.
  Tools: Faker, Mimesis (Python), or commercial data tools Tonic, Protegrity etc.
  Pro:  Clean GDPR position, fully repeatable.
  Con:  Synthetic data may not reflect edge cases in real production data patterns. Need separate pipelines to build schema refelctive data points.
```

```
Option B — Anonymised Production Data
  Export production dataset with PII masked:
  Names → random strings
  Emails → hashed
  Addresses → region-only (country/state retained for distribution accuracy, street removed)
  Pro:  Reflects real data distribution patterns
  Con:  Anonymisation process itself needs Stewards sign-off before execution
        Re-identification risk must be assessed
```

```
Option C — Scaled Production Subset
  Take 10% of real production records (fully anonymised) and scale load 10X to simulate 5X on full dataset
  Pro:  Smaller dataset, faster to anonymise
  Con:  May miss data distribution edge cases at the tail of the customer record set
```

**What We Need From the Customer:**


1. Data office engaged by end of month 1 as Load testing strategy cannot proceed without DPO sign-off on data handling

2. Decision on which option by end of month 2 Blocks month 3 environment setup and month 5 load simulation

3. If Option B or C chosen:
   Legal sign-off on anonymisation approach before any data leaves production environment


### Risk of Not Resolving:

**Month 5 load simulation runs against synthetic data only. If real production data has edge cases that synthetic data does not cover, those surface in production at peak — not in testing.**

---

### Open Item 2 — DPO Engagement for GDPR

```
Owner:       Customer Legal / Compliance Team
Required by: End of Month 1
Blocks:      EU region architecture sign-off
             Dual-write compliance validation
             Load testing data strategy
```


**Customer's Data Protection Office must be engaged before architecture is finalised.**

Three decisions require DPO input:

```
1. Which fields constitute PII in the customer profile and loyalty dataset?
   (determines what gets pinned to EU region)

2. Is the dual-write window — where data exists  in both RDBMS and document store simultaneously — acceptable under the customer's GDPR
   data processing agreements?

3. What anonymisation standard is acceptable for load testing data preparation?
```

This is not a consulting team decision.
It cannot be assumed or deferred to month 3.

---

### Open Item 3 — Change Control Authority

```
Owner:       Customer — Executive Decision
Required by: End of Month 1
Blocks:      Delivery governance
```

One named person on the customer side must hold change control authority for this engagement.

Scope changes approved by committee mid-flight are one of the most common causes of missed delivery timelines on engagements of this complexity.

We need a single decision-maker or a delegate who can say yes or no to a scope change within 48 hours.

---
